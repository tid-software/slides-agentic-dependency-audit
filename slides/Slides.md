---
marp: true
theme: custom-default
paginate: true
footer: 'https://github.com/tid-software/slides-agentic-dependency-audit'
math: mathjax
---

<!-- _paginate: skip -->
<!-- _footer: "" -->
<!-- _class: lead -->

# Feeding & Watering Your .NET Solution

AI-Assisted Dependency Auditing with GitHub Copilot Agent Skills

![bg right opacity:0.6](https://images.unsplash.com/photo-1558618666-fcd25c85cd64?w=800)

<!-- Speaker notes: Welcome. Today we're going to talk about a problem every .NET team has but rarely talks about — dependency drift. We're going to look at how we used GitHub Copilot's agentic capabilities to automate a full dependency audit, and we'll share the real results from a non trivial solution. -->

---

<!-- _class: lead invert -->

## The Problem

> Every week you ship without upgrading your dependencies,  
> the cost of catching up **compounds**.

<!-- Speaker notes: Here's the uncomfortable truth. Dependencies drift. Not because anyone decides to let them drift — it just happens. Releases ship, the backlog fills up, and before long you're two major versions behind on something critical. And when you finally have to deal with it, it's not one upgrade — it's ten, and they interact. 

This is the messaging that we tell our costomers; our ops team spends hours manualy pullingtogether depenency information into a cohesive message to deliver our customers.

-->

---

## The Manual Audit Problem

<div class="columns">
<div>

### Where the data lives

- `dotnet list package` — version numbers only
- NuGet registry API — licence, age, downloads
- GitHub / source repos — maintainer health
- CVE databases — security advisories
- EOL calendars — platform support windows
- Licence texts — compliance verification

</div>
<div>

### What it costs

- Hours of copy-paste per solution
- Stale before it's finished
- No consistent format or scoring
- Impossible to repeat reliably
- Scales poorly — 76 projects × 115 packages
- Different engineer, different result

</div>
</div>


<!-- Speaker notes: This is the hidden cost. Every time a customer asks "are we keeping on top of our dependencies?", someone on our ops team manually assembles the answer. It takes hours. It's inconsistent. And by the time the report lands in an inbox, some of it is already out of date. -->

---
<!-- _paginate: skip -->
<!-- _footer: "" -->
<!-- _class: lead -->

![bg right:40%](img/this-is-fine.jpeg)

> This isn't a one-off task — it's a recurring obligation  
> that nobody has time to do properly.

<!-- Speaker notes: so to smmary; everybodody is busy doing all the things  . -->

---

## The Communication Gap

<div class="columns">
<div>

### What the engineer sees

```
coverlet.collector 6.0.4 → 8.0.0
MassTransit 9.0.0-develop.23
74 / 115 packages outdated
12 pre-release
0 CVEs
```

Actionable — if you know what it means.

</div>
<div>

### What the business hears

- *"Some packages need updating"*
- *"There are pre-release things"*
- *"CVEs are fine"*
- *"...so are we okay?"*

No context. No priority.  
No answer to **"should I be worried?"**

</div>
</div>

<!-- Speaker notes: This is the second problem. Even when someone runs the audit, the output is written for engineers. A product owner reading "74 of 115 packages outdated, 12 pre-release, 0 CVEs" has no idea whether to raise this with the board or ignore it. The numbers don't tell a story — they're just numbers. Our job is to turn the numbers into a narrative that drives the right decision. -->

---

<!-- _footer: "" -->
<!-- _class: lead -->

![bg right:40%](img/confused_math_lady_meme.jpg)

> The data exists. The story doesn't.  
> Translating one into the other is the gap.

<!-- Speaker notes: This is the second problem. Even when someone runs the audit, the output is written for engineers. A product owner reading "74 of 115 packages outdated, 12 pre-release, 0 CVEs" has no idea whether to raise this with the board or ignore it. The numbers don't tell a story — they're just numbers. Our job is to turn the numbers into a narrative that drives the right decision. -->

---

## What Does Drift Look Like?

<div class="columns">
<div>

### Today

- 74 of 115 packages outdated
- 12 pre-release builds in production
- 1 unofficial branch build (`nblumhardt`)
- Unknown licence metadata on ~30 packages
- 1 major version gap in test infrastructure

</div>
<div>

### Left Unchecked

- Minor → Major version jumps compound
- Breaking changes stack up
- Security fixes get missed
- Licence changes go unnoticed
- One "upgrade sprint" becomes an initiative

</div>
</div>

<!-- Speaker notes: These are real numbers — from a real solution we audited this week. The drift isn't catastrophic yet, but the trajectory matters. Each release cycle that passes without housekeeping makes the next one harder. -->

---
<!-- _footer: "" -->
<!-- _class: lead invert -->

![bg left:40%](img/disaster_girl_meame.webp)

## TL;DR

The solution is **not on fire**.

But someone left the stove on,  
the smoke alarm is beeping,  
and everyone is too busy shipping  
to check what's burning.

<!-- Speaker notes: This is the TL;DR. Not a crisis — but the early signs of one. The good news is we caught it. The better news is we can fix it. -->

---

<!-- _class: lead -->

# Act 1
## What Are AI Agent Skills?

---

## What Are AI Agent Skills?

A **skill** is a markdown file (`SKILL.md`) that tells GitHub Copilot how to do a specific task.

<div class="columns">
<div>

### What it contains

- **When to use** this skill
- **What commands** to run
- **How to interpret** the output
- **What to produce** at the end

</div>
<div>

### What it's not

- Not a plugin
- Not compiled code
- Not a configuration file
- Just **structured natural language**  
  that the agent reads at runtime

</div>
</div>

> Think of it like onboarding documentation —  
> but your agent reads it and actually follows it.

<!-- Speaker notes: This is the key insight. Skills aren't magic — they're just well-written instructions. The agent reads them the same way a new developer would read a runbook. The difference is the agent actually does it. -->

---

## When Does a Skill Get Invoked?

<div class="mermaid">
flowchart LR
    A[👤 Developer prompt] --> B{Does it mention\na known topic?}
    B -->|Yes| C[Agent finds\nmatching SKILL.md]
    B -->|No| D[Agent uses\ngeneral knowledge]
    C --> E[Agent reads skill\nfrontmatter + content]
    E --> F[Agent follows\nthe skill's workflow]
    F --> G[✅ Consistent,\nguided output]
</div>

<!-- Speaker notes: The matching is semantic, not keyword-based. If you ask "can you audit our dependencies", Copilot recognises this matches the nuget-audit-orchestrator skill description and loads it automatically. You don't have to say "use skill X". -->

<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true });
</script>

