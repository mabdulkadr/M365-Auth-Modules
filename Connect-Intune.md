# 📦 Connect-Intune Module (Connect-Intune.psm1)

> Part of **[M365 Auth Modules](../README.md)**: centralized OAuth 2.0 + Certificate-based Microsoft 365 authentication.

![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-blue.svg)
![Graph](https://img.shields.io/badge/API-Microsoft%20Graph-0078D6.svg)
![Auth](https://img.shields.io/badge/Auth-OAuth%202.0%20%2B%20Cert-2ea44f.svg)
![Version](https://img.shields.io/badge/version-1.1-green.svg)

---

## 📖 Overview

Connect-Intune.psm1 connects to Microsoft Intune through the Graph API using Intune-specific scopes. It supports app-only sign-in with client credentials and delegated sign-in through a browser. The module includes helpers for listing devices, sending sync commands, and reading policies.

- Sign-in uses OAuth 2.0 client credentials or an interactive user login
- Calls go to `https://graph.microsoft.com/Beta/`
- It is useful for device audits, sync automation, and policy reports

> 📖 For App Registration setup see [`SETUP.md`](SETUP.md) section **1️⃣ Connect-Intune App Registration**.

---

## ✨ Core Features

### 🔹 Connect-Intune / Disconnect-Intune
- Signs in with app-only client credentials or an interactive login
- Installs `Microsoft.Graph.Authentication` if it is missing
- Detects Graph SDK v2 or v1 and switches to the Beta profile for v1
- Turns `AADSTS700016`, `AADSTS7000215`, and "Access is denied" into readable errors

### 🔹 Get-IntuneDevices
- Lists managed devices from `/Beta/deviceManagement/managedDevices`
- Follows `@odata.nextLink` to fetch every page
- Accepts an OData `-Filter`, for example `deviceName eq 'PC-001'`
- `-Top` caps how many devices are returned per page

### 🔹 Send-IntuneDeviceSync
- Posts the `syncDevice` action to one device ID
- Returns `$true` when it works and a descriptive error when it fails

### 🔹 Get-IntunePolicy
- Lists device configuration policies
- Filters on `-Name` as a substring on the client
- Pages through all policy results

---

## ⚙️ Requirements

| Requirement | Minimum | Purpose |
|-------------|---------|---------|
| PowerShell | 5.1+ | Script execution |
| Module | `Microsoft.Graph.Authentication` (auto-installed if missing) | Graph SDK |
| Azure AD App | App registration with Intune permissions | Authentication |
| API Permissions | `DeviceManagementManagedDevices.ReadWrite.All`, `DeviceManagementConfiguration.ReadWrite.All`, `DeviceManagementManagedDevices.PrivilegedOperations.All`, `Directory.Read.All` (Application) | Intune data + remote actions |
| Network | Outbound HTTPS to `login.microsoftonline.com` and `graph.microsoft.com` | Token + Graph calls |

---

## 🚀 How to Run

### Option 1 — App-Only (Recommended for Automation)

```powershell
$env:INTUNE_CLIENT_SECRET = "your-client-secret"

Import-Module .\Connect-Intune.psm1
Connect-Intune -TenantId "your-tenant-id" -ClientId "your-app-id"

$devices = Get-IntuneDevices
$devices | Where-Object { $_.osVersion -lt "10.0.22631" } |
    Select-Object deviceName, osVersion, lastSyncDateTime

Disconnect-Intune
```

### Option 2 — Interactive (Delegated)

```powershell
Import-Module .\Connect-Intune.psm1
Connect-Intune -Interactive

Get-IntunePolicy -Name "Windows Update" | Format-Table displayName, lastModifiedDateTime
Disconnect-Intune
```

### Option 3 — Trigger Device Sync (Scheduled)

```powershell
$env:INTUNE_CLIENT_SECRET = $env:INTUNE_CLIENT_SECRET
Import-Module "C:\Modules\Connect-Intune.psm1"
Connect-Intune

$devices = Get-IntuneDevices -Filter "lastSyncDateTime le 2025-01-01T00:00:00Z"
foreach ($d in $devices) {
    Send-IntuneDeviceSync -DeviceId $d.id
    Write-Host "Synced: $($d.deviceName)"
}

Disconnect-Intune
```

---

## ⚙️ Parameters

### Connect-Intune

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `-ClientId` | string | `$script:DefaultClientId` | Azure AD application (client) ID |
| `-TenantId` | string | `$script:DefaultTenantId` | Azure AD tenant ID |
| `-ClientSecret` | SecureString | — | Client secret override |
| `-Interactive` | switch | `$false` | Use interactive user login |
| `-Scopes` | string[] | module defaults | Custom Graph scopes for interactive auth |

### Secret Resolution Order (App-Only)

1. `-ClientSecret` parameter (SecureString)
2. `$env:INTUNE_CLIENT_SECRET` environment variable
3. `$script:DefaultClientSecret` module-scoped default

### Get-IntuneDevices

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `-Filter` | string | — | OData filter (e.g., `deviceName eq 'PC-001'`) |
| `-Top` | int | `0` (all) | Max devices per page; `0` = all |

### Send-IntuneDeviceSync

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `-DeviceId` | string | — (required) | Intune device GUID |

### Get-IntunePolicy

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `-Name` | string | — | Substring filter on `displayName` |

---

## 🔄 Typical Workflow

```
1. Connect-Intune                     ──▶ Token acquired; profile set to Beta
2. Get-IntuneDevices                  ──▶ Page through @odata.nextLink
3. Filter / report on devices         ──▶ e.g., lastSyncDateTime, osVersion
4. Send-IntuneDeviceSync -DeviceId $d ──▶ Force a sync on stale devices
5. Get-IntunePolicy -Name "..."       ──▶ Filter policies client-side
6. Disconnect-Intune                  ──▶ Terminate Graph session
```

---

## 📊 Operational Safeguards

- ✅ BSTR-safe SecureString handling with `ZeroFreeBSTR` in `finally` blocks
- 🔒 `script:Connected` state-checked before every Graph call
- 🛟 Auto-pagination via `@odata.nextLink` avoids silently dropping results
- 🔄 Token re-acquired on each `Connect-Intune` call (no stale tokens)
- 📝 `-Verbose` flag on all functions for debug output
- 🆘 Module auto-installs `Microsoft.Graph.Authentication` from PSGallery
- 🛡 Friendly AADSTS mapping translates `AADSTS700016` / `7000215` / `Access denied`

---

## 🐛 Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `AADSTS700016` | Wrong `ClientId` or app not in tenant | Verify app registration values |
| `AADSTS7000215` | Invalid `ClientSecret` | Regenerate in Certificates & secrets |
| `403 Forbidden` | Permissions missing | Add Intune permissions + **Grant admin consent** |
| `Not connected. Run Connect-Intune first.` | Function called before connect | Run `Connect-Intune` first |
| `Failed to install Microsoft.Graph.Authentication` | PSGallery blocked | `Install-Module Microsoft.Graph.Authentication -Scope CurrentUser` |
| `Beta endpoint not found` | Using older Graph SDK | `Update-Module Microsoft.Graph.* -Force` |
| `Get-IntuneDevices` returns empty | Wrong endpoint filter or no devices | Verify with `Get-MgDeviceManagementManagedDevice` directly |
| `Send-IntuneDeviceSync` 404 | Device not managed by this tenant | Verify device ID against `Get-IntuneDevices` |
| `Get-MgContext` empty | Disconnected state | Re-run `Connect-Intune` |

---

## 🛡 Design Principles

- Example placeholder values are shown; there are no secrets in code, so use `$env:INTUNE_CLIENT_SECRET` in production
- `@odata.nextLink` is consumed automatically
- Required Graph modules are installed on first import
- The `script:Connected` guard prevents calls before connection
- SecureString conversion uses `ZeroFreeBSTR` in `finally`
- AADSTS codes are translated to plain English
- Uses `/Beta/deviceManagement/managedDevices` for richer data
- Detects v2 vs v1 SDK and selects the correct profile

---

## 📜 License

This project is licensed under the [MIT License](../LICENSE) file.

---

## 👤 Author

**Mohammad Abdulkader Omar**  
Website: https://momar.tech  
LinkedIn: https://www.linkedin.com/in/mabdulkadr/  

---

## ⚠ Disclaimer

These scripts are provided as-is. Test them in a staging environment before applying them to production. The author is not responsible for any unintended outcomes resulting from their use.
