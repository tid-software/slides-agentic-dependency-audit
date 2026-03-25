# .NET Framework Legacy Lifecycle Reference

Static reference table for `nuget-audit-platform`. No API exists for .NET Framework lifecycle data.

Source: https://learn.microsoft.com/en-us/lifecycle/products/microsoft-net-framework

---

## Key Facts About .NET Framework

- **.NET Framework is Windows-only** — no cross-platform support
- **No new major versions planned** — 4.8.1 is the final version
- **Patch/security updates** are delivered via Windows Update (monthly cumulative updates)
- **Support is tied to the underlying Windows OS lifecycle**
- .NET Framework 4.x (4.5+) all coexist on the same machine; you may have multiple versions

---

## .NET Framework Version Lifecycle

| Version | Release Date | Mainstream Support End | Extended Support End | Status |
|---------|-------------|----------------------|---------------------|--------|
| 4.8.1 | 2022-08-09 | Tied to Windows OS | Tied to Windows OS | **Supported** |
| 4.8 | 2019-04-18 | Tied to Windows OS | Tied to Windows OS | **Supported** |
| 4.7.2 | 2018-04-30 | Tied to Windows OS | Tied to Windows OS | **Supported** |
| 4.7.1 | 2017-10-17 | 2023-01-10 | 2023-01-10 | **EOL** |
| 4.7 | 2017-04-05 | 2023-01-10 | 2023-01-10 | **EOL** |
| 4.6.2 | 2016-08-02 | 2023-01-10 | 2023-01-10 | **EOL** |
| 4.6.1 | 2015-11-30 | 2022-04-26 | 2022-04-26 | **EOL** |
| 4.6 | 2015-07-20 | 2022-04-26 | 2022-04-26 | **EOL** |
| 4.5.2 | 2014-05-05 | 2022-04-26 | 2022-04-26 | **EOL** |
| 4.5.1 | 2013-10-17 | 2016-01-12 | 2016-01-12 | **EOL** |
| 4.5 | 2012-08-15 | 2016-01-12 | 2016-01-12 | **EOL** |
| 4.0 | 2010-04-12 | 2016-01-12 | 2016-01-12 | **EOL** |
| 3.5 SP1 | 2008-11-19 | Tied to Windows OS | Tied to Windows OS | **Supported*** |
| 3.5 | 2007-11-19 | 2011-01-04 | 2011-01-04 | **EOL** |
| 3.0, 2.0, 1.x | Various | Long past | Long past | **EOL** |

> *3.5 SP1 remains supported as a Windows OS component on Windows 10/11, but is not recommended for new development.

---

## Supported Versions Table (Summary)

| TFM | Status |
|-----|--------|
| `net481` | ✅ Supported (latest) |
| `net48` | ✅ Supported |
| `net472` | ✅ Supported |
| `net471` | ❌ EOL (Jan 2023) |
| `net47` | ❌ EOL (Jan 2023) |
| `net462` | ❌ EOL (Jan 2023) |
| `net461` | ❌ EOL (Apr 2022) |
| `net46` | ❌ EOL (Apr 2022) |
| `net452` | ❌ EOL (Apr 2022) |
| `net45` and below | ❌ EOL |
| `net35` SP1 | ⚠️ Windows OS component only |

---

## Guidance for Agents

When a solution targets an EOL .NET Framework version, report:
```
Status: EOL — .NET Framework {version} reached end of support on {date}.
Recommendation: Upgrade to net481 (if staying on .NET Framework) or migrate to net8.0+ (recommended).
Migration effort: HIGH — requires testing on all dependent Windows OS versions.
```

When a solution targets a supported .NET Framework version:
```
Status: Supported (tied to Windows OS lifecycle).
Latest available: 4.8.1
Notes: No new features will be added to .NET Framework. Consider a migration timeline to modern .NET for
cross-platform capability, performance improvements, and ongoing feature investment.
```

---

## Detecting Installed Patches (Windows Registry)

```powershell
# Detect installed .NET Framework version
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP" -Recurse |
  Get-ItemProperty -Name Version, Release -ErrorAction SilentlyContinue |
  Where-Object { $_.PSChildName -eq "Full" } |
  Select-Object PSChildName, Version, Release

# Detect installed security updates
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Updates" |
  Where-Object { $_.Name -like "*.NET Framework*" } |
  Get-ChildItem | Get-ItemProperty -ErrorAction SilentlyContinue |
  Select-Object DisplayName, InstallDate
```

---

## Reference Links

- Microsoft Lifecycle (.NET Framework): https://learn.microsoft.com/en-us/lifecycle/products/microsoft-net-framework
- .NET Framework & Windows OS versions: https://learn.microsoft.com/en-us/dotnet/framework/install/versions-and-dependencies
- .NET Framework release notes: https://learn.microsoft.com/en-us/dotnet/framework/release-notes/release-notes
- Installed version detection: https://learn.microsoft.com/en-us/dotnet/framework/install/how-to-determine-which-net-framework-updates-are-installed