---

## A Simple Example

<div class="columns">
<div>

### `SKILL.md` (excerpt)

```markdown
---
name: dotnet-restore-check
description: >
  Validates that a .NET solution
  restores cleanly before building.
---

## Steps

1. Run `dotnet restore`
2. If exit code != 0, report errors
3. List any missing packages
4. Suggest fixes for common errors
```

</div>
<div>

### What Copilot does

1. Reads the skill at invocation time
2. Runs `dotnet restore` in the terminal
3. Parses the output
4. Formats a structured report
5. Suggests fixes from its training

**No C# code. No plugins.**  
Just instructions the agent follows.

</div>
</div>

<!-- Speaker notes: This is the beautiful simplicity of it. The skill author writes what they know — the commands, the logic, what good output looks like. The agent brings the execution and the reasoning. -->

---

## Learn More: GitHub Copilot Agent Mode

| Resource | Link |
|----------|------|
| 🤖 GitHub Copilot Agent Mode | `github.com/features/copilot/agent-mode` |
| 📄 Custom Instructions | `docs.github.com/en/copilot/customizing-copilot` |
| 🛠️ Copilot Coding Agent | `docs.github.com/en/copilot/using-github-copilot/using-copilot-coding-agent` |
| 📦 Skills / Prompt Files | `docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions` |
| 🏫 GitHub Copilot Docs | `docs.github.com/en/copilot` |

