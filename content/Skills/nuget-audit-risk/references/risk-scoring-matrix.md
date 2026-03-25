# Risk Scoring Matrix

Reference table for `nuget-audit-risk`. Use this to assign consistent `riskScore` values.

Apply rules **in order** — the **first matching rule** determines the score.

---

## Scoring Rules (Highest to Lowest Priority)

### CRITICAL

| Rule | Signal(s) |
|------|-----------|
| R-C1 | `vulnerability.severity == "critical"` on a direct package |
| R-C2 | `vulnerability.severity == "high"` on a direct package |
| R-C3 | `vulnerability.severity == "critical"` on a transitive package |

> A CRITICAL finding means there is a known security exploit. This should be surfaced immediately
> regardless of other package signals.

---

### HIGH

| Rule | Signal(s) |
|------|-----------|
| R-H1 | `vulnerability.severity == "high"` on a transitive package |
| R-H2 | `vulnerability.severity == "moderate"` on a direct package |
| R-H3 | `deprecated == true` AND no `alternativePackage` provided |
| R-H4 | `deprecated == true` AND `alternativePackage` provided (still HIGH — requires migration) |
| R-H5 | `sourceRepoArchived == true` (GitHub repo archived) |
| R-H6 | Installed version is **> 2 major versions** behind latest (e.g. installed v1.x, latest v4.x) |
| R-H7 | `frameworkCompatible == false` for the latest version (can't upgrade without TFM change) |

---

### MEDIUM

| Rule | Signal(s) |
|------|-----------|
| R-M1 | `vulnerability.severity == "moderate"` on a transitive package |
| R-M2 | `vulnerability.severity == "low"` on a direct package |
| R-M3 | `stalenessMonths > 24` (last release was > 2 years ago) |
| R-M4 | `licence.changed == true` (licence changed between installed and latest version) |
| R-M5 | `lowAdoption == true` (total download count < 10,000) |
| R-M6 | Installed version is **1-2 major versions** behind latest |
| R-M7 | `deprecated == true` on a transitive package (indirect exposure) |

---

### LOW

| Rule | Signal(s) |
|------|-----------|
| R-L1 | `vulnerability.severity == "low"` on a transitive package |
| R-L2 | Only patch version behind (same major.minor, lower patch) |
| R-L3 | Only minor version behind (same major, lower minor) with no other signals |
| R-L4 | `stalenessMonths` between 12 and 24 (getting old but not critical) |

---

### OK

| Rule | Signal(s) |
|------|-----------|
| R-O1 | No signals triggered — package is current, no vulnerabilities, no deprecation |

---

## Multi-Signal Examples

When multiple signals apply, the **highest matching** rule wins. Record ALL triggered signals
in `riskFactors`.

| Scenario | Signals | Score |
|----------|---------|-------|
| Package has moderate CVE + is deprecated | R-C2 misses (severity moderate not high direct), R-H2 + R-H3 | **HIGH** |
| Package is 3 major versions behind + stale | R-H6 + R-M3 | **HIGH** (R-H6 wins) |
| Package has licence change + stale 30mo | R-M4 + R-M3 | **MEDIUM** (both M-level) |
| Package is 1 patch behind, no other issues | R-L2 | **LOW** |
| Transitive dep has high CVE | R-H1 | **HIGH** |
| Package is current but low adoption | R-M5 | **MEDIUM** |

---

## Risk Score → Business Narrative

Use this table when generating the `nuget-audit-report.md` recommendations:

| Score | Business Impact | Urgency | Effort Tier |
|-------|----------------|---------|-------------|
| CRITICAL | Active security risk — potential data breach or system compromise | Immediate (< 1 sprint) | Quick Win or Project |
| HIGH | Significant security exposure or blocked upgrade path | Short-term (1-2 sprints) | Project |
| MEDIUM | Technical debt accumulating; compliance or licence risk | Medium-term (next quarter) | Project or Initiative |
| LOW | Hygiene — small improvements, low risk | Backlog | Quick Win |
| OK | No action required | — | — |

---

## Effort Tier Definitions

| Tier | Description | Typical Effort |
|------|-------------|---------------|
| **Quick Win** | Patch or minor bump; low blast radius (1-2 projects); automated testing confirms no breaks | < 1 day |
| **Project** | Major version bump or deprecated package replacement; moderate blast radius; requires testing | 1-5 days |
| **Initiative** | Platform migration (e.g. .NET Framework → .NET 8); systemic licence changes; framework incompatibility | Weeks to months |
