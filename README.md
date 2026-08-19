# 🚀 M365 Auth Modules

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-blue.svg)
![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey.svg)
![Graph](https://img.shields.io/badge/API-Microsoft%20Graph-0078D6.svg)
![EXO](https://img.shields.io/badge/Exchange-Online-0078D4.svg)
![Intune](https://img.shields.io/badge/Intune-Microsoft%20Graph-0078D6.svg)
![Teams](https://img.shields.io/badge/Teams-Microsoft%20Graph-6264A7.svg)
![SharePoint](https://img.shields.io/badge/SharePoint-Microsoft%20Graph-038387.svg)
![Version](https://img.shields.io/badge/version-1.1-green.svg)

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-☕-FFDD00?style=for-the-badge)](https://www.buymeacoffee.com/mabdulkadrx)

---

## 📖 Overview

**M365 Auth Modules** is a collection of reusable, production-ready PowerShell modules for **Microsoft 365 authentication and Azure AD automation**. It provides centralized OAuth 2.0 and certificate-based authentication for **Intune**, **Exchange Online**, **Teams**, **SharePoint**, and **Microsoft Graph** — eliminating the need to embed credentials or auth logic in every script.

> 🔐 **All authentication uses OAuth 2.0 and certificate-based authentication** — no Basic Auth, no SMTP, no legacy protocols. This is future-proof: Microsoft is deprecating SMTP AUTH Basic Authentication (disabled by default Dec 2026, final removal H2 2027). Every module in this project is built on modern, secure authentication from day one.

**Authentication methods used:**

| Method | Modules | Protocol |
|--------|---------|----------|
| **OAuth 2.0 Client Credentials** | Connect-Intune, Send-ExchangeMail, Connect-Teams, Connect-SharePoint, Connect-MgGraph | Microsoft Graph API via `login.microsoftonline.com` |
| **Certificate-based Auth** | Connect-ExchangeOnline | `ExchangeOnlineManagement` module with X.509 certificate |
| **Interactive Delegated** | All modules (optional) | Browser-based OAuth consent flow |

Each module is self-contained: its code, documentation, and examples live together in a single file. No cross-folder dependencies, no scattered files.

```
                 M365 Auth Modules
                        │
     ┌──────────┬───────┼───────┬──────────┐
     │          │       │       │          │
     ▼          ▼       ▼       ▼          ▼
┌─────────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌──────────┐
│ INTUNE  │ │EXO  │ │TEAMS│ │ SPO │ │   MG     │
│ Module  │ │     │ │     │ │     │ │  GRAPH   │
│         │ │Cert │ │Chan │ │List │ │          │
│ Devices │ │Auth │ │nels │ │Items│ │ Users    │
│ Sync    │ │IPPS │ │Memb │ │Files│ │ Groups   │
│ Policy  │ │     │ │ers  │ │     │ │ Devices  │
└─────────┘ └─────┘ └─────┘ └─────┘ └──────────┘
```

---

## 🗂 Folder Structure

```
Modules-GitHub/
│
├── README.md                               # 📖 This file
├── SETUP.md                                # 🛠 Azure AD App Registration guide
├── CHANGELOG.md                            # 📋 Version history
├── LICENSE                                 # ⚖ MIT License
├── .gitignore                              # 🔒 Prevents secrets from being committed
│
├── Connect-Intune.psm1                     # 📦 Intune/Graph connection module
├── Connect-Intune.md                       # 📖 Intune module documentation
│
├── Send-ExchangeMail.psm1                  # 📦 Email via Microsoft Graph module
├── Send-ExchangeMail.md                    # 📖 Email module documentation
│
├── Connect-ExchangeOnline.psm1             # 📦 Exchange Online connection module
├── Connect-ExchangeOnline.md               # 📖 Exchange Online module documentation
│
├── Connect-Teams.psm1                      # 📦 Microsoft Teams management module
├── Connect-Teams.md                        # 📖 Teams module documentation
│
├── Connect-SharePoint.psm1                 # 📦 SharePoint Online management module
├── Connect-SharePoint.md                   # 📖 SharePoint module documentation
│
├── Connect-MgGraph.psm1                    # 📦 General Microsoft Graph module
└── Connect-MgGraph.md                      # 📖 Microsoft Graph module documentation
```

---

## ✨ Core Features

### 🔹 Connect-Intune (`Connect-Intune.psm1`)

Authenticate to Microsoft Graph with Intune-specific permission scopes. Supports both **app-only** (client credentials) and **interactive** (delegated) authentication.

**`Connect-Intune` / `Disconnect-Intune`:**
- OAuth 2.0 client credentials flow (app-only) or interactive delegated login
- Auto-installs `Microsoft.Graph.Authentication` from PSGallery
- Friendly AADSTS error mapping — `AADSTS700016` / `7000215` translated to plain English

**`Get-IntuneDevices` / `Send-IntuneDeviceSync` / `Get-IntunePolicy`:**
- Lists managed devices with auto-pagination via `@odata.nextLink`
- Server-side OData `-Filter` (`deviceName eq 'PC-001'`)
- Sends `syncDevice` command to a device by GUID
- Filters device configuration policies by name substring

> 📖 Docs: [`Connect-Intune.md`](Connect-Intune.md) · ⚙ Setup: [`SETUP.md` §1](SETUP.md#1️⃣-connect-intune-app-registration)

### 🔹 Send-ExchangeMail (`Send-ExchangeMail.psm1`)

Send HTML email through Microsoft Graph API using OAuth 2.0 client credentials (app-only). **No SMTP**, **no Basic Auth**, **no legacy protocols** — future-proof against the Microsoft SMTP AUTH deprecation.

**`Send-ExchangeMail`:**
- HTML body, CC/BCC recipient arrays, file attachments as `fileAttachment`
- BSTR-safe SecureString handling — `ZeroFreeBSTR` in `finally`
- Multi-source secret resolution: `-ClientSecret` → `$env:EXCHANGE_CLIENT_SECRET` → module default
- `-SaveToSentItems` switch for audit trail compliance

> 📖 Docs: [`Send-ExchangeMail.md`](Send-ExchangeMail.md) · ⚙ Setup: [`SETUP.md` §2](SETUP.md#2️⃣-send-exchangemail-app-registration)

### 🔹 Connect-ExchangeOnline (`Connect-ExchangeOnline.psm1`)

Connect to **Exchange Online (EXO)** and the **Security & Compliance Center (IPPS)** using the `ExchangeOnlineManagement` module with **certificate-based app-only authentication** (recommended for automation) or interactive user login.

**`Connect-EXOSession` / `Disconnect-EXOSession`:**
- Certificate-based auth via `Connect-ExchangeOnline -CertificateThumbprint`
- Interactive login via browser OAuth
- TLS 1.2 forced on every connection
- Connects to IPPS by default (Security & Compliance Center); `-SkipIPPS` to opt out
- Friendly mapping of `AADSTS700016` / `7000215` / cert-not-found errors

> 📖 Docs: [`Connect-ExchangeOnline.md`](Connect-ExchangeOnline.md) · ⚙ Setup: [`SETUP.md` §3](SETUP.md#3️⃣-connect-exchangeonline-app-registration)

### 🔹 Connect-Teams (`Connect-Teams.psm1`)

Manage **Microsoft Teams** via Microsoft Graph API. List teams, channels, members, and create new channels (standard, private, shared).

**`Connect-Teams` / `Disconnect-Teams`:**
- OAuth 2.0 client credentials or interactive delegated auth
- Auto-installs `Microsoft.Graph.Teams` from PSGallery
- Token caching with `finally`-block leak protection

**`Get-TeamsAll` / `Get-TeamsChannel` / `Get-TeamsMember`:**
- Auto-pagination via `@odata.nextLink` with `-All` switch
- Lists channels per team + members per team with role information

**`New-TeamsChannel`:**
- Creates standard / private / shared channels
- `ShouldProcess` enabled — supports `-WhatIf` for safe dry-runs

> 📖 Docs: [`Connect-Teams.md`](Connect-Teams.md) · ⚙ Setup: [`SETUP.md` §4](SETUP.md#4️⃣-connect-teams-app-registration)

### 🔹 Connect-SharePoint (`Connect-SharePoint.psm1`)

Manage **SharePoint Online** via Microsoft Graph API. Access sites, lists, and files at scale.

**`Connect-SharePoint` / `Disconnect-SharePoint`:**
- OAuth 2.0 client credentials or interactive delegated auth
- Auto-installs `Microsoft.Graph.Sites` + `Microsoft.Graph.Files`
- Optional `-SiteUrl` pre-resolves the current site on connect

**`Get-SharePointSite` / `Get-SharePointList` / `Add-SharePointListItem`:**
- Site resolution by URL via `Get-MgSiteByHostName`
- List queries with OData `$filter` and `-All` auto-pagination
- Item creation with `ShouldProcess` (`-WhatIf`) — OData injection prevented
- Site caching: `-SiteUrl` on connect avoids repeated resolutions

**`Get-SharePointFile` / `Upload-SharePointFile`:**
- Streaming file download to local path
- File upload with `conflictBehavior = rename` (configurable)

> 📖 Docs: [`Connect-SharePoint.md`](Connect-SharePoint.md) · ⚙ Setup: [`SETUP.md` §5](SETUP.md#5️⃣-connect-sharepoint-app-registration)

### 🔹 Connect-MgGraph (`Connect-MgGraph.psm1`)

General-purpose **Microsoft Graph** connection for Users, Groups, Devices, and organization-wide directory operations.

**`Connect-MgGraphModule` / `Disconnect-MgGraphModule`:**
- OAuth 2.0 client credentials or interactive delegated auth
- Auto-installs `Microsoft.Graph.Authentication`, `Microsoft.Graph.Users`, `Microsoft.Graph.Groups`, `Microsoft.Graph.Identity.DirectoryManagement`
- BSTR-safe SecureString handling — `ZeroFreeBSTR` in `finally` blocks

**`Get-MgGraphUser` / `Get-MgGraphGroup` / `Get-MgGraphDevice`:**
- OData `-Filter` for server-side queries (reduces payload + throttling)
- Auto-pagination via `-All` switch
- `-WithMembers` on `Get-MgGraphGroup` enumerates per-group members
- Customizable `-Property` arrays for selectivity

> 📖 Docs: [`Connect-MgGraph.md`](Connect-MgGraph.md) · ⚙ Setup: [`SETUP.md` §6](SETUP.md#6️⃣-connect-mggraph-app-registration)

---

## ⚙️ Requirements

| Requirement | Minimum | Notes |
|-------------|---------|-------|
| PowerShell | 5.1+ | Built into Windows 10/11 |
| .NET Framework | 4.6.2+ | Required by Microsoft.Graph.Authentication |
| Microsoft.Graph.Authentication | Auto-installed | Used by Connect-Intune |
| ExchangeOnlineManagement | Auto-installed | Used by Connect-ExchangeOnline |
| Azure AD App Registration | — | With appropriate permissions (see below) |
| Internet Access | — | Outbound HTTPS to Graph, EXO, and login endpoints |

> 📖 **First time setup?** See the [App Registration Guide](SETUP.md) for step-by-step instructions.

---

## 🚀 How to Run

### Connect to Intune (App-Only)

```powershell
$env:INTUNE_CLIENT_SECRET = "your-client-secret"

Import-Module .\Modules\Connect-Intune.psm1
Connect-Intune -TenantId "your-tenant-id" -ClientId "your-app-id"
$devices = Get-IntuneDevices
Send-IntuneDeviceSync -DeviceId $devices[0].id
Disconnect-Intune
```

### Connect to Intune (Interactive)

```powershell
Import-Module .\Modules\Connect-Intune.psm1
Connect-Intune -Interactive
$devices = Get-IntuneDevices
Disconnect-Intune
```

### Send Email via Graph

```powershell
$env:EXCHANGE_CLIENT_SECRET = "your-client-secret"

Import-Module .\Modules\Send-ExchangeMail.psm1
Send-ExchangeMail -From "sender@your-domain.onmicrosoft.com" `
    -To "user@your-domain.com" `
    -Subject "Report" `
    -Body "<h1>Hello</h1>" `
    -TenantId "your-tenant-id" `
    -ClientId "your-app-id"
```

### Connect to Exchange Online (Certificate Auth)

```powershell
$env:EXO_CERT_THUMBPRINT = "your-cert-thumbprint"

Import-Module .\Modules\Connect-ExchangeOnline.psm1
Connect-EXOSession -TenantId "your-tenant-id" -AppId "your-app-id" -Organization "your-domain.com"
Get-QuarantineMessage -PageSize 10
Disconnect-EXOSession
```

### Connect to Microsoft Teams

```powershell
$env:TEAMS_CLIENT_SECRET = "your-client-secret"

Import-Module .\Modules\Connect-Teams.psm1
Connect-Teams -TenantId "your-tenant-id" -ClientId "your-app-id"
$teams = Get-TeamsAll -Top 10
New-TeamsChannel -TeamId $teams[0].id -Name "Reports" -MembershipType standard
Disconnect-Teams
```

### Connect to SharePoint Online

```powershell
$env:SHAREPOINT_CLIENT_SECRET = "your-client-secret"

Import-Module .\Modules\Connect-SharePoint.psm1
Connect-SharePoint -TenantId "your-tenant-id" -ClientId "your-app-id" `
    -SiteUrl "https://your-domain.sharepoint.com/sites/IT"
$items = Get-SharePointList -ListName "Hardware" -All
$items | Select-Object -First 5
Disconnect-SharePoint
```

### General Microsoft Graph (Users, Groups, Devices)

```powershell
$env:GRAPH_CLIENT_SECRET = "your-client-secret"

Import-Module .\Modules\Connect-MgGraph.psm1
Connect-MgGraphModule -TenantId "your-tenant-id" -ClientId "your-app-id"
$users  = Get-MgGraphUser -Filter "accountEnabled eq true" -All
$groups = Get-MgGraphGroup -WithMembers -Top 50
Disconnect-MgGraphModule
```

### Using in Scripts (Recommended Pattern)

Place the `Modules` folder two levels above your script:

```
ProjectRoot/
  Modules/
    Connect-Intune.psm1
    Send-ExchangeMail.psm1
    Connect-ExchangeOnline.psm1
  Scripts/
    MyScript/
      MyScript.ps1
```

Inside `MyScript.ps1`:

```powershell
$modulePath = Join-Path $PSScriptRoot '..\..\Modules'
Import-Module (Join-Path $modulePath 'Connect-Intune.psm1') -Force
Import-Module (Join-Path $modulePath 'Send-ExchangeMail.psm1') -Force

Connect-Intune

# ... your logic ...

Send-ExchangeMail -From "noreply@your-domain.com" -To "admin@your-domain.com" `
    -Subject "Report" -Body "<p>Done</p>"

Disconnect-Intune
```

---

## 🔐 Environment Variables

| Variable | Module | Purpose |
|---|---|---|
| `$env:INTUNE_CLIENT_SECRET` | Connect-Intune | Client secret for app-only auth |
| `$env:EXCHANGE_CLIENT_SECRET` | Send-ExchangeMail | Client secret for app-only auth |
| `$env:EXO_CERT_THUMBPRINT` | Connect-ExchangeOnline | Certificate thumbprint for app-only auth |
| `$env:TEAMS_CLIENT_SECRET` | Connect-Teams | Client secret for app-only auth |
| `$env:SHAREPOINT_CLIENT_SECRET` | Connect-SharePoint | Client secret for app-only auth |
| `$env:GRAPH_CLIENT_SECRET` | Connect-MgGraph | Client secret for app-only auth |

---

## 🔑 Permissions

### Connect-Intune

| Permission | Type | Purpose |
|------------|------|---------|
| `DeviceManagementManagedDevices.ReadWrite.All` | Application | Read/write managed devices |
| `DeviceManagementConfiguration.ReadWrite.All` | Application | Read/write device configurations |
| `DeviceManagementManagedDevices.PrivilegedOperations.All` | Application | Remote actions (sync, wipe, etc.) |
| `Directory.Read.All` | Application | Read directory data |

### Send-ExchangeMail

| Permission | Type | Purpose |
|------------|------|---------|
| `Mail.Send` | Application | Send email as any mailbox |

### Connect-ExchangeOnline

| Permission | Type | Purpose |
|------------|------|---------|
| `Exchange.ManageAsApp` | Application | Manage Exchange as app |

### Connect-Teams

| Permission | Type | Purpose |
|------------|------|---------|
| `Team.ReadBasic.All` | Application | Read basic team info |
| `Channel.ReadBasic.All` | Application | Read channel info |
| `Channel.Create` | Application | Create channels |
| `TeamMember.Read.All` | Application | Read team members |

### Connect-SharePoint

| Permission | Type | Purpose |
|------------|------|---------|
| `Sites.ReadWrite.All` | Application | Read/write SharePoint sites |
| `Files.ReadWrite.All` | Application | Read/write files in all site collections |
| `AllSites.FullControl` | Application | Full control on all sites |

### Connect-MgGraph

| Permission | Type | Purpose |
|------------|------|---------|
| `User.Read.All` | Application | Read all users |
| `Group.Read.All` | Application | Read all groups |
| `GroupMember.Read.All` | Application | Read group members |
| `Device.Read.All` | Application | Read all devices |
| `Directory.Read.All` | Application | Read directory data |
| `Organization.Read.All` | Application | Read organization info |

---

## 🔄 Authentication Flow

```
┌───────────────────────────────────────────────────────────┐
│                    AUTH FLOW                              │
│                                                           │
│  Script calls Connect-Intune                              │
│       │                                                   │
│       ├── App-Only (Client Credentials)                   │
│       │   ├── Resolve secret: param > env > default       │
│       │   ├── POST login.microsoftonline.com/.../token    │
│       │   ├── Receive access_token                        │
│       │   └── Connect-MgGraph -AccessToken $token         │
│       │                                                   │
│       └── Interactive (User Login)                        │
│           ├── Connect-MgGraph -Scopes @(...)              │
│           └── Browser-based OAuth consent                 │
│                                                           │
│  Script calls Send-ExchangeMail                           │
│       │                                                   │
│       ├── Resolve secret: param > env > default           │
│       ├── POST login.microsoftonline.com/.../token        │
│       ├── Receive access_token                            │
│       └── POST graph.microsoft.com/v1.0/.../sendMail      │
│                                                           │
│  Script calls Connect-EXOSession                          │
│       │                                                   │
│       ├── Certificate Auth                                │
│       │   └── ExchangeOnlineManagement\Connect-Exchange   │
│       │       -AppId $app -CertThumbprint $cert           │
│       │                                                   │
│       └── Interactive                                     │
│           └── ExchangeOnlineManagement\Connect-Exchange   │
└───────────────────────────────────────────────────────────┘
```

---

## 🐛 Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `AADSTS700016` | Wrong ClientId or app not found | Verify app registration in Azure AD |
| `AADSTS7000215` | Invalid client secret/certificate | Regenerate in Azure AD > Certificates & secrets |
| `Access is denied` / `403` | App lacks required permissions | Grant and admin-consent permissions |
| `Failed to install module` | No internet or PSGallery blocked | Install module manually |
| `Unable to connect to remote server` | Firewall/proxy blocks endpoints | Allow HTTPS to Graph/EXO endpoints |
| `SendAsDenied` | App not authorized for `-From` mailbox | Add Send As permission in Exchange Admin |
| `Certificate not found` | Certificate not installed locally | Install in Local Machine\Personal store |
| Intune `Get-IntuneDevices` returns empty | Not connected | Run `Connect-Intune` first |
| `Invoke-MgGraphRequest` not found | Graph module not installed | `Connect-Intune` auto-installs it |

---

## 🛡 Design Principles

- **No secrets in code** — Example placeholder values shown; use `$env:XXX_CLIENT_SECRET` in production
- **Parameter-driven** — Credentials via params, env vars, or secrets manager
- **Auto-dependency** — Modules install required PowerShell modules from PSGallery if missing
- **Pagination handled** — All Get-* functions handle `@odata.nextLink` automatically
- **TLS enforced** — Connect-ExchangeOnline forces TLS 1.2 for every connection
- **Error handling** — Try/catch with descriptive error messages and AADSTS code mapping
- **Verbose logging** — `-Verbose` on any function for debug output
- **PowerShell 5.1+** — Tested on Windows PowerShell 5.1 and PowerShell 7+
- **Modular design** — Each module handles one M365 service independently
- **Centralized auth** — Single auth module per service, reusable across all scripts
- **BSTR-safe** — All SecureString handling uses `ZeroFreeBSTR` in `finally` blocks
- **Future-proof** — OAuth 2.0 + Certificate Auth; no Basic Auth, no SMTP, no legacy protocols

---

## 📊 Operational Safeguards

| Category | Safeguard |
|----------|-----------|
| Secret Handling | Never hardcoded; `$null` defaults; env var or parameter required |
| Token Expiry | Modules re-acquire tokens on each connection |
| Module Loading | Auto-installs missing modules from PSGallery |
| Pagination | Automatic `@odata.nextLink` handling in Graph queries |
| TLS | Enforced TLS 1.2 for all Exchange Online connections |
| Error Messages | Descriptive AADSTS error codes mapped to plain English |

---

## 📜 License

MIT License — see [LICENSE](LICENSE) file.

---

## 👤 Author

**Mohammad Abdulkader Omar**  
Website: https://momar.tech  
LinkedIn: https://www.linkedin.com/in/mabdulkadr/  

---

## 📊 Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2025-06-01 | Initial release: Connect-Intune, Send-ExchangeMail, Connect-ExchangeOnline |
| v1.1 | 2025-06-15 | Added: Connect-Teams, Connect-SharePoint, Connect-MgGraph |

---

## ⚠ Disclaimer

These scripts are provided as-is. Test them in a staging environment before applying them to production. The author is not responsible for any unintended outcomes resulting from their use.