> All resources are **GitHub Copilot** specific — no third-party AI required.

<!-- Speaker notes: These are the official GitHub docs. Bookmark the custom instructions page in particular — that's where the skill system described today lives. -->

---

<!-- _class: lead -->

# Act 2
## The Audit in Action

---

## What We Built: `nuget-audit-*`

Six self-contained skills. Zero external dependencies. Runs entirely from the `dotnet` CLI.

| Skill | Role |
|-------|------|
| `nuget-audit-orchestrator` | Pipeline coordinator — runs the others in order |
| `nuget-audit-discovery` | Inventories all projects, packages, and TFMs |
| `nuget-audit-platform` | Checks .NET platform lifecycle & EOL status |
| `nuget-audit-packages` | NuGet metadata: version, licence, age, complexity |
| `nuget-audit-risk` | CVEs, abandonment signals, community health |
| `nuget-audit-report` | Merges JSON → customer-facing markdown report |

> 6 skills · 11 files · ~1,500 lines of markdown instructions

<!-- Speaker notes: Each skill is completely standalone. Discovery doesn't know about risk. Risk doesn't know about the report. The orchestrator is the only one that knows the full picture. This makes them composable and reusable. -->

---

## The Audit Pipeline

<div class="mermaid">
flowchart TD
    O[nuget-audit-orchestrator] --> D[nuget-audit-discovery\ndiscovery.json]
    D --> P[nuget-audit-platform\nplatform.json]
    D --> PK[nuget-audit-packages\npackages.json]
    D --> R[nuget-audit-risk\nrisk.json]
    P --> REP[nuget-audit-report\nnuget-audit-result.json\nnuget-audit-report.md]
    PK --> REP
    R --> REP
</div>

<!-- Speaker notes: Discovery runs first. Then platform, packages, and risk all run in parallel — they're independent. Finally the report skill merges all three JSON outputs into the final result. -->

---

## Platform Health: Grade A ✅

| Metric | Value |
|--------|-------|
| Target Framework | `net10.0` |
| Release Type | **LTS** |
| Released | 2025-11-11 |
| End of Support | **2028-11-14** |
| Days Until EOL | ~985 |
| Projects | 76 of 76 |
| Status | ✅ **Current** |

> The solution is fully migrated to modern .NET — no .NET Framework projects remain.  
> ~2.7 years of platform support ahead.

<!-- Speaker notes: The great news first. They're on a current LTS, fully migrated. No platform action needed for nearly three years. This is where we should start the customer conversation — on a positive. -->

---

## The Dependency Landscape

<div class="columns">
<div>

### Direct packages: **115**

| Delta | Count |
|-------|-------|
| ✅ Current | 41 |
| 🔷 Patch | 29 |
| 🔶 Minor | 29 |
| 🔺 Major | 16 |
| Pre-release | 12 |

</div>
<div>

### Complexity breakdown

| Complexity | Count |
|------------|-------|
| NONE | 41 |
| LOW | 29 |
| MEDIUM | 29 |
| HIGH | 16 |

**260 transitive packages**  
(managed automatically — 0 CVEs)

</div>
</div>

<!-- Speaker notes: 41 packages are already at latest. The majority of outdated ones are patch or minor — those are quick wins. The 16 major upgrades are the ones that need thought. -->

---

## Risk Breakdown

<div class="columns">
<div>

### 🟢 The good news

| Signal | Status |
|--------|--------|
| Security vulnerabilities | **0** |
| Deprecated packages | **0** |
| .NET Framework projects | **0** |
| EOL platform | **0** |

The security posture is **clean**.

</div>
<div>

### 🟡 The watch list

