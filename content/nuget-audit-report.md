# NuGet Dependency Audit Report

> **Solution**: stemuh  
> **Audit date**: March 4, 2026  
> **Generated**: 2026-03-04 17:28  
> **Overall Health Grade**: **C** — Moderate technical debt; no active security vulnerabilities but significant upgrade backlog.

---

## 1. Executive Summary

The mgnz-stemuh solution targets **.NET 10.0 (LTS)** across all 76 projects — a strong foundation with
approximately **2.7 years of platform support remaining** (EOL: 2028-11-14). There are **no known security
vulnerabilities** and **no deprecated packages**, which reflects well on the security hygiene of the solution.

However, **74 of 115 direct packages** are behind their latest stable version,
and **12 packages** are running pre-release builds in what may be production code paths. The
accumulation of version drift creates compounding upgrade cost: every release cycle that passes without updating
makes the next upgrade harder and riskier.

| Metric | Value |
|--------|-------|
| Overall Health Grade | **C** |
| Platform Status | ✅ Current (net10.0 LTS) |
| Security Vulnerabilities | ✅ None |
| Deprecated Packages | ✅ None |
| Outdated Packages | ⚠️ 74 / 115 |
| Pre-release in Use | ⚠️ 12 |
| HIGH risk packages | 🟠 1 |
| MEDIUM risk packages | 🟡 83 |
| LOW risk packages | 🟢 16 |
| Transitive packages | 260 |

> 💡 **Pull-through insight**: The cost of upgrading now is low — the team is on a current LTS platform,
> security is clean, and most upgrades are patch/minor bumps. Delaying increases the risk that future breaking
> changes compound, making a single "upgrade sprint" insufficient and requiring a longer remediation initiative.

---

## 2. Platform Health

| TFM | Type | Release | EOL | Days Until EOL | Status | LTS/STS |
|-----|------|---------|-----|----------------|--------|---------|
| net10.0 | Modern | 2025-11-11 | 2028-11-14 | ~985 | ✅ Current | LTS |

**All 76 projects** target `net10.0`. The solution is fully migrated to modern .NET — no .NET Framework
projects remain. Platform grade: **A**.

> The next LTS release (.NET 12) is expected in November 2027. Planning migration to .NET 11 (STS, Nov 2026)
> or waiting for .NET 12 (LTS, Nov 2027) is recommended as a medium-term roadmap item.

---

## 3. Dependency Health Overview

| Risk Level | Count | Meaning |
|------------|-------|---------|
| 🔴 CRITICAL | 0 | Known CVE — immediate action required |
| 🟠 HIGH | 1 | Major version drift, known abandonment |
| 🟡 MEDIUM | 83 | Pre-release, stale, licence unknown, wide blast radius |
| 🟢 LOW | 16 | Minor gaps, low risk |
| ⚪ INFO | 15 | Already at latest or no signals |

**Key signals driving MEDIUM ratings:**
- Pre-release packages in production code paths (MassTransit, Serilog, Metalama, Azure Storage)
- Unknown licence metadata for multiple packages (Microsoft packages are MIT by policy; others need verification)
- Wide blast radius: packages referenced by >3 projects amplify the impact of any update

---

## 4. Package Detail Table

> Sorted by risk level then name. Licence ❓ = not available from NuGet registry (manual verification needed).

