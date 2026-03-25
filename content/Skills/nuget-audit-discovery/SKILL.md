---
name: nuget-audit-discovery
description: >
  .NET dependency discovery skill. Inventories all projects, target frameworks, direct NuGet packages,
  and transitive NuGet packages in a .NET solution. Produces discovery.json consumed by other
  nuget-audit-* skills. Does NOT depend on any other project skills.
---

# NuGet Audit — Discovery

Produces the raw dependency inventory for the solution. All other `nuget-audit-*` analysis skills
consume `discovery.json` produced here.

## When to Use This Skill

Invoke as **Phase 1** of `nuget-audit-orchestrator`, or standalone when you only need a dependency inventory without full analysis.

---

## Prerequisites

```bash
dotnet --version          # Confirm dotnet CLI is available
dotnet nuget list source  # Confirm feed(s) configured (public and/or private)
dotnet restore            # Ensure packages are restored before listing
```

---

## Step 1: List All Projects

```bash
# From the solution root
dotnet sln list

# If using a directory (no .sln), find all .csproj files
Get-ChildItem -Recurse -Filter "*.csproj" | Select-Object -ExpandProperty FullName
# or on Linux/macOS:
find . -name "*.csproj"
```

Apply any `projectFilter` glob at this step if provided.

---

## Step 2: Extract Target Frameworks

For each `.csproj` found, read the `<TargetFramework>` or `<TargetFrameworks>` element:

```bash
# PowerShell — extract TFMs from all csproj files
Get-ChildItem -Recurse -Filter "*.csproj" |
  Select-String -Pattern "<TargetFramework[s]?>(.+?)<" |
  ForEach-Object { $_.Matches.Groups[1].Value }

# Or use grep
grep -rh "<TargetFramework" . --include="*.csproj" | grep -oP '(?<=>)[^<]+'
```

**Classification rules:**

| TFM pattern | Classification | Examples |
|-------------|---------------|---------|
| `net5.0`, `net6.0` … `net9.0`, `net10.0` | **modern** | `net8.0`, `net9.0` |
| `netcoreapp2.x`, `netcoreapp3.x` | **modern-legacy** | `netcoreapp3.1` |
| `netstandard1.x`, `netstandard2.x` | **standard** | `netstandard2.1` |
| `net40`, `net45`, `net46`, `net47`, `net48`, `net481` | **legacy-framework** | `net48`, `net481` |

---

## Step 3: List Direct Packages

```bash
# JSON output (requires .NET SDK 7+)
dotnet list package --format json

# If --format json is unavailable, use plain output:
dotnet list package
```

For each project, this gives: package id + resolved version.

---

## Step 4: List Transitive Packages

```bash
dotnet list package --include-transitive --format json
```

Record which packages are **direct** vs **transitive** per project. A package is transitive if it appears only in `--include-transitive` output and not in the direct list.

---

## Step 5: Build `discovery.json`

Assemble all collected data into the following structure:

```json
{
  "collectedAt": "ISO-8601 timestamp",
  "solution": "relative/path/to/solution.sln",
  "projects": [
    {
      "name": "MyApp",
      "path": "src/MyApp/MyApp.csproj",
      "targetFrameworks": ["net8.0"],
      "classification": "modern",
      "directPackages": [
        { "id": "Newtonsoft.Json", "version": "12.0.3" },
        { "id": "Serilog", "version": "3.0.0" }
      ],
      "transitivePackages": [
        { "id": "Serilog.Sinks.File", "version": "5.0.0", "pulledInBy": ["Serilog"] }
      ]
    }
  ],
  "distinctTfms": ["net8.0"],
  "allDirectPackages": [
    { "id": "Newtonsoft.Json", "version": "12.0.3", "referencedByProjects": ["MyApp"] }
  ],
  "allTransitivePackages": [
    { "id": "Serilog.Sinks.File", "version": "5.0.0", "referencedByProjects": ["MyApp"] }
  ]
}
```

**Notes:**
- `allDirectPackages` deduplicates across projects; `referencedByProjects` is the list of all projects using it
- `pulledInBy` in transitive entries should identify the direct package(s) that introduce the transitive dep where inferable from the restore graph
- `referencedByProjects` count is used by `nuget-audit-packages` for blast radius / upgrade complexity

---

## Agent Checklist

- [ ] `dotnet restore` ran successfully before listing
- [ ] All `.csproj` files parsed for `<TargetFramework>` / `<TargetFrameworks>`
- [ ] Multi-targeting projects (`<TargetFrameworks>` plural) captured as an array
- [ ] Direct vs transitive distinction recorded per project
- [ ] `discovery.json` saved to `{outputPath}/discovery.json`
- [ ] `distinctTfms` array contains only unique TFM strings

---

## Common Issues

| Problem | Resolution |
|---------|------------|
| `--format json` not supported | Use plain `dotnet list package` and parse the tabular output |
| Packages not listed (restore failed) | Run `dotnet restore` first; check feed auth with `dotnet nuget list source` |
| Multi-targeting project | Split into one entry per TFM in `targetFrameworks` array; packages are shared unless conditional |
| Solution file not found | Fall back to discovering all `.csproj` files from directory root |