| Risk | Count |
|------|-------|
| 🔴 CRITICAL | 0 |
| 🟠 HIGH | 1 |
| 🟡 MEDIUM | 83 |
| 🟢 LOW | 16 |
| ⚪ INFO | 15 |

83 MEDIUM = version drift + pre-release + unknown licences

</div>
</div>

<!-- Speaker notes: Zero CVEs is genuinely good news and should be front-and-centre. The 83 MEDIUM items sound alarming but most are a combination of version drift, unknown licence metadata, and wide blast radius. They're manageable. -->

---

## Pre-release Packages in Production

<!-- _class: small -->

12 packages running non-stable versions — including **unofficial branch builds**:

| Package | Installed | Stable Available? |
|---------|-----------|-----------------|
| `Serilog.Sinks.OpenTelemetry` | `4.2.1-nblumhardt-02317` | ⚠️ Unofficial personal branch |
| `MassTransit` | `9.0.0-develop.23` | ✅ `9.0.1` available |
| `MassTransit.EntityFrameworkCore` | `9.0.0-develop.23` | ✅ `9.0.1` available |
| `MassTransit.RabbitMQ` | `9.0.0-develop.23` | ✅ `9.0.1` available |
| `Metalama.Extensions.DependencyInjection` | `2026.0.5-preview` | ✅ `2026.0.16` available |
| `Azure.Storage.Blobs` | `12.27.0-beta.1` | ✅ `12.27.0` available |
| `Microsoft.Data.SqlClient` | `7.0.0-preview2.25289.6` | ❌ No stable yet |
| `OpenTelemetry.Instrumentation.*` | `1.x-beta.1` | ❌ No stable yet |

<!-- Speaker notes: The Serilog one is the most concerning. nblumhardt is the author's GitHub username — this is a personal branch build that was never released as an official stable version. It carries no stability guarantees and could be removed from NuGet at any time. MassTransit is simpler — stable is available, just needs upgrading. -->

---

## Notable Flags

<div class="columns">
<div>

### 🔺 coverlet.collector

**Installed**: `6.0.4`  
**Latest**: `8.0.0`  
**Gap**: 2 major versions  
**Impact**: Test infrastructure only

> Code coverage may report  
> incorrectly on .NET 10

**Upgrade effort**: 1–2 days  
(validation-heavy, not code-heavy)

</div>
<div>

### ❓ Unknown licences

~30 packages return no `licenseExpression`  
from the NuGet registry API.

**Microsoft packages** → MIT by policy  
(embedded in nuspec, not API field)

**Non-Microsoft packages** → manual  
verification needed:
- `AutoBogus`
- `JsonApiSerializer`
- `Bogus`, `OneOf`, `Moq`...

</div>
</div>

<!-- Speaker notes: The coverlet gap is annoying but low-risk in terms of production impact. The licence unknowns are a compliance concern — not a security one, but something legal may care about. -->

---

## The Cost of Delay

<!-- _class: invert -->

> Each release cycle that passes without updating  
> turns a **patch bump** into a **minor bump**,  
> and a minor bump into a **major**.

| Today | In 12 months (if ignored) |
|-------|--------------------------|
| 74 packages need updating | 90+ packages need updating |
| Most are patch/minor | Major bumps multiply |
| ~5–10 day effort | Multi-sprint initiative |
| One PR per family | Breaking changes compound |

**The upgrade tax is always cheaper now than later.**

<!-- Speaker notes: This is the pull-through argument. It's not that the work is urgent today — it's that the work gets harder every sprint you skip it. Patch bumps become minor bumps. Minor bumps become major bumps. Major bumps come with breaking changes that interact with each other. -->

---

## Recommended Remediation Roadmap

<!-- _class: small -->