| Risk | Package | Installed | Latest | Delta | Complexity | Licence | Pub Date |
|------|---------|-----------|--------|-------|------------|---------|----------|
| 🟠 HIGH | coverlet.collector | 6.0.4 | 8.0.0 | 🔺 Major | HIGH | MIT | 2026-02 |
| 🟡 MEDIUM | Aspire.Hosting.AppHost | 13.1.0 | 13.1.2 | 🔷 Patch | MEDIUM | MIT | 2026-02 |
| 🟡 MEDIUM | Aspire.Hosting.SqlServer | 13.1.0 | 13.1.2 | 🔷 Patch | MEDIUM | MIT | 2026-02 |
| 🟡 MEDIUM | Aspire.Microsoft.EntityFrameworkCore.SqlServer | 13.1.0 | 13.1.2 | 🔷 Patch | MEDIUM | MIT | 2026-02 |
| 🟡 MEDIUM | AutoBogus | 2.13.1 | 2.13.1 | ✅ Current | NONE | url:https://www… | 2021-07 |
| 🟡 MEDIUM | Azure.Storage.Blobs | 12.27.0-beta.1 | 12.27.0 | 🔶 Minor | MEDIUM | MIT | 2024-04 |
| 🟡 MEDIUM | Azure.Storage.Common | 12.26.0-beta.1 | 12.26.0 | 🔶 Minor | MEDIUM | MIT | 2024-07 |
| 🟡 MEDIUM | Azure.Storage.Files.Shares | 12.25.0-beta.1 | 12.25.0 | 🔶 Minor | MEDIUM | MIT | 2025-03 |
| 🟡 MEDIUM | Azure.Storage.Queues | 12.25.0-beta.1 | 12.25.0 | 🔶 Minor | MEDIUM | MIT | 2024-09 |
| 🟡 MEDIUM | BenchmarkDotNet | 0.15.8 | 0.15.8 | ✅ Current | NONE | MIT | 2023-10 |
| 🟡 MEDIUM | Bogus | 35.6.5 | 35.6.5 | ✅ Current | NONE | ❓ | ? |
| 🟡 MEDIUM | Dapper | 2.1.66 | 2.1.66 | ✅ Current | NONE | url:http://www.… | 2018-05 |
| 🟡 MEDIUM | FluentAssertions | 8.8.0 | 8.8.0 | ✅ Current | NONE | ❓ | ? |
| 🟡 MEDIUM | FluentValidation | 12.1.1 | 12.1.1 | ✅ Current | NONE | ❓ | ? |
| 🟡 MEDIUM | GitVersion.MsBuild | 6.5.1 | 6.6.0 | 🔶 Minor | HIGH | MIT | 2026-02 |
| 🟡 MEDIUM | Hangfire | 1.8.22 | 1.8.23 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Hangfire.AspNetCore | 1.8.22 | 1.8.23 | 🔷 Patch | LOW | url:https://raw… | 2021-04 |
| 🟡 MEDIUM | Hangfire.Core | 1.8.22 | 1.8.23 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Hangfire.SqlServer | 1.8.22 | 1.8.23 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Humanizer | 3.0.1 | 3.0.1 | ✅ Current | NONE | url:https://raw… | 2016-02 |
| 🟡 MEDIUM | JetBrains.Annotations | 2025.2.4 | 2025.2.4 | ✅ Current | NONE | MIT | 2023-10 |
| 🟡 MEDIUM | MassTransit | 9.0.0-develop.23 | 9.0.1 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | MassTransit.EntityFrameworkCore | 9.0.0-develop.23 | 9.0.1 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | MassTransit.RabbitMQ | 9.0.0-develop.23 | 9.0.1 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | MediatR | 14.0.0 | 14.1.0 | 🔶 Minor | HIGH | url:https://www… | 2026-03 |
| 🟡 MEDIUM | MediatR.Contracts | 2.0.1 | 2.0.1 | ✅ Current | NONE | Apache-2.0 | 2023-02 |
| 🟡 MEDIUM | Metalama.Extensions.DependencyInjection | 2026.0.5-preview | 2026.0.16 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Meziantou.Framework.StronglyTypedId | 2.3.9 | 2.3.11 | 🔷 Patch | MEDIUM | MIT | 2026-01 |
| 🟡 MEDIUM | Microsoft.AspNetCore.Authentication.JwtBearer | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.AspNetCore.Authentication.OpenIdConnect | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.AspNetCore.Mvc.Testing | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.AspNetCore.OpenApi | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | MIT | 2024-06 |
| 🟡 MEDIUM | Microsoft.AspNetCore.TestHost | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.CodeAnalysis.NetAnalyzers | 10.0.101 | 10.0.103 | 🔷 Patch | MEDIUM | MIT | 2025-08 |
| 🟡 MEDIUM | Microsoft.Data.SqlClient | 7.0.0-preview2.25289.6 | 7.0.0-preview2.25289.6 | ✅ Current | NONE | MIT | 2023-01 |
| 🟡 MEDIUM | Microsoft.EntityFrameworkCore | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.EntityFrameworkCore.Design | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.EntityFrameworkCore.Relational | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.EntityFrameworkCore.Sqlite | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.EntityFrameworkCore.Sqlite.Core | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.EntityFrameworkCore.SqlServer | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Caching.Memory | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Caching.StackExchangeRedis | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Configuration | 10.0.1 | 10.0.3 | 🔷 Patch | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Configuration.FileExtensions | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Configuration.Json | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.DependencyInjection | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Hosting.Abstractions | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Http | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Logging | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.Extensions.Logging.Abstractions | 10.0.1 | 10.0.3 | 🔷 Patch | LOW | ❓ | ? |
| 🟡 MEDIUM | Microsoft.IdentityModel.JsonWebTokens | 8.15.0 | 8.16.0 | 🔶 Minor | MEDIUM | MIT | 2023-09 |
| 🟡 MEDIUM | Microsoft.IdentityModel.Tokens | 8.15.0 | 8.16.0 | 🔶 Minor | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.NET.Test.Sdk | 18.0.1 | 18.3.0 | 🔶 Minor | HIGH | ❓ | ? |
| 🟡 MEDIUM | Microsoft.TestPlatform.ObjectModel | 18.0.1 | 18.3.0 | 🔶 Minor | MEDIUM | ❓ | ? |
| 🟡 MEDIUM | Microsoft.VisualStudio.Threading.Analyzers | 17.14.15 | 17.14.15 | ✅ Current | NONE | MIT | 2021-01 |
| 🟡 MEDIUM | Moq | 4.20.72 | 4.20.72 | ✅ Current | NONE | url:https://raw… | 2017-06 |
| 🟡 MEDIUM | Newtonsoft.Json | 13.0.4 | 13.0.4 | ✅ Current | NONE | MIT | 1900-01 |
| 🟡 MEDIUM | NodaTime | 3.2.2 | 3.3.1 | 🔶 Minor | HIGH | ❓ | ? |
| 🟡 MEDIUM | NodaTime.Testing | 3.2.2 | 3.3.1 | 🔶 Minor | HIGH | url:http://www.… | 2017-10 |
| 🟡 MEDIUM | Npgsql.EntityFrameworkCore.PostgreSQL | 10.0.0 | 10.0.0 | ✅ Current | NONE | ❓ | ? |
| 🟡 MEDIUM | Npgsql.EntityFrameworkCore.PostgreSQL.NodaTime | 10.0.0 | 10.0.0 | ✅ Current | NONE | PostgreSQL | 2022-02 |
| 🟡 MEDIUM | Nuke.Common | 10.1.0 | 10.1.0 | ✅ Current | NONE | ❓ | ? |
| 🟡 MEDIUM | OneOf | 3.0.271 | 3.0.271 | ✅ Current | NONE | url:https://git… | 2019-03 |
| 🟡 MEDIUM | OpenTelemetry.Exporter.OpenTelemetryProtocol | 1.14.0 | 1.15.0 | 🔶 Minor | HIGH | Apache-2.0 | 2024-05 |
| 🟡 MEDIUM | OpenTelemetry.Extensions.Hosting | 1.14.0 | 1.15.0 | 🔶 Minor | HIGH | Apache-2.0 | 2026-01 |
| 🟡 MEDIUM | OpenTelemetry.Instrumentation.AspNetCore | 1.14.0 | 1.15.0 | 🔶 Minor | HIGH | Apache-2.0 | 2026-01 |
| 🟡 MEDIUM | OpenTelemetry.Instrumentation.EntityFrameworkCore | 1.10.0-beta.1 | 1.10.0-beta.1 | ✅ Current | NONE | Apache-2.0 | 2026-01 |
| 🟡 MEDIUM | OpenTelemetry.Instrumentation.Http | 1.14.0 | 1.15.0 | 🔶 Minor | HIGH | Apache-2.0 | 2026-01 |
| 🟡 MEDIUM | OpenTelemetry.Instrumentation.Runtime | 1.14.0 | 1.15.0 | 🔶 Minor | HIGH | Apache-2.0 | 2026-01 |
| 🟡 MEDIUM | OpenTelemetry.Instrumentation.StackExchangeRedis | 1.14.0-beta.1 | 1.14.0-beta.1 | ✅ Current | NONE | Apache-2.0 | 2026-01 |
| 🟡 MEDIUM | Roslyn.Analyzers | 1.0.3.4 | 1.0.3.4 | ✅ Current | NONE | url:https://git… | 2017-08 |
| 🟡 MEDIUM | Roslynator.Analyzers | 4.14.1 | 4.15.0 | 🔶 Minor | HIGH | Apache-2.0 | 2025-01 |
| 🟡 MEDIUM | Scalar.AspNetCore | 2.11.1 | 2.12.54 | 🔶 Minor | HIGH | ❓ | ? |
| 🟡 MEDIUM | Serilog.AspNetCore | 10.0.0 | 10.0.0 | ✅ Current | NONE | Apache-2.0 | 2021-03 |
| 🟡 MEDIUM | Serilog.Enrichers.Span | 3.1.0 | 3.1.0 | ✅ Current | NONE | MIT | 2023-01 |
| 🟡 MEDIUM | Serilog.Exceptions | 8.4.0 | 8.4.0 | ✅ Current | NONE | MIT | 2022-08 |
| 🟡 MEDIUM | Serilog.Sinks.OpenTelemetry | 4.2.1-nblumhardt-02317 | 4.2.1-nblumhardt-02317 | ✅ Current | NONE | Apache-2.0 | 2024-05 |
| 🟡 MEDIUM | Serilog.Sinks.Seq | 9.0.0 | 9.0.0 | ✅ Current | NONE | ❓ | ? |
| 🟡 MEDIUM | Testcontainers | 4.9.0 | 4.10.0 | 🔶 Minor | HIGH | MIT | 2026-01 |
| 🟡 MEDIUM | Testcontainers.PostgreSql | 4.9.0 | 4.10.0 | 🔶 Minor | HIGH | MIT | 2026-01 |
| 🟡 MEDIUM | xunit.analyzers | 1.26.0 | 1.27.0 | 🔶 Minor | HIGH | Apache-2.0 | 2026-01 |
| 🟡 MEDIUM | xunit.runner.visualstudio | 3.1.5 | 3.1.5 | ✅ Current | NONE | MIT | 2023-07 |
| 🟡 MEDIUM | xunit.v3 | 3.2.1 | 3.2.2 | 🔷 Patch | MEDIUM | Apache-2.0 | 2026-01 |
| 🟢 LOW | Aspire.Hosting.Azure.Sql | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Hosting.PostgreSQL | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Hosting.RabbitMQ | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Hosting.Redis | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Hosting.Seq | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Hosting.Testing | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Microsoft.Data.SqlClient | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Npgsql.EntityFrameworkCore.PostgreSQL | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.Seq | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Aspire.StackExchange.Redis.OutputCaching | 13.1.0 | 13.1.2 | 🔷 Patch | LOW | MIT | 2026-02 |
| 🟢 LOW | Azure.Core | 1.50.0 | 1.51.1 | 🔶 Minor | MEDIUM | MIT | 2025-07 |
| 🟢 LOW | Microsoft.Extensions.Http.Resilience | 10.1.0 | 10.3.0 | 🔶 Minor | MEDIUM | MIT | 2026-02 |
| 🟢 LOW | Microsoft.Extensions.ServiceDiscovery | 10.1.0 | 10.3.0 | 🔶 Minor | MEDIUM | MIT | 2026-02 |
| 🟢 LOW | Npgsql.OpenTelemetry | 10.0.0 | 10.0.1 | 🔷 Patch | LOW | PostgreSQL | 2025-12 |
| 🟢 LOW | Testcontainers.Azurite | 4.9.0 | 4.10.0 | 🔶 Minor | MEDIUM | MIT | 2026-01 |
| 🟢 LOW | Testcontainers.MsSql | 4.9.0 | 4.10.0 | 🔶 Minor | MEDIUM | MIT | 2026-01 |
| ⚪ INFO | Aspire.Dashboard.Sdk.win-x64 | 13.1.0 | 13.1.0 | ✅ Current | NONE | MIT | 2026-02 |
| ⚪ INFO | Aspire.Hosting.Orchestration.win-x64 | 13.1.0 | 13.1.0 | ✅ Current | NONE | MIT | 2026-02 |
| ⚪ INFO | AspNetCore.HealthChecks.NpgSql | 9.0.0 | 9.0.0 | ✅ Current | NONE | Apache-2.0 | 2024-12 |
| ⚪ INFO | AspNetCore.HealthChecks.Rabbitmq | 9.0.0 | 9.0.0 | ✅ Current | NONE | Apache-2.0 | 2024-12 |
| ⚪ INFO | AspNetCore.HealthChecks.Redis | 9.0.0 | 9.0.0 | ✅ Current | NONE | Apache-2.0 | 2024-12 |
| ⚪ INFO | AspNetCore.HealthChecks.SqlServer | 9.0.0 | 9.0.0 | ✅ Current | NONE | Apache-2.0 | 2024-12 |
| ⚪ INFO | AspNetCore.HealthChecks.Uris | 9.0.0 | 9.0.0 | ✅ Current | NONE | Apache-2.0 | 2024-12 |
| ⚪ INFO | Azure.Data.Tables | 12.11.0 | 12.11.0 | ✅ Current | NONE | MIT | 2025-05 |
| ⚪ INFO | FluentAssertions.Analyzers | 0.34.1 | 0.34.1 | ✅ Current | NONE | MIT | 2024-10 |
| ⚪ INFO | Humanizer.Core | 3.0.1 | 3.0.1 | ✅ Current | NONE | MIT | 2025-11 |
| ⚪ INFO | JsonApiSerializer | 2.0.1 | 2.0.1 | ✅ Current | NONE | MIT | 2024-04 |
| ⚪ INFO | Microsoft.AspNetCore.App.Internal.Assets | 10.0.3 | 10.0.3 | ✅ Current | NONE | MIT | 2026-02 |
| ⚪ INFO | Microsoft.Bcl.HashCode | 6.0.0 | 6.0.0 | ✅ Current | NONE | MIT | 2024-11 |
| ⚪ INFO | Microsoft.Extensions.Azure | 1.13.1 | 1.13.1 | ✅ Current | NONE | MIT | 2025-11 |
| ⚪ INFO | OneOf.SourceGenerator | 3.0.271 | 3.0.271 | ✅ Current | NONE | MIT | 2024-05 |

