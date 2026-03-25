# .NET Modern Release Lifecycle Reference

Static reference table for `nuget-audit-platform`. Prefer the live `endoflife.date` API when available:
`GET https://endoflife.date/api/v1/products/dotnet/`

Use this table as a fast-path fallback when the API is unreachable or for offline use.
**Update this file when new .NET versions are released.**

---

## .NET Core / .NET 5+ Lifecycle

| Version | Type | Release Date | End of Support | Status (as of 2026-03) |
|---------|------|-------------|----------------|------------------------|
| .NET 10 | LTS | 2025-11-11 | 2028-05-12 | Current |
| .NET 9 | STS | 2024-11-12 | 2026-05-12 | Current |
| .NET 8 | LTS | 2023-11-14 | 2026-11-10 | Current |
| .NET 7 | STS | 2022-11-08 | 2024-05-14 | **EOL** |
| .NET 6 | LTS | 2021-11-08 | 2024-11-12 | **EOL** |
| .NET 5 | STS | 2020-11-10 | 2022-05-10 | **EOL** |
| .NET Core 3.1 | LTS | 2019-12-03 | 2022-12-13 | **EOL** |
| .NET Core 3.0 | STS | 2019-09-23 | 2020-03-03 | **EOL** |
| .NET Core 2.1 | LTS | 2018-05-30 | 2021-08-21 | **EOL** |

## Support Type Definitions

- **LTS (Long Term Support)**: Supported for 3 years from release date
- **STS (Standard Term Support)**: Supported for 18 months (pre-.NET 9) or 24 months (.NET 9+)
- Even-numbered releases (.NET 6, 8, 10…) are LTS
- Odd-numbered releases (.NET 7, 9, 11…) are STS

## .NET Standard

.NET Standard is **not a runtime** and has no EOL date. It is an API specification.

| Version | Compatible With |
|---------|----------------|
| netstandard2.0 | .NET Framework 4.6.1+, .NET Core 2.0+, .NET 5+ |
| netstandard2.1 | .NET Core 3.0+, .NET 5+ (NOT .NET Framework) |

> **Guidance**: .NET Standard 2.1 is the final standard version. New libraries should target
> `net8.0` or later. Existing netstandard2.0 libraries are fine to keep as-is for broad compatibility.

## Links

- Official .NET support policy: https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core
- Microsoft Lifecycle: https://learn.microsoft.com/en-us/lifecycle/products/microsoft-net-and-net-core
- endoflife.date API: https://endoflife.date/api/v1/products/dotnet/