| # | Action | Effort | Risk Reduction |
|---|--------|--------|----------------|
| 1 | Replace `Serilog.Sinks.OpenTelemetry` unofficial build | < 1 day | 🔴 High |
| 2 | Upgrade MassTransit family to stable `9.0.1` | 1 day | 🟠 High |
| 3 | Stabilise remaining pre-release packages | 1–2 days | 🟡 Medium |
| 4 | `coverlet.collector` 6 → 8 major upgrade | 1–2 days | 🟡 Medium |
| 5 | Patch/minor hygiene sweep (Microsoft.*, Aspire.*, EF Core) | 1–2 days | 🟢 Low |
| 6 | Verify licence compliance for unknown-licence packages | 0.5 day | 📋 Compliance |

**Total estimated effort: ~5–10 days**

> 💡 Steps 1–3 are the highest value. Steps 4–6 are hygiene — schedule as a recurring cadence.

---

## Overall Health Grade: **C**

<!-- _class: lead -->

> **Platform**: A&ensp;·&ensp;**Security**: A&ensp;·&ensp;**Packages**: C

The foundation is strong. The housekeeping is overdue.  
The remediation path is clear and bounded.

**This is a solvable problem — and the cost is known.**

<!-- Speaker notes: Grade C is not a failing grade. It means: good bones, some technical debt, clear path to improvement. The security A and platform A are the things to celebrate. The C is the opportunity. -->

---

<!-- _class: lead -->

# Act 3
## What We'd Build Next

---

## Where the Agent Struggled

The agent is excellent at **reasoning** but inconsistent at **determinism**.

<div class="columns">
<div>

### LLM heuristics (today)

- Guesses licence from URL patterns
- Estimates "stale" from months since publish
- Infers OSS health from general knowledge
- Version delta = text comparison

</div>
<div>

### Tools needed (tomorrow)

- **Fetch** the licence text and summarise it
- **Query** GitHub API for repo health metrics
- **Pull** NuGet download trends over time
- **Calculate** risk score deterministically

</div>
</div>

> Tools make the agent **precise** where it is currently **approximate**.

<!-- Speaker notes: This is the pattern we see over and over with agentic AI. The reasoning is strong, the data retrieval is weak. The solution is to give the agent tools — small, focused programs that return reliable data. -->

---

## Tool 1: Licence Resolver

A single-file C# app (`LicenceResolver.cs`):

```csharp
// dotnet script LicenceResolver.cs -- "https://opensource.org/licenses/MIT"
// Output: { "spdxId": "MIT", "summary": "...", "fullText": "..." }
```

**What it does**:
1. Fetches the URL from the `licenseUrl` field in NuGet registration metadata
2. Strips HTML, extracts licence text
3. Returns structured JSON — agent summarises in plain English

**Why it matters**: ~30 packages in the audit had `licenseUrl` but no `licenseExpression`.  
The agent had to guess. This tool would give it facts.

<!-- Speaker notes: Single-file C# with dotnet-script means no project files, no compilation step. The agent calls it like any shell command and gets structured output back. -->

---

## Tool 2: OSS Health Checker

A single-file C# app (`OssHealthChecker.cs`):

```csharp
// dotnet script OssHealthChecker.cs -- "https://github.com/JasperFx/marten"
// Output: { "riskScore": 12, "lastCommitDays": 3, "stars": 4100, 
//           "openIssues": 87, "archived": false, "contributors": 142 }
```

**Deterministic risk score (0–100)**:

| Signal | Weight |
|--------|--------|
| Last commit > 365 days | +30 |
| Archived repo | +40 |
| Stars < 100 | +15 |
| Contributors < 3 | +20 |
| Open issues > 500 | +10 |

<!-- Speaker notes: This replaces the current heuristic of "does the package seem popular?". The agent now gets a number with a breakdown. It can explain why a score is high or low rather than guessing. -->

---

## Tool 3: NuGet Download Trend

A single-file C# app (`NuGetDownloadTrend.cs`):

```csharp
// dotnet script NuGetDownloadTrend.cs -- "AutoBogus"
// Output: { "trend": "declining", "totalDownloads": 892341,
//           "last30Days": 1200, "prev30Days": 3800, "changePercent": -68 }
```