---

## 5. Pre-Release Packages

The following direct packages are running pre-release (non-stable) versions. This is a risk signal because:
- Pre-release packages may have breaking changes without major version signals
- Some packages below are **unofficial branch builds**, not official releases

| Package | Installed | Stable Available | Recommended Action |
|---------|-----------|-----------------|-------------------|
| Azure.Storage.Blobs | 12.27.0-beta.1 | ✅ 12.27.0 | Upgrade to stable |
| Azure.Storage.Common | 12.26.0-beta.1 | ✅ 12.26.0 | Upgrade to stable |
| Azure.Storage.Files.Shares | 12.25.0-beta.1 | ✅ 12.25.0 | Upgrade to stable |
| Azure.Storage.Queues | 12.25.0-beta.1 | ✅ 12.25.0 | Upgrade to stable |
| MassTransit | 9.0.0-develop.23 | ✅ 9.0.1 | Upgrade to stable |
| MassTransit.EntityFrameworkCore | 9.0.0-develop.23 | ✅ 9.0.1 | Upgrade to stable |
| MassTransit.RabbitMQ | 9.0.0-develop.23 | ✅ 9.0.1 | Upgrade to stable |
| Metalama.Extensions.DependencyInjection | 2026.0.5-preview | ✅ 2026.0.16 | Upgrade to stable |
| Microsoft.Data.SqlClient | 7.0.0-preview2.25289.6 | ❌ None yet | Monitor for stable release |
| OpenTelemetry.Instrumentation.EntityFrameworkCore | 1.10.0-beta.1 | ❌ None yet | Monitor for stable release |
| OpenTelemetry.Instrumentation.StackExchangeRedis | 1.14.0-beta.1 | ❌ None yet | Monitor for stable release |
| Serilog.Sinks.OpenTelemetry | 4.2.1-nblumhardt-02317 | ❌ None yet | Monitor for stable release |

