# Narrative Framing Guide

Language patterns for the pull-through narrative in `nuget-audit-report.md`.

The goal is to connect technical findings to business outcomes — framing dependency debt as
investment risk rather than a technical problem, and creating a compelling case for action.

---

## Core Framing Principle

> **"The cost of not acting compounds."**

Each month a dependency goes unaddressed:
- Security exposure window grows
- The version gap widens, increasing upgrade complexity
- Breaking changes accumulate across multiple versions
- Compatibility problems with newer tooling and frameworks emerge

Position each HIGH/CRITICAL finding as **deferred investment + compounding interest**, not just a
technical TODO.

---

## Executive Summary Framing

### Health Score Headlines

| Score | Opening Line |
|-------|-------------|
| A | "This solution has a strong dependency health profile with no immediate action required." |
| B | "This solution is in good health with a small number of items that warrant attention in the near term." |
| C | "This solution has several dependency health concerns that, if left unaddressed, will become progressively more costly to resolve." |
| D | "This solution has significant dependency risk. Deferred action on the items below carries real security and compliance exposure." |
| F | "This solution has critical security vulnerabilities and end-of-life platform components. Immediate action is required." |

### Investment Opportunity Paragraph (fill in N values)

```
This audit identifies {criticalCount + highCount} high-priority items and {mediumCount} medium-priority
items across {directCount} direct dependencies. Left unaddressed, these represent compounding technical
debt: as version gaps widen, future upgrades require progressively more effort, and the risk of a
security incident increases.

Addressing the Quick Wins alone resolves {lowCount} findings at minimal cost. The Projects tier
addresses the bulk of risk at manageable effort. The Initiatives represent strategic investments
with the highest long-term return — each day of delay makes them more expensive.
```

---

## Platform Section Framing

### Modern .NET — EOL Warning

```
.NET {version} ({LTS|STS}) reached end of support on {date}. Microsoft no longer issues security
patches for this runtime. Any vulnerability discovered after this date will remain unpatched, creating
ongoing security exposure for the application and its data.

Upgrading to a supported LTS release (.NET {recommendedVersion}) is the single most impactful change
this solution can make — it unlocks security patches, enables use of modern NuGet packages that have
dropped support for older runtimes, and reduces the complexity of future dependency upgrades.
```

### Modern .NET — Approaching EOL (< 6 months)

```
.NET {version} reaches end of support on {date} — in approximately {daysUntilEol} days. Planning an
upgrade now avoids a reactive, time-pressured migration. Upgrading while still in support means testing
can be thorough and unhurried.
```

### .NET Framework — Legacy Framing

```
This solution targets .NET Framework {version}, which is a Windows-only runtime with no cross-platform
capability and no planned future feature investment from Microsoft. While {version} remains supported
(security patches delivered via Windows Update), it is the final major version of .NET Framework.

Every NuGet package that the community modernises drops .NET Framework support over time, creating a
growing compatibility gap. The longer a migration is deferred, the more dependencies will have moved
to modern .NET-only releases — making a future migration more complex and costly.

We recommend establishing a migration roadmap to .NET 8 (LTS), which delivers significant performance
improvements, cross-platform capability, and a much richer ecosystem of actively maintained packages.
```

---

## Package Risk Framing

### CRITICAL — Security Vulnerability

```
**{PackageName}** has a known security vulnerability rated **{severity}** ({advisoryUrl}).
This is an active risk — the vulnerability is publicly documented and may already be being exploited
in the wild. The fix is available in v{fixedVersion}.

**Recommended action**: Upgrade immediately. This is a Quick Win if the version delta is small;
plan for a Project if it requires a major version jump.
```

### HIGH — Deprecated Package

```
**{PackageName}** has been officially deprecated by its author{, who recommends {alternativePackage} as a replacement}.
Deprecated packages receive no bug fixes or security patches. As the .NET ecosystem evolves, deprecated
packages become compatibility blockers — preventing upgrades to newer runtimes or other dependencies.

The cost of replacing this package grows with time: the more code that uses it, the more places need
to change. Current blast radius: {referencedByCount} project(s).
```

### HIGH — Source Repository Archived

```
**{PackageName}**'s source repository has been archived — the maintainer has indicated no further
development. While the package currently works, it will not receive security fixes, and compatibility
with future .NET versions is not guaranteed.

Packages from archived repositories are a long-term maintenance liability. We recommend evaluating
alternatives before they become a blocker.
```

### MEDIUM — Licence Changed

```
**{PackageName}** changed its licence from **{installedLicence}** to **{latestLicence}** between
the installed version and the latest. Licence changes can have legal and procurement implications —
particularly if moving from a permissive licence (MIT, Apache-2.0) to a commercial or copyleft licence.

We recommend reviewing the new licence with your legal team before upgrading.
```

### MEDIUM — Stale Package

```
**{PackageName}** has not had a new release in **{stalenessMonths} months**. Packages with no recent
activity may no longer be compatible with future .NET or OS versions, and vulnerabilities discovered
in the future will not be patched.

This is a leading indicator of abandonment. We recommend identifying an alternative or monitoring
this package's maintenance status.
```

---

## Recommendations Section Framing

### Quick Wins Introduction

```
The following items can be resolved with minimal effort — typically a package version bump that is
straightforward to test. Addressing these first builds confidence and reduces the risk count quickly.
```

### Projects Introduction

```
These items require more deliberate effort — a planned sprint or dedicated engineering time. Each one
resolves significant risk and is best addressed in the near term before the gap widens further.
```

### Initiatives Introduction

```
These are strategic investments that deliver the highest long-term value. They are more complex to
plan and execute, but the cost of continued deferral is significant — both in terms of compounding
upgrade complexity and growing security exposure. We recommend establishing a roadmap and timeline
for each.
```

### Cost of Delay (generic, append to any initiative)

```
Each {quarter|year} of deferral on this initiative is estimated to increase the remediation effort
by {15-30}% as additional packages drop support for the current runtime and breaking changes
accumulate across major versions.
```

---

## Tone Guidelines

- **Factual, not alarmist** — report facts; let the data speak
- **Action-oriented** — every finding has a "recommended action"
- **Business-framed** — use words like "investment", "risk", "exposure", "return", not just "bug" or "old version"
- **Quantified where possible** — "X packages", "Y months", "Z projects affected"
- **Future-focused** — frame delay as compounding cost, not just a current problem