**Why download trends matter**:
- A package with **10M total downloads but declining** may be abandoned
- A package with **50K downloads and growing** may be a rising replacement
- The current audit had no way to distinguish — everything was "unknown adoption"

<!-- Speaker notes: NuGet does expose download statistics through their API. This tool pulls the last 30 days vs the previous 30 days and computes a trend signal. Combined with the OSS Health score, you get a much richer picture of abandonment risk. -->

---

## New Skills to Build

<div class="columns3">

<div>

### `nuget-audit-licences`

Full FOSS compliance analysis:

- GPL / AGPL detection
- Licence compatibility matrix
- Copyleft propagation risk
- Change detection between versions

</div>

<div>

### `nuget-audit-supply-chain`

Package integrity signals:

- NuGet package signing verification
- SourceLink presence check
- Reproducible builds flag
- Package author verification

</div>

<div>

### `nuget-audit-alternatives`

When a package is risky:

- Suggests maintained replacements
- Compares API surface compatibility
- Estimates migration effort
- Links to community migration guides

</div>
</div>

<!-- Speaker notes: These three skills extend the audit into compliance and security territory that the current skill set only touches on. nuget-audit-licences in particular is something every enterprise customer will ask for eventually. -->

---

<!-- _class: lead -->

# Act 4
## Full Disclosure

---

## AI Wrote the Skills

The `nuget-audit-*` family — **6 skills, 11 files, ~1,500 lines** — was generated by GitHub Copilot.

The human role was:

1. **Define the business outcome** — *"customers need to understand their dependency health"*
2. **Describe the shape of the output** — *"JSON + markdown, self-contained, sub-agent friendly"*
3. **Make design decisions when asked** — *"prefix: nuget-audit-; private feed: assume CLI configured"*
4. **Review and approve** — read the output, verify it made sense, move on

> The agent wrote the runbooks.  
> The human wrote the brief.

<!-- Speaker notes: This is the uncomfortable admission and the important one. We didn't hand-craft these skill files. We described what we needed and the agent produced them. We reviewed them. They were good. -->

---

## AI Wrote This Presentation

This deck was drafted by **GitHub Copilot** from:

- The audit output (`nuget-audit-report.md`)
- The session history (conversation turns with the agent)
- The Marp template conventions (from `.github/agents/marp.md`)
- The feedback you gave during planning

The human role was to **guide the narrative** across four planning rounds:

1. *"Add a primer on what skills are"*
2. *"Audience are developers unfamiliar with agentic tools"*
3. *"Add an act on what we'd build next"*
4. *"Disclose that AI wrote everything — and add the promise of a public repo"*

<!-- Speaker notes: The meta point: even this disclosure slide was written by the agent. The planning conversation shaped the content. The agent executed. This is the pattern. -->

---

<!-- _paginate: skip -->
<!-- _footer: "" -->
<!-- _class: lead invert -->

## Our Role as Developers

> We don't write the code.  
> **We write the brief.**  
> We approve the work.

Understand the **business outcome**.  
Guide the **agent toward it**.  
Review, steer, and ship.

---

<!-- _paginate: skip -->
<!-- _footer: "" -->
<!-- _class: lead -->

# Coming Soon

The `nuget-audit-*` skills and this presentation will be published to a **public GitHub repo**.

The deck will be **live on GitHub Pages** — Marp renders to HTML, deployed by a GitHub Actions workflow — shortly after this talk - and yes that was vibecoded as well :P

```
github.com/YOUR-ORG/nuget-audit-skills
```

![bg right opacity:0.4](https://images.unsplash.com/photo-1618401471353-b98afee0b2eb?w=800)

---

<!-- _paginate: skip -->
<!-- _footer: "" -->
<!-- _class: lead invert -->

# <!--fit--> Thank You

Questions?

<i class="fa-brands fa-github"></i> `github.com/YOUR-ORG/nuget-audit-skills`  
<i class="fa fa-window-maximize"></i> Slides live on GitHub Pages post-talk

