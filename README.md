# PrivLens

**See your Active Directory the way an attacker does.**

PrivLens is a single-binary, read-only Active Directory security scanner for Windows. It finds high-impact misconfigurations that attackers routinely exploit — Kerberoastable service accounts, weak password policy, excessive Domain Admins, and more — and produces a clean, client-ready PDF report in seconds.

Built for IT admins, security teams, MSPs, and consultants who need **fast, focused, actionable AD assessment** without installing agents, creating accounts, or sending data off the network.

> Product website: [privlens.com](https://privlens.com) (marketing site in `../privlens_website`)

---

## What PrivLens does

PrivLens connects to your domain using the **current Windows user's credentials** (SSPI/Kerberos), runs a curated set of **read-only LDAP checks**, and writes a **self-contained PDF report** locally. Each finding includes plain-English detail and a concrete remediation step.

| Capability | Detail |
|------------|--------|
| **Read-only** | Never modifies Active Directory — search-only LDAP access |
| **Air-gapped safe** | No HTTP client, telemetry, update checks, or cloud sync |
| **Fast** | Most domains complete in under a minute |
| **Shareable output** | PDF report ready for clients, auditors, or internal review |
| **Graceful degradation** | Checks that cannot run are marked *skipped*; the scan continues |

### What it does **not** do

- Change anything in the directory
- Upload or transmit scan data
- Require a signup, account, or internet connection
- Replace a full penetration test (it focuses on common, high-impact misconfigs)

---

## How it works

```
Download → Run on a domain-joined Windows machine → Review privlens-report.pdf
```

1. **Discover** — Reads `USERDNSDOMAIN` (or `-domain`) and locates a domain controller via DNS SRV.
2. **Connect** — Binds with your Windows token over **LDAPS (636)** first; falls back to LDAP/389 automatically if LDAPS is unavailable (with a warning).
3. **Scan** — Runs edition-gated security checks in deterministic order against the directory.
4. **Report** — Aggregates results into a PDF with coverage summary, severity counts, and per-finding remediation.

```
PS C:\Tools> .\privlens.exe -company "Contoso Ltd"

  PrivLens v1.0
  Organization:        Contoso Ltd
  Edition:             Community
  ...
  Running 7 check(s) [Community edition]...
  Report written to .\privlens-report.pdf
```

The PDF includes your organization name in the header and on every page footer, plus domain metadata, check coverage, and findings grouped by check.

---

## Editions

PrivLens ships in four editions. See `docs/PrivLens_Pricing_Comparison.pdf` for the full rule catalog.

| Edition | Price | Checks | License |
|---------|-------|--------|---------|
| **Community** | Free forever | 7 | `-company` required |
| **Professional** | $599 / 5 years | 30 | Signed `privlens.license` |
| **Enterprise** | Contact us | 30+ | Signed `privlens.license` (`edition: enterprise`) |
| **MSP** | Contact us | 30+ | Signed `privlens.license` (`edition: msp`) |

### Scan scope by edition

Scan breadth is determined automatically from your edition (not configurable on the command line):

| Edition | Scope | LDAP breadth |
|---------|-------|--------------|
| **Community** | `domain` | Current domain only (single `defaultNamingContext`) |
| **Professional** | `tree` | Domain tree rooted at joined or `-domain` FQDN |
| **Enterprise / MSP** | `forest` | All in-forest domains (external/trust domains excluded) |

**Community (default)** — no license file. Pass `-company "Your Organization"`; that name appears on the report. Runs 7 checks: domain admin exposure, privileged password-never-expires, inactive privileged accounts, service accounts with admin rights, weak password policy, basic unconstrained delegation, and high-risk operator group health.

**Professional / Enterprise** — vendor-signed `privlens.license`. The license `customer_name` appears on the report. Do not pass `-company` with these licenses.

**MSP (white-label)** — vendor-signed `privlens.license` with `"edition": "msp"`. MSP is the only edition that may combine a license with `-company`: use `-company` for the end-customer name on the report while the license unlocks the MSP tier. Without `-company`, the license `customer_name` is used.

If neither a valid license nor `-company` is provided, PrivLens exits before scanning.

Licensing details: [`docs/licensing.md`](docs/licensing.md)

### Catalog overview (30 checks)

| Tier | Count | Examples |
|------|-------|----------|
| Community (7) | CHK-001–004, CHK-011, CHK-025, CHK-030 | Domain admins, priv pwd never expires, inactive priv, svc admin rights, password policy, basic delegation, operator group health |
| Professional+ (23) | CHK-005–010, CHK-012–024, CHK-026–029 | Nested groups, stale priv, Kerberoastable SPNs, dormant accounts, etc. |

See `docs/PrivLens_Pricing_Comparison.pdf` for the full catalog names.

---

## Quick start

### Requirements

- **Windows** 10, 11, or Server 2016+
- Machine **joined to the domain** being scanned
- Network reachability to a domain controller (LDAPS **636** preferred)
- Standard domain user credentials (elevated rights help some checks but are not required)

### Community edition

```powershell
.\privlens.exe -company "Acme Corporation" -out .\privlens-report.pdf
```

### Professional edition

```powershell
# privlens.license must be present (signed JSON — see docs/licensing.md)
.\privlens.exe -out .\privlens-report.pdf
```

### Enterprise edition

```powershell
# privlens.license with "edition": "enterprise"
.\privlens.exe -out .\privlens-report.pdf
```

### MSP edition (white-label)

```powershell
# privlens.license with "edition": "msp"
# Report shows license customer_name unless -company names the end customer:
.\privlens.exe -company "End Customer Inc." -out .\privlens-report.pdf
```

### Command-line options

| Flag | Description |
|------|-------------|
| `-company` | Organization on the report. Required for Community. **MSP only:** may be used with a license for white-label end-customer branding. Not allowed with Professional or Enterprise licenses. |
| `-out` | Output PDF path (default: `./privlens-report.pdf`) |
| `-license` | Path to license file (default: `./privlens.license`) |
| `-domain` | Override auto-discovered DNS domain (**Professional+ only**; not available in Community) |
| `-version` | Print version and exit |

---
