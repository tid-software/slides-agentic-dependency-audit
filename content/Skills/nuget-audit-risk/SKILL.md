---
name: nuget-audit-risk
description: >
  NuGet dependency risk analysis skill. Enriches packages with security vulnerability data, abandonment
  signals, transitive dependency risk mapping, community health indicators, and a composite CRITICAL/
  HIGH/MEDIUM/LOW risk score. Produces risk.json. Does NOT depend on any other project skills.
---

# NuGet Audit — Risk

Surfaces risk signals across direct and transitive NuGet packages to identify security exposure,
maintenance risk, and community health concerns.

## When to Use This Skill

Invoke as part of **Phase 2** of `nuget-audit-orchestrator` (parallel with `nuget-audit-platform` and
`nuget-audit-packages`) after `nuget-audit-discovery` completes.
Input: `discovery.json`. Output: `risk.json`.

---

## Risk Signal 1: Security Vulnerabilities (CVEs)

```bash
# Check direct packages
dotnet list package --vulnerable

# Check all packages including transitive
dotnet list package --vulnerable --include-transitive
```

For each vulnerable package, record:
- `advisoryUrl` — link to the GitHub Advisory or NVD entry
- `severity` — `low | moderate | high | critical` (as reported by dotnet CLI)
- `isDirect` — `true` if this is a direct package reference, `false` if transitive

> The `dotnet` CLI sources vulnerability data from the GitHub Advisory Database via NuGet's built-in
> audit feed. Results are only available when packages are restored. Run `dotnet restore` first.

---

## Risk Signal 2: Package Deprecation

```bash
dotnet list package --deprecated
```

For each deprecated package:
- `deprecationReason` — the reason string (e.g. `"Legacy"`, `"CriticalBugs"`, `"Other"`)
- `alternativePackage` — suggested replacement (if the author provided one)

---

## Risk Signal 3: Abandonment Signals

A package is considered **abandoned/stale** if:

### Staleness (no recent releases)
- More than **24 months** since the latest published version AND the package is behind its latest version
- Source: `publishedLatest` from `nuget-audit-packages` output (or recompute from registration API)

### Official Deprecation
- `deprecated: true` in NuGet metadata (covered in Signal 2)

### Source Repository Archival (GitHub)
If the package's `projectUrl` or `repositoryUrl` in NuGet metadata points to a GitHub repository:

```
GET https://api.github.com/repos/{owner}/{repo}
```

Check:
- `archived: true` → repo has been archived by the owner
- `pushed_at` → last push date (if > 24 months ago and not archived, flag as `stale-activity`)

> **Rate limiting**: Unauthenticated GitHub API allows 60 requests/hour. For large solutions, check
> only packages with > 12 months since last NuGet publish to prioritise API calls.
> Set `User-Agent` header to avoid rejection.

---

## Risk Signal 4: Transitive Dependency Risk

For each **transitive** package that has a vulnerability or is deprecated:
- Identify which **direct** package(s) pull it in (from `discovery.json` `pulledInBy` field)
- Record as `transitiveDependents` on the transitive package entry
- Also add a `transitiveRisks` array to the relevant **direct** package entry

This guides the upgrade path: fixing the transitive risk means upgrading the direct package that introduces it.

---

## Risk Signal 5: Community Health

### Download Count

From the NuGet registration metadata (`totalDownloads` field on the package page):

```
GET https://api.nuget.org/v3/registration5-gz-semver2/{id-lower}/index.json
```

The `totalDownloads` field is not always in registration. Alternative — NuGet stats endpoint:

```
GET https://api.nuget.org/v3/stats/packageVersionDownloads
```

Thresholds:
- < **10,000** total downloads → `lowAdoption: true` (high risk for obscure/abandoned packages)
- ≥ 10,000 but < 100,000 → `lowAdoption: false`, note as "moderate adoption"
- ≥ 100,000 → well-adopted, no flag

> If download count is unavailable (private packages, API timeout), skip this signal and note `downloadCount: null`.

### Last Version Activity

Already computed as `ageMonths` in `nuget-audit-packages`. Reference that; do not recompute.

---

## Risk Signal 6: Known Breaking Changes

For Microsoft-ecosystem packages (namespaces `Microsoft.*`, `System.*`, `Azure.*`, `AspNetCore.*`):
- Link to: `https://learn.microsoft.com/en-us/dotnet/core/compatibility/`
- Narrow by library name if possible (e.g. search the page for the package name)

For non-Microsoft packages:
- Check if `releaseNotesUrl` is present in NuGet metadata
- If a GitHub repo is found, link to `{repoUrl}/releases` for release notes

---

## Composite Risk Score

Apply the following rules in order (first matching rule wins):

| Risk Level | Conditions |
|------------|-----------|
| **CRITICAL** | Has a vulnerability with severity `critical` or `high`; OR is a transitive dep pulled in by a direct package AND that transitive has `critical`/`high` CVE |
| **HIGH** | Deprecated (with no clear migration path); OR vulnerability severity `moderate`; OR source repo is archived; OR > 2 major versions behind latest; OR framework incompatible with latest version |
| **MEDIUM** | Stale > 24 months; OR licence changed between installed and latest; OR low adoption (< 10k downloads); OR vulnerability severity `low`; OR > 1 minor version behind |
| **LOW** | Only patch versions behind; OR minor version behind with no other signals |
| **OK** | No issues found |

**Important:** Record ALL signals that contributed to the score, not just the highest one.

---

## Output: `risk.json`

```json
{
  "analysedAt": "ISO-8601 timestamp",
  "packages": [
    {
      "id": "SomePackage",
      "type": "direct",
      "riskScore": "HIGH",
      "riskFactors": ["deprecated", "sourceRepoArchived"],
      "vulnerabilities": [],
      "deprecated": true,
      "deprecationReason": "Legacy",
      "alternativePackage": "BetterPackage",
      "abandoned": true,
      "sourceRepoArchived": true,
      "stalenessMonths": 36,
      "downloadCount": 85000,
      "lowAdoption": false,
      "breakingChangesUrl": null,
      "transitiveRisks": []
    },
    {
      "id": "VulnerableTransitiveDep",
      "type": "transitive",
      "riskScore": "CRITICAL",
      "riskFactors": ["vulnerability-critical"],
      "vulnerabilities": [
        {
          "advisoryUrl": "https://github.com/advisories/GHSA-xxxx",
          "severity": "critical",
          "isDirect": false
        }
      ],
      "transitiveDependents": ["DirectPackage.X"],
      "deprecated": false,
      "abandoned": false,
      "stalenessMonths": 8,
      "downloadCount": 5000000,
      "lowAdoption": false
    }
  ],
  "summary": {
    "critical": 1,
    "high": 2,
    "medium": 3,
    "low": 4,
    "ok": 10
  }
}
```

---

## Agent Checklist

- [ ] `dotnet restore` run before `dotnet list package --vulnerable`
- [ ] Both direct AND transitive packages checked for vulnerabilities
- [ ] Deprecated packages captured with reason and alternative
- [ ] GitHub API only called for packages with stale NuGet publish dates (rate limit conservation)
- [ ] `transitiveDependents` populated from `discovery.json` `pulledInBy` data
- [ ] Risk score applied consistently using the scoring table above
- [ ] All contributing signals recorded in `riskFactors` array
- [ ] Output saved to `{outputPath}/risk.json`

---

## Scoring Reference

See [`references/risk-scoring-matrix.md`](references/risk-scoring-matrix.md) for the full decision table with worked examples.
