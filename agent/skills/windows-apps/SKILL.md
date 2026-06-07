---
name: windows-apps
description: Windows ecosystem app design with .NET, WinUI, WPF, services, CLIs, installers, diagnostics, and deployment patterns
---

# Windows App Design Skill

## Scope
Use this skill when designing or implementing applications in the Windows ecosystem, especially tools that support admin, SRE, automation, or infrastructure workflows.

Good fits:
- .NET console apps and CLIs
- PowerShell modules
- Windows Services / Worker Services
- WPF and WinUI desktop applications
- tray utilities and background agents
- internal admin dashboards
- installers and enterprise deployment
- diagnostics, logging, telemetry, and supportability

## Design Principles
- Fit the operator workflow first: fast diagnosis, clear remediation, safe defaults.
- Prefer boring Windows-native foundations over novelty.
- Keep admin privileges narrow and explicit.
- Design for logs, troubleshooting, and support from day one.
- Treat configuration, credentials, and secrets as first-class architecture concerns.

## Technology Choices

### CLI / Automation
Prefer:
- PowerShell module for admin-oriented automation
- .NET CLI for cross-platform or compiled tooling
- `System.CommandLine` or Spectre.Console when richer CLI UX is useful

### Desktop UI
Prefer:
- WPF for mature internal tools and broad compatibility
- WinUI 3 for modern Windows-native UI where packaging/runtime constraints are acceptable
- WebView2 only when web tech materially improves delivery

### Background Work
Prefer:
- .NET Worker Service for long-running services
- Windows Service when it must run without an interactive user
- Scheduled Task for periodic jobs that do not need a resident service

### Packaging / Deployment
Consider:
- winget manifests for simple distribution
- MSIX for modern packaging where constraints are acceptable
- MSI/WiX for enterprise deployment control
- Intune, GPO, SCCM/MECM, or RMM tooling depending on environment

## Architecture Checklist
- What privilege level is required and why?
- Is the app user-interactive, service-like, scheduled, or event-driven?
- Where is config stored: file, registry, environment, policy, or cloud?
- How are secrets acquired and protected: DPAPI, Windows Credential Manager, managed identity, Key Vault?
- What logs are written and how will support collect them?
- How are updates, rollback, and compatibility handled?
- How does the app behave offline, behind proxy/VPN, or under limited permissions?

## Logging and Diagnostics
Use structured logs where possible.

Recommended sinks:
- console output for CLI tools
- Windows Event Log for services/admin tools
- rolling file logs under `%ProgramData%\<Vendor>\<App>\Logs` for machine-wide apps
- `%LOCALAPPDATA%\<Vendor>\<App>\Logs` for per-user apps
- ETW/OpenTelemetry when deeper observability is justified

.NET examples:
- `Microsoft.Extensions.Logging`
- EventLog provider for Windows Services
- Serilog when richer sinks/enrichment are needed

## Service Design
For Windows Services:
- use least-privileged service accounts
- avoid LocalSystem unless justified
- define recovery actions intentionally
- write clear startup/shutdown logs
- implement health checks or heartbeat events where useful
- handle network unavailability and dependency startup order

Useful checks:
```powershell
Get-Service <name>
Get-CimInstance Win32_Service -Filter "Name='<name>'"
sc.exe qc <name>
Get-WinEvent -LogName Application -MaxEvents 50
```

## PowerShell Module Design
- Use approved verbs.
- Emit objects, not formatted text.
- Support pipeline input where it helps.
- Add comment-based help.
- Use `SupportsShouldProcess` for mutating commands.
- Separate discovery (`Get-*`, `Test-*`) from mutation (`Set-*`, `Repair-*`, `Invoke-*`).

## Security
- Do not store plaintext secrets.
- Prefer Windows-integrated auth where possible.
- Use DPAPI or Credential Manager for local user/machine secrets.
- Validate paths and external process arguments.
- Avoid running arbitrary shell commands from UI input.
- Sign scripts/binaries when enterprise policy requires it.

## UX for Admin Tools
- Show target scope before making changes.
- Provide dry-run / preview for remediations.
- Include copyable diagnostic summaries.
- Prefer clear status and remediation guidance over decorative UI.
- Make failures actionable: include failing command, exit code, relevant log location, and next diagnostic step.

## Validation
Before shipping or recommending:
- run unit tests where available
- test on a non-admin account if admin is not required
- test offline/proxy/failure paths
- verify logs and uninstall/rollback behavior
- check application manifests and requested execution level
- confirm packaging works on clean Windows machines
