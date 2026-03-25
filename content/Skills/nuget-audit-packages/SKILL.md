---
name: nuget-audit-packages
description: >
  NuGet package metadata enrichment skill. For each direct package in a .NET solution, retrieves the
  latest version, licence (with change detection), last published date, framework compatibility, and
  computes an upgrade complexity rating. Produces packages.json. Does NOT depend on any other project
  skills.
---

# NuGet Audit — Packages

Enriches each direct NuGet package with metadata from the NuGet V3 API: version currency, licence,
publish date, framework compatibility, deprecation status, and upgrade complexity.

## When to Use This Skill

Invoke as part of **Phase 2** of `nuget-audit-orchestrator` (parallel with `nuget-audit-platform` and
`nuget-audit-risk`) after `nuget-audit-discovery` completes.
Input: `discovery.json`. Output: `packages.json`.

---

## NuGet V3 API Endpoints

All endpoints below use `{id-lower}` = package ID lowercased.

| Purpose | URL |
|---------|-----|
| All available versions | `GET https://api.nuget.org/v3-flatcontainer/{id-lower}/index.json` |
| Full metadata (licence, dates, TFMs) | `GET https://api.nuget.org/v3/registration5-gz-semver2/{id-lower}/index.json` |
| Deprecation info | Included in registration metadata (`deprecation` field) |

> **Private feeds**: The skill assumes the `dotnet` CLI and NuGet.org are both accessible.
> For packages only on private feeds, the NuGet.org REST API will return 404 — fall back to
> `dotnet package search {id} --exact-match --format json` which uses configured feed(s).

---

## Step 1: Get Latest Version

```
GET https://api.nuget.org/v3-flatcontainer/{id-lower}/index.json
```

Response: `{ "versions": ["1.0.0", "1.0.1", ..., "13.0.3"] }`

Latest stable = last version that does NOT contain `-alpha`, `-beta`, `-preview`, `-rc`.

---

## Step 2: Get Registration Metadata

```
GET https://api.nuget.org/v3/registration5-gz-semver2/{id-lower}/index.json
```

The response contains paginated `items` arrays. For each version entry (`catalogEntry`), extract:

- `version` — version string
- `published` — ISO-8601 publish timestamp
- `licenseExpression` — SPDX expression (e.g. `"MIT"`, `"Apache-2.0"`)
- `licenseUrl` — fallback if `licenseExpression` absent (deprecated field)
- `deprecation` — object present if package is deprecated; contains `message` and `reasons`
- `listed` — `false` if the version is unlisted/hidden
- `dependencyGroups` — array of `{ targetFramework, dependencies }` used for TFM compatibility

Retrieve entries for both the **installed version** and the **latest stable version**.

### Licence Change Detection

| Condition | Flag |
|-----------|------|
| `licenseExpression` (or URL) is identical between installed and latest | `changed: false` |
| Different expressions or URLs | `changed: true` |
| Installed has expression, latest has none (or vice versa) | `changed: true` — note "licence metadata removed" |

---

## Step 3: Framework Compatibility Check

From the `dependencyGroups` in the registration entry for the **latest** version, extract supported TFMs:

```json
"dependencyGroups": [
  { "targetFramework": "net6.0", "dependencies": [] },
  { "targetFramework": "netstandard2.0", "dependencies": [] }
]
```

Check whether **any** entry is compatible with the project's TFM using these rules:
- `netstandard2.0` and `netstandard2.1` are compatible with all modern .NET TFMs
- Exact TFM match is always compatible
- `net6.0` package is NOT compatible with `net8.0` unless the package also lists a `net8.0` group or `netstandard` fallback
- If `dependencyGroups` is empty or missing, assume compatible (package targets any framework)

Flag `frameworkCompatible: false` only when you are confident the latest version does NOT support the project's TFM.

---

## Step 4: Upgrade Complexity Rating

Rate the effort required to upgrade from installed to latest:

### Version Delta

| Delta type | Base rating |
|------------|-------------|
| Same version (already latest) | `none` |
| Patch bump (same major.minor) | `LOW` |
| Minor bump (same major) | `MEDIUM` |
| Major bump | `HIGH` |

### Blast Radius Modifier

If the package is referenced by **> 3 projects**, upgrade the rating one level (LOW → MEDIUM, MEDIUM → HIGH, HIGH stays HIGH).

### Known Breaking Changes Modifier

If the package's `releaseNotesUrl` is present in metadata, or if it is a Microsoft package (namespace `Microsoft.*`, `System.*`, `Azure.*`):
- Link to `https://learn.microsoft.com/en-us/dotnet/core/compatibility/` for known breaking changes
- Flag `hasBreakingChangesReference: true`

### Deprecated Packages

```bash
dotnet list package --deprecated
```

Record `deprecated: true` and `deprecationReason` if present. A deprecated package is always rated `HIGH` complexity regardless of version delta, because a replacement package must be found.

---

## Step 5: Age Calculation

`ageMonths` = months since the installed version was published (from `published` field).

Thresholds:
- `ageMonths` > 36 → flag `isStale: true`
- `ageMonths` > 24 → flag `isStale: true` (if no newer version has been published either)

---

## Output: `packages.json`

```json
{
  "analysedAt": "ISO-8601 timestamp",
  "packages": [
    {
      "id": "Newtonsoft.Json",
      "installed": "12.0.3",
      "latest": "13.0.3",
      "versionDelta": "major",
      "upgradeComplexity": "HIGH",
      "upgradeComplexityReason": "Major version bump; referenced by 5 projects",
      "licence": {
        "installed": "MIT",
        "latest": "MIT",
        "changed": false
      },
      "publishedInstalled": "2019-06-20T00:00:00Z",
      "publishedLatest": "2023-03-08T00:00:00Z",
      "ageMonths": 70,
      "isStale": true,
      "frameworkCompatible": true,
      "deprecated": false,
      "deprecationReason": null,
      "hasBreakingChangesReference": true,
      "breakingChangesUrl": "https://learn.microsoft.com/en-us/dotnet/core/compatibility/",
      "referencedByProjects": ["MyApp", "MyApp.Api", "MyApp.Workers"],
      "referencedByCount": 3
    }
  ]
}
```

---

## Agent Checklist

- [ ] Only **direct** packages from `discovery.json` processed (not transitive — risk skill handles those)
- [ ] NuGet V3 API called for each package; 404 = try `dotnet package search` fallback
- [ ] Latest stable version excludes pre-release suffixes
- [ ] Licence change detection performed for every package
- [ ] TFM compatibility checked against the project's TFM from `discovery.json`
- [ ] `ageMonths` calculated from today's date
- [ ] Upgrade complexity rating documented with reason string
- [ ] Output saved to `{outputPath}/packages.json`

---

## Common Issues

| Problem | Resolution |
|---------|------------|
| Package not on NuGet.org (private) | Fallback to `dotnet package search {id} --exact-match --format json` |
| Pagination in registration response | If `items` is a URL string (not array), GET that URL for the page |
| `licenseExpression` absent | Use `licenseUrl`; if both absent, set `"unknown"` |
| Very large solution (100+ packages) | Process in batches of 20; respect NuGet API rate limits |
