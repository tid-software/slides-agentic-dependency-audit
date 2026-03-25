---
name: nuget-audit-platform
description: >
  .NET platform lifecycle analysis skill. For each distinct target framework in a solution, determines
  version currency, LTS vs STS classification, end-of-support date, and days until EOL. Covers both
  modern .NET (5+) and legacy .NET Framework. Produces platform.json. Does NOT depend on any other
  project skills.
---

# NuGet Audit — Platform

Analyses the .NET runtime/SDK platform health for every distinct Target Framework Moniker (TFM) in the solution.

## When to Use This Skill

Invoke as part of **Phase 2** of `nuget-audit-orchestrator` after `nuget-audit-discovery` completes.
Input: `discovery.json`. Output: `platform.json`.

---

## Input

Read `distinctTfms` from `discovery.json`. Process each unique TFM.

---

## Classification

Determine platform type from TFM string:

| TFM pattern | Type |
|-------------|------|
| `net5.0` and later (`net6.0`, `net7.0`, `net8.0`, `net9.0`, `net10.0`…) | `modern` |
| `netcoreapp2.x`, `netcoreapp3.x` | `modern-legacy` |
| `netstandard1.x`, `netstandard2.x` | `standard` |
| `net40`–`net48`, `net481` | `legacy-framework` |

---

## Modern .NET (net5.0+, netcoreapp*, netstandard*)

### Data Source

Use the **endoflife.date API** — the only reliable programmatic source for .NET lifecycle dates:

```
GET https://endoflife.date/api/v1/products/dotnet/
```

Response is an array of release objects. Key fields:

```json
[
  {
    "cycle": "10",
    "releaseDate": "2025-11-11",
    "eol": "2028-05-12",
    "latest": "10.0.0",
    "latestReleaseDate": "2025-11-11",
    "lts": true,
    "support": "2028-05-12"
  }
]
```

- `cycle` = major version number
- `lts: true` = LTS (3-year support); `lts: false` = STS (18-month support for older, 24-month from .NET 9+)
- `eol` = end-of-support date
- `latest` = latest patch version

### Mapping TFM to Cycle

| TFM | Cycle |
|-----|-------|
| `net8.0` | `"8"` |
| `net9.0` | `"9"` |
| `netcoreapp3.1` | `"3.1"` |
| `netstandard2.1` | N/A — see note below |

> **netstandard**: .NET Standard is not a runtime and has no EOL. Flag it as `type: standard` with a
> note: "Consider targeting net8.0 or later for new libraries — .NET Standard 2.1 is the last standard version."

### Status Rules

Compute `status` from today's date and `eol`:

| Condition | Status |
|-----------|--------|
| `eol` date has passed | `eol` |
| `eol` date within 6 months | `warn` |
| Otherwise | `current` |

Compute `daysUntilEol` as `(eol date − today)` in whole days (negative = already past EOL).

---

## Legacy .NET Framework (net40–net481)

### Data Source

No API exists. Fetch the Microsoft Lifecycle page and parse:

```
GET https://learn.microsoft.com/en-us/lifecycle/products/microsoft-net-framework
```

Alternatively, use the static reference table in [`references/netfx-lifecycle.md`](references/netfx-lifecycle.md).

### .NET Framework Lifecycle Notes

- .NET Framework is a **Windows-only, closed platform** — no new major versions will be released
- Microsoft ships monthly **cumulative updates** via Windows Update; these are security/reliability patches, NOT new features
- The latest version is **.NET Framework 4.8.1** (released August 2022)
- All versions from 4.x share the same support policy as the underlying Windows OS

**Key versions:**

| Version | Status | Notes |
|---------|--------|-------|
| net45, net46, net461 | **EOL** | No longer supported |
| net462, net47, net471 | **EOL** | No longer supported |
| net472, net48, net481 | **Supported** | Support tied to Windows OS lifecycle |

### Patch Status Detection

To detect whether the latest cumulative updates are applied (local machine):

```powershell
# Check installed .NET Framework version from registry
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP" -Recurse |
  Get-ItemProperty -Name Version, Release -ErrorAction SilentlyContinue |
  Where-Object { $_.PSChildName -eq "Full" } |
  Select-Object PSChildName, Version, Release

# Check installed security updates
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Updates\.NET Framework*" |
  Get-ChildItem | Get-ItemProperty | Select-Object DisplayName, InstallDate
```

---

## Output: `platform.json`

```json
{
  "analysedAt": "ISO-8601 timestamp",
  "platforms": [
    {
      "tfm": "net8.0",
      "type": "modern",
      "cycle": "8",
      "releaseType": "LTS",
      "installedVersion": "8.0.x",
      "latestVersion": "8.0.x",
      "releaseDate": "2023-11-14",
      "endOfSupport": "2026-11-10",
      "daysUntilEol": 250,
      "status": "current",
      "referencedByProjects": ["MyApp", "MyApp.Api"]
    },
    {
      "tfm": "net48",
      "type": "legacy-framework",
      "cycle": "4.8",
      "releaseType": "legacy",
      "installedVersion": "4.8",
      "latestVersion": "4.8.1",
      "endOfSupport": "tied-to-windows-os",
      "status": "current",
      "notes": "Latest version is 4.8.1. Consider migrating to modern .NET for cross-platform support and improved performance.",
      "referencedByProjects": ["LegacyService"]
    }
  ],
  "overallPlatformStatus": "current | warn | eol"
}
```

`overallPlatformStatus` = worst status across all platforms.

---

## Agent Checklist

- [ ] `endoflife.date` API called for modern .NET TFMs
- [ ] `daysUntilEol` computed correctly from today's date
- [ ] Legacy .NET Framework TFMs looked up from static table or MS Lifecycle page
- [ ] `netstandard` TFMs flagged with guidance note, not as EOL
- [ ] `referencedByProjects` populated from `discovery.json`
- [ ] `overallPlatformStatus` set to worst across all platforms
- [ ] Output saved to `{outputPath}/platform.json`
