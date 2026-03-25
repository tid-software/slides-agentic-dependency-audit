---
name: nuget-audit-report
description: >
  NuGet audit report generation skill. Assembles the canonical nuget-audit-result.json and renders
  a customer-facing nuget-audit-report.md from the outputs of nuget-audit-platform,
  nuget-audit-packages, and nuget-audit-risk. Includes pull-through narrative framing and
  prioritised recommendations. Does NOT depend on any other project skills.
---

# NuGet Audit — Report

Assembles all analysis outputs into the final deliverables: a structured JSON file and a
customer-facing markdown report with pull-through narrative framing.

## When to Use This Skill

Invoke as **Phase 3** of `nuget-audit-orchestrator` after all Phase 2 skills complete.
Inputs: `discovery.json`, `platform.json`, `packages.json`, `risk.json`.
Outputs: `nuget-audit-result.json`, `nuget-audit-report.md`.

---

## Step 1: Merge JSON → `nuget-audit-result.json`

Combine all four input files into the canonical schema defined in `nuget-audit-orchestrator`:

```json
{
  "auditedAt": "<ISO-8601 timestamp>",
  "solution": "<solutionPath>",
  "summary": {
    "projectCount": "<from discovery.json>",
    "directPackageCount": "<from discovery.json>",
    "transitivePackageCount": "<from discovery.json>",
    "riskCounts": "<from risk.json summary>",
    "platformStatus": "<from platform.json overallPlatformStatus>",
    "overallHealthScore": "<computed — see orchestrator scoring table>"
  },
  "platform": "<platform.json platforms array>",
  "packages": "<join packages.json + risk.json by package id>",
  "risk": "<risk.json packages array>"
}
```

When joining `packages.json` and `risk.json`:
- Match on `id` (case-insensitive)
- Merge fields from both into a single package object
- Packages in `risk.json` that are transitive-only (not in `packages.json`) still appear in the `risk` array

Save to `{outputPath}/nuget-audit-result.json`.

---

## Step 2: Render `nuget-audit-report.md`

The report has **7 sections**. See [`references/report-templates.md`](references/report-templates.md) for
full markdown templates. See [`references/narrative-framing.md`](references/narrative-framing.md) for
pull-through language patterns.

### Section 1: Executive Summary

- **Health Score** prominently at the top (A / B / C / D / F)
- **Headline finding** — one sentence summarising the biggest risk
- **Key metrics**: project count, direct packages, transitive packages, CRITICAL/HIGH/MEDIUM/LOW counts
- **Platform status** — one line per TFM (current / warn / EOL)
- **Pull-through hook**: "This report identifies N items representing estimated uplift investment of…"

Use pull-through language from `references/narrative-framing.md`.

### Section 2: Platform Health

Table format:

| TFM | Type | Version | Release Type | End of Support | Days Until EOL | Status |
|-----|------|---------|-------------|----------------|----------------|--------|
| net8.0 | Modern | 8 | LTS | 2026-11-10 | 250 | ✅ Current |
| net48 | Legacy | 4.8 | Legacy | Tied to Windows | — | ⚠️ Supported |
| net471 | Legacy | 4.7.1 | Legacy | 2023-01-10 | -400 | ❌ EOL |

Status icons: ✅ current, ⚠️ warn (< 6 months to EOL or legacy-supported), ❌ EOL.

Include a paragraph below the table explaining what EOL means for the customer (no security patches,
compliance exposure, vendor support risks). Use `references/narrative-framing.md` § Platform section.

### Section 3: Dependency Health Overview

Summary table of all direct packages by risk level:

| Risk Level | Count | Examples |
|------------|-------|---------|
| 🔴 CRITICAL | N | Package A, Package B |
| 🟠 HIGH | N | Package C |
| 🟡 MEDIUM | N | Package D, Package E |
| 🟢 LOW | N | … |
| ✅ OK | N | … |

Then a brief paragraph quantifying the technical debt: "Of N direct packages, X% require attention."

### Section 4: Package Detail Table

Full table of all **direct** packages:

| Package | Installed | Latest | Delta | Upgrade Complexity | Licence | Last Published | Compatible | Risk |
|---------|-----------|--------|-------|--------------------|---------|---------------|------------|------|
| Newtonsoft.Json | 12.0.3 | 13.0.3 | Major | 🔴 HIGH | MIT (unchanged) | 2023-03 | ✅ | HIGH |
| Serilog | 3.0.0 | 4.0.0 | Major | 🟠 HIGH | Apache-2.0 (unchanged) | 2024-01 | ✅ | LOW |

Delta colour coding: 🔴 Major, 🟠 Minor, 🟡 Patch, ✅ Current.

Complexity colour coding: 🔴 HIGH, 🟠 MEDIUM, 🟢 LOW, ✅ None.

Sort by risk score descending (CRITICAL first), then by upgrade complexity descending.

### Section 5: Vulnerability Report

Only include this section if there are any vulnerabilities.

Group by severity:

```
## 🔴 Critical Vulnerabilities

### {PackageName} v{version}
- **Advisory**: [{GHSA-xxxx}](https://github.com/advisories/GHSA-xxxx)
- **Type**: Direct / Transitive (introduced by {DirectPackage})
- **Recommended action**: Upgrade to v{fixedVersion} or later

## 🟠 High Vulnerabilities
...
```

### Section 6: Recommendations

Prioritised action list. Group by effort tier (Quick Win first, then Project, then Initiative):

```
## Quick Wins (< 1 day each)

1. **Upgrade {Package} from {installed} to {latest}** (patch bump, no breaking changes)
   - Risk resolved: LOW
   - Projects affected: {list}

## Projects (1–5 days)

2. **Replace deprecated {Package} with {Alternative}**
   - Risk resolved: HIGH
   - Estimated effort: 2 days
   - Why now: The package author has archived the source repository...

## Initiatives (Weeks–Months)

3. **Migrate from .NET Framework 4.7.1 to .NET 8**
   - Risk resolved: Platform EOL, unlocks N HIGH-risk package upgrades
   - Business value: Cross-platform deployment, 30-40% performance improvement, ongoing security support
```

Use `references/narrative-framing.md` § Recommendations for pull-through language.

### Section 7: Appendix — Transitive Dependencies

Table of all transitive packages with any non-OK risk:

| Package | Version | Risk | Introduced By | Signal |
|---------|---------|------|--------------|--------|
| VulnerableDep | 1.2.0 | 🔴 CRITICAL | DirectPkg.X | CVE-2024-1234 |
| StaleDep | 3.0.0 | 🟡 MEDIUM | DirectPkg.Y | Stale 36 months |

---

## Step 3: Validate Outputs

- [ ] `nuget-audit-result.json` is valid JSON (all fields present, no `null` for required fields)
- [ ] `nuget-audit-report.md` contains all 7 sections
- [ ] Section 5 omitted only if vulnerability count is genuinely 0
- [ ] Health score matches the criteria in `nuget-audit-orchestrator`
- [ ] Recommendations sorted: Quick Win → Project → Initiative
- [ ] Save both files to `{outputPath}/`

---

## Reference Files

- [`references/report-templates.md`](references/report-templates.md) — full markdown templates per section
- [`references/narrative-framing.md`](references/narrative-framing.md) — pull-through language patterns and business impact descriptions
