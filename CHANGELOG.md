# 📋 Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> Project name: **M365 Auth Modules** (formerly "Microsoft 365 PowerShell Modules")

---

## [1.1.0] - 2025-06-15

### Added

- Connect-Teams.psm1: Microsoft Teams management module
  - List teams, channels, and members via Microsoft Graph
  - Create new channels (standard, private, shared)
  - Auto-pagination via `-All` switch
  - App-only (client credentials) and interactive delegated auth
- Connect-SharePoint.psm1: SharePoint Online management module
  - Resolve sites by URL
  - CRUD operations on SharePoint lists
  - File upload and download
  - Auto-pagination via `-All` switch
- Connect-MgGraph.psm1: general-purpose Microsoft Graph module
  - List users, groups, and devices
  - OData filter support
  - Group member enumeration
- CHANGELOG.md: this file

### Changed

- README.md: expanded the Overview to highlight **OAuth 2.0 / Certificate Auth**
  instead of Basic Auth. Added an authentication methods table.
- README.md: added Teams, SharePoint, and MgGraph to all sections
  (folder structure, core features, env vars, permissions).
- SETUP.md: added dedicated sections for Teams, SharePoint, and MgGraph
  App Registration, plus an updated combined permissions table.
- SETUP.md: replaced real-looking organizational domain examples
  (`contoso.onmicrosoft.com`, `contoso.com`) with generic `your-domain` placeholders.
- Connect-Intune.psm1: removed the unused `$script:IntuneAppScopes` variable.
- All .psm1 files: updated example placeholder TenantId / ClientId / ClientSecret
  values to consistent random defaults.

### Fixed

- Connect-SharePoint.psm1: OData filter string interpolation now uses
  proper escaping for `displayName` queries.
- All modules: verified BSTR leak-free secure string handling with
  `ZeroFreeBSTR` in `finally` blocks.
- All modules: verified BSTR is not released before use.

---

## [1.0.0] - 2025-06-01

### Added

- Initial release of M365 Auth Modules
- Connect-Intune.psm1: Microsoft Graph authentication for Intune
  - `Connect-Intune`, `Disconnect-Intune`
  - `Get-IntuneDevices`, `Send-IntuneDeviceSync`, `Get-IntunePolicy`
  - OAuth 2.0 client credentials flow (app-only)
  - Interactive delegated authentication
  - Auto-installs Microsoft.Graph.Authentication module
- Send-ExchangeMail.psm1: HTML email via Microsoft Graph
  - `Send-ExchangeMail` function
  - HTML body, CC/BCC, file attachments
  - Send-As mailbox support
  - Secret resolution: parameter → env var → default
- Connect-ExchangeOnline.psm1: Exchange Online + Security & Compliance
  - `Connect-EXOSession`, `Disconnect-EXOSession`
  - Certificate-based app-only authentication
  - Interactive user authentication
  - TLS 1.2 enforced
- README.md: main documentation in SCCM-OSD style
- SETUP.md: complete App Registration guide
- .gitignore: prevents secrets from being committed

### Security

- All modules use OAuth 2.0 and certificate auth, with no Basic Auth, SMTP,
  or legacy protocols.
- BSTR leak-free secure string handling.
- Example placeholder values are clearly marked as `replace with your own`.

---

## Legend

| Symbol | Meaning |
|--------|---------|
| 🆕 | New feature |
| 🔧 | Change/improvement |
| 🐛 | Bug fix |
| 🔒 | Security |
| ⚠️ | Deprecation/breaking |
| 📝 | Documentation |
