---
name: nuget-audit-orchestrator
description: >
  Full .NET dependency audit orchestrator. Use when a customer needs a complete health audit of all
  .NET dependencies in a solution — covering platform lifecycle, NuGet package currency, licences,
  security vulnerabilities, and risk scoring. Produces nuget-audit-result.json and
  nuget-audit-report.md. Does NOT depend on any other project skills.
---

# NuGet Audit — Orchestrator

Entry point for the `nuget-audit-*` skill suite. Coordinates the full audit pipeline and assembles final outputs.

## When to Use This Skill

Use when asked to:
- Audit, review, or analyse .NET dependencies in a solution
- Produce a dependency health report for a customer
- Identify upgrade risk, security issues, or licence changes across a .NET solution
- Generate a structured JSON + markdown output of solution health

---

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `solutionPath` | Path to `.sln`, `.slnx`, or root directory | Yes |
| `projectFilter` | Glob pattern to limit projects (e.g. `src/**`) | No |
| `outputPath` | Directory for output files | No — defaults to `./nuget-audit-output/` |

---

## Pipeline

Execute in the following order. Each phase produces JSON files consumed by the next.

### Phase 1 — Discovery (must complete first)

Invoke **`nuget-audit-discovery`**:
- Input: `solutionPath`, optional `projectFilter`
- Output: `discovery.json`
- Contains: project list, TFMs, direct packages, transitive packages

### Phase 2 — Analysis (run in parallel, all consume `discovery.json`)

Invoke these three simultaneously:

| Sub-skill | Output | Covers |
|-----------|--------|--------|
| `nuget-audit-platform` | `platform.json` | .NET & .NET Framework lifecycle, LTS/STS, EOL |
| `nuget-audit-packages` | `packages.json` | Versions, licences, compatibility, upgrade complexity |
| `nuget-audit-risk` | `risk.json` | CVEs, abandonment, transitive risk, community health |

### Phase 3 — Report (sequential, needs all Phase 2 outputs)

Invoke **`nuget-audit-report`**:
- Input: all four JSON files from Phases 1 & 2
- Output: `nuget-audit-result.json`, `nuget-audit-report.md`

---

## Output Files

```
{outputPath}/
├── discovery.json           # Raw inventory (projects, packages, TFMs)
├── platform.json            # Platform lifecycle analysis
├── packages.json            # Package metadata + upgrade analysis
├── risk.json                # Risk signals per package
├── nuget-audit-result.json  # Merged canonical JSON output
└── nuget-audit-report.md    # Customer-facing markdown report
```

---

## Top-Level JSON Schema (`nuget-audit-result.json`)

```json
{
  "auditedAt": "2025-01-01T00:00:00Z",
  "solution": "path/to/solution.sln",
  "summary": {
    "projectCount": 0,
    "directPackageCount": 0,
    "transitivePackageCount": 0,
    "riskCounts": {
      "critical": 0,
      "high": 0,
      "medium": 0,
      "low": 0
    },
    "platformStatus": "current | warn | eol",
    "overallHealthScore": "A | B | C | D | F"
  },
  "platform": [],
  "packages": [],
  "risk": []
}
```

---

## Overall Health Score

| Score | Criteria |
|-------|----------|
| **A** | No CRITICAL or HIGH findings; all platforms current |
| **B** | No CRITICAL; ≤ 2 HIGH; platforms current or > 6 months from EOL |
| **C** | ≤ 1 CRITICAL or ≤ 5 HIGH; or a platform is within 6 months of EOL |
| **D** | > 1 CRITICAL or > 5 HIGH; or a platform is past EOL |
| **F** | Platform EOL AND multiple CRITICAL vulnerabilities |

Apply the worst-matching grade. Report the score prominently in the Executive Summary.

---

## Agent Checklist

Before calling sub-skills, verify:
- [ ] `.sln` / `.slnx` / solution directory is accessible
- [ ] `dotnet` CLI is available: `dotnet --version`
- [ ] NuGet feeds are configured (check `dotnet nuget list source`)

After pipeline completes, verify:
- [ ] All 6 output files exist
- [ ] `nuget-audit-result.json` passes JSON validation
- [ ] `nuget-audit-report.md` is non-empty and contains all 7 sections

---

## Quick Reference: Sub-Skill Invocation Order

```
nuget-audit-discovery
       ↓
  ┌────┴─────────────┐
  ▼                  ▼                  ▼
nuget-audit-platform  nuget-audit-packages  nuget-audit-risk
  └────┬─────────────┘
       ↓
nuget-audit-report
```