> **⚠️ Special note on Serilog.Sinks.OpenTelemetry 4.2.1-nblumhardt-02317**: This is an **unofficial personal
> branch build** (nblumhardt is the author's GitHub username). This should be treated as a MEDIUM-HIGH risk
> item and replaced with the official stable Serilog.Sinks.OpenTelemetry package as soon as possible.

> **⚠️ Special note on MassTransit 9.0.0-develop.23**: This is a developer branch snapshot. The stable
> release MassTransit 9.0.1 is available and should be adopted. Develop builds may contain incomplete
> features or unresolved issues.

---

## 6. Recommendations

### 🔺 Initiative: coverlet.collector major version upgrade (6.0.4 → 8.0.0)

The test coverage collector has two major versions of drift. While this doesn't affect production, it means
test infrastructure is running on an unsupported version and may produce incorrect coverage results on .NET 10.

**Recommended action**: Upgrade coverlet.collector and coverlet.msbuild (if used) to 8.x. Validate coverage
reports still produce expected output after upgrade. **Effort: 1–2 days** (including test validation).

### 🟡 Project: Stabilise pre-release package dependencies

The solution has **12 pre-release packages** in use, including production-path dependencies:

- **Azure.Storage.Blobs** `12.27.0-beta.1` — stable 12.27.0 available
- **Azure.Storage.Common** `12.26.0-beta.1` — stable 12.26.0 available
- **Azure.Storage.Files.Shares** `12.25.0-beta.1` — stable 12.25.0 available
- **Azure.Storage.Queues** `12.25.0-beta.1` — stable 12.25.0 available
- **MassTransit** `9.0.0-develop.23` — stable 9.0.1 available
- **MassTransit.EntityFrameworkCore** `9.0.0-develop.23` — stable 9.0.1 available
- **MassTransit.RabbitMQ** `9.0.0-develop.23` — stable 9.0.1 available
- **Metalama.Extensions.DependencyInjection** `2026.0.5-preview` — stable 2026.0.16 available
- **Microsoft.Data.SqlClient** `7.0.0-preview2.25289.6` — no stable version yet
- **OpenTelemetry.Instrumentation.EntityFrameworkCore** `1.10.0-beta.1` — no stable version yet
- **OpenTelemetry.Instrumentation.StackExchangeRedis** `1.14.0-beta.1` — no stable version yet
- **Serilog.Sinks.OpenTelemetry** `4.2.1-nblumhardt-02317` — no stable version yet

> ⚠️  Pre-release packages carry no stability guarantees and may receive breaking changes without major-version
> signalling. MassTransit 9.0.0-develop.23 is a developer branch build — the stable 9.0.1 should be adopted.
> Serilog.Sinks.OpenTelemetry 4.2.1-nblumhardt-02317 is an **unofficial branch build** and should be replaced
> with the official stable release as a priority.

**Recommended action**: Audit each pre-release package, adopt the stable equivalent where available.
**Effort: 1–3 days** (testing and validation per package).

### 🔷 Quick Wins: Patch/minor updates (low risk, high hygiene value)

The majority of outdated packages are **patch or minor** bumps, many in the Microsoft.AspNetCore.*,
Microsoft.EntityFrameworkCore.*, and Aspire.* families. These are well-tested, low-risk updates.

**Recommended action**: Group into a single 'dependency hygiene' PR. Estimate **1–2 days** including
CI validation. This is a high-ROI activity: it reduces risk accumulation and keeps the solution
current for future major upgrades.

### 🗓️ Suggested Upgrade Roadmap

| Priority | Action | Effort | Risk Reduction |
|----------|--------|--------|----------------|
| 1 | Replace unofficial pre-release packages (Serilog.Sinks.OTel, MassTransit develop) | 1 day | HIGH |
| 2 | Stabilise remaining pre-release packages | 2–3 days | MEDIUM |
| 3 | coverlet.collector major version upgrade | 1–2 days | LOW (test infra only) |
| 4 | Patch/minor hygiene sweep (Microsoft.*, Aspire.*, EF Core) | 1–2 days | MEDIUM |
| 5 | Verify licence compliance for unknown-licence packages | 0.5 day | Compliance |
| 6 | Plan .NET 11/12 migration (horizon planning) | Planning only | Future |

---

## 7. Appendix: Transitive Dependencies

The solution has **260 transitive (indirect) packages**. These are managed automatically by the NuGet
dependency resolver and are not directly referenced in project files.

| Signal | Count |
|--------|-------|
| Total transitive packages | 260 |
| Security vulnerabilities | 0 |
| Deprecated transitives | 0 |

No action is required on transitive packages at this time. They will be updated automatically when direct
package upgrades are performed.

---

## 8. Appendix: Licence Notes

Several Microsoft packages show "unknown" licence in the NuGet registry metadata. This is a known NuGet
packaging behaviour where Microsoft embeds licence information in the nuspec file rather than the
licenseExpression field. These packages are MIT licensed by policy:

- Microsoft.AspNetCore.* — MIT
- Microsoft.EntityFrameworkCore.* — MIT
- Microsoft.Extensions.* — MIT
- System.* — MIT

For non-Microsoft packages showing "unknown" licence, manual verification against the package's GitHub
repository or NuGet.org listing is recommended.

---

*This report was generated by the nuget-audit-orchestrator skill.*  
*Data sources: dotnet CLI, NuGet V3 Registration API, endoflife.date API*
