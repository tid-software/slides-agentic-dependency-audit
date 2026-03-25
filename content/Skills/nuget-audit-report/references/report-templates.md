# Report Templates

Markdown templates for each section of `nuget-audit-report.md`. Copy and fill in the placeholders.

---

## Document Header

```markdown
# .NET Dependency Health Audit

**Solution**: `{solutionName}`
**Audited**: {date}
**Overall Health Score**: {A|B|C|D|F}

---
```

---

## Section 1: Executive Summary Template

```markdown
## Executive Summary

> **Health Score: {score}** — {one-sentence headline finding}

This audit covers **{projectCount} projects**, **{directCount} direct dependencies**, and
**{transitiveCount} transitive dependencies**.

| Risk Level | Direct Packages | Transitive Packages |
|------------|-----------------|---------------------|
| 🔴 Critical | {n} | {n} |
| 🟠 High | {n} | {n} |
| 🟡 Medium | {n} | {n} |
| 🟢 Low | {n} | {n} |
| ✅ OK | {n} | {n} |

**Platform Status**: {overall platform status line}

### Key Findings

- {Finding 1 — most critical issue}
- {Finding 2}
- {Finding 3}

### Investment Opportunity

{Pull-through paragraph — see narrative-framing.md § Executive Summary}
```

---

## Section 2: Platform Health Template

```markdown
## Platform Health

| TFM | Type | Version | Release Type | End of Support | Days Until EOL | Status |
|-----|------|---------|-------------|----------------|----------------|--------|
| {tfm} | {Modern\|Legacy} | {version} | {LTS\|STS\|Legacy} | {date\|"Tied to Windows"} | {n\|"—"} | {✅\|⚠️\|❌} {label} |

{Platform narrative paragraph — see narrative-framing.md § Platform}
```

---

## Section 3: Dependency Health Overview Template

```markdown
## Dependency Health Overview

| Risk Level | Count | Notable Packages |
|------------|-------|-----------------|
| 🔴 Critical | {n} | {pkg1}, {pkg2} |
| 🟠 High | {n} | {pkg3} |
| 🟡 Medium | {n} | {pkg4}, {pkg5} |
| 🟢 Low | {n} | … |
| ✅ OK | {n} | — |

Of **{totalDirect}** direct packages, **{percentAtRisk}%** require attention. 
{Technical debt paragraph}
```

---

## Section 4: Package Detail Table Template

```markdown
## Package Details

| Package | Installed | Latest | Delta | Complexity | Licence | Last Published | TFM ✓ | Risk |
|---------|-----------|--------|-------|------------|---------|---------------|--------|------|
| {id} | {installed} | {latest} | {🔴 Major\|🟠 Minor\|🟡 Patch\|✅ Current} | {🔴 HIGH\|🟠 MEDIUM\|🟢 LOW\|✅ None} | {licence} ({changed?\|unchanged}) | {YYYY-MM} | {✅\|❌} | {score} |
```

Sort: CRITICAL first → HIGH → MEDIUM → LOW → OK. Within each group, sort by upgrade complexity descending.

---

## Section 5: Vulnerability Report Template

```markdown
## Vulnerability Report

{Include only if vulnerability count > 0}

### 🔴 Critical

#### {PackageName} v{version}
- **Advisory**: [{GHSA-id}]({advisoryUrl})
- **Severity**: Critical
- **Reference type**: Direct / Transitive (via {directPackage})
- **Remediation**: Upgrade to v{fixedVersion} or later

---

### 🟠 High

{same structure}

---

### 🟡 Medium / 🟢 Low

{same structure}
```

---

## Section 6: Recommendations Template

```markdown
## Recommendations

Actions are ordered by effort tier. Resolve CRITICAL and HIGH risks first.

### ⚡ Quick Wins *(< 1 day each)*

{For each LOW/MEDIUM risk that is a patch or minor bump with low blast radius}

**{n}. Upgrade {Package} {installed} → {latest}**
- Risk resolved: {score}
- Projects affected: {list}
- Effort: Patch version bump — run `dotnet add package {Package} --version {latest}` in each project

---

### 🔨 Projects *(1–5 days each)*

{For each HIGH risk or major bumps with moderate blast radius}

**{n}. {Action description}**
- Risk resolved: {score}
- Projects affected: {list}
- Estimated effort: {x} days
- Why: {pull-through reason}

---

### 🏗️ Initiatives *(Weeks to months)*

{For platform migrations, systemic changes}

**{n}. {Initiative name}**
- Risks resolved: {list}
- Business value: {value statement}
- Estimated effort: {range}
- Why now: {compounding cost of delay framing}
```

---

## Section 7: Appendix — Transitive Dependencies Template

```markdown
## Appendix: Transitive Dependencies

The following transitive (indirect) dependencies have risk signals. The "Introduced By" column
identifies which direct package is responsible — upgrading that direct package typically resolves
the transitive risk.

| Package | Version | Risk | Introduced By | Signal |
|---------|---------|------|--------------|--------|
| {id} | {version} | {score} | {directPkg} | {signal description} |

{Omit rows with OK risk to keep the table manageable.}
```
