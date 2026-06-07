---
name: windows
description: Windows administration, troubleshooting, PowerShell, services, logs, networking, identity, storage, SRE, and Infrastructure as Code
---

# Windows Sysadmin / SRE Skill

## Operating Principles
- Troubleshoot from evidence: symptoms, timeline, scope, recent changes, logs, and reproducible checks.
- Prefer read-only diagnostics before remediation.
- Preserve evidence before changing state.
- Use PowerShell objects where possible instead of parsing plain text.
- Be comfortable designing automation, runbooks, and Windows-native tools when a recurring issue needs a durable fix.

## Safety First
Ask before running commands that:
- install/remove applications or Windows features
- change services, scheduled tasks, firewall rules, Defender policy, registry, GPO, identity, certificates, BitLocker, boot/BCD, disks, or production infrastructure
- run deployments or mutating IaC commands such as `terraform apply` or `terraform destroy`

Disk operations require live verification first:
```powershell
Get-Disk
Get-Partition
Get-Volume
mountvol
```
Never rely only on remembered disk numbers or drive letters.

## Core Diagnostics
```powershell
Get-ComputerInfo
systeminfo
Get-CimInstance Win32_OperatingSystem
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Get-Service | Where-Object Status -ne 'Running'
Get-ScheduledTask | Where-Object State -eq 'Ready'
```

## Event Logs
Prefer `Get-WinEvent` for modern logs and filters:
```powershell
Get-WinEvent -LogName System -MaxEvents 50
Get-WinEvent -LogName Application -MaxEvents 50
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2; StartTime=(Get-Date).AddHours(-24)}
Get-WinEvent -ListLog * | Sort-Object RecordCount -Descending | Select-Object -First 20 LogName,RecordCount
```

Useful logs:
- System
- Application
- Security, when authorized
- Microsoft-Windows-WindowsUpdateClient/Operational
- Microsoft-Windows-TaskScheduler/Operational
- Microsoft-Windows-PowerShell/Operational
- Microsoft-Windows-TerminalServices-* where relevant

## Services
```powershell
Get-Service <name>
Get-CimInstance Win32_Service -Filter "Name='<name>'" | Select-Object *
sc.exe qc <name>
sc.exe queryex <name>
```

Common checks:
- service account and permissions
- executable path and arguments
- dependencies
- recent service control events in System log
- crash events in Application log

## Networking
```powershell
ipconfig /all
Get-NetIPConfiguration
Get-NetAdapter | Format-Table Name,Status,LinkSpeed,MacAddress
Get-NetRoute
Resolve-DnsName example.com
Test-NetConnection example.com -Port 443
Get-NetTCPConnection | Sort-Object State,LocalPort
netstat -ano
```

Troubleshooting flow:
1. local interface/IP state
2. DNS resolution
3. route selection
4. firewall/proxy/VPN
5. remote listener and TLS
6. app-level logs

## Storage / Filesystems
```powershell
Get-Disk
Get-Partition
Get-Volume
Get-PhysicalDisk
Get-StoragePool
Get-ChildItem C:\ -Force
fsutil volume diskfree C:
```

Check before remediation:
- volume identity, filesystem, free space
- BitLocker status
- Storage Spaces membership
- shadow copies/backups
- ownership and ACLs

## Identity / Security
```powershell
whoami /all
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember Administrators
Get-MpComputerStatus
Get-ChildItem Cert:\LocalMachine\My
```

Domain/Entra/AD work:
- verify domain, tenant, and target object before changes
- avoid changing policy, membership, or credentials without approval
- preserve auditability

## Reliability and Repair
Read-only or low-risk checks:
```powershell
sfc /verifyonly
DISM /Online /Cleanup-Image /ScanHealth
perfmon /rel
```

Ask before repair commands such as:
```powershell
sfc /scannow
DISM /Online /Cleanup-Image /RestoreHealth
Reset-ComputerMachinePassword
```

## PowerShell Automation Standards
- Use advanced functions with `[CmdletBinding()]` for reusable scripts.
- Support `-WhatIf` / `-Confirm` for mutating operations.
- Emit objects, not formatted strings, unless presenting final output.
- Use structured logging for scripts that may run unattended.
- Prefer idempotent changes and clear rollback notes.

Skeleton:
```powershell
function Invoke-SafeTask {
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [Parameter(Mandatory)]
        [string]$Name
    )

    if ($PSCmdlet.ShouldProcess($Name, 'Perform safe task')) {
        # change here
    }
}
```

## Infrastructure as Code
Safe by default:
```powershell
terraform fmt -check
terraform validate
terraform plan
bicep build main.bicep
az deployment group what-if --resource-group <rg> --template-file main.bicep
ansible-playbook site.yml --check --diff
```

Ask before:
- `terraform apply`, `terraform destroy`, imports/state surgery, backend changes
- Azure/AWS/GCP deployments or deletes
- changing remote state, secrets, service principals, managed identities, roles, or policies

IaC review checklist:
- state/backend safety
- provider versions and lock files
- naming/tagging conventions
- least privilege IAM/RBAC
- secrets handling
- drift and import strategy
- rollback plan
- environment separation

## Incident / SRE Workflow
1. Define impact and blast radius.
2. Establish timeline and recent changes.
3. Gather logs/metrics from the narrowest useful scope.
4. Form hypotheses and test them with read-only checks.
5. Mitigate safely before root-cause deep dives when impact is ongoing.
6. Document root cause, contributing factors, detection gaps, and prevention.

## Windows App / Tooling Awareness
For recurring operational pain, consider a Windows-native tool:
- PowerShell module or CLI
- Windows service / worker service
- scheduled task
- tray utility
- WPF/WinUI admin UI
- Event Log source + structured logging
- MSI/MSIX/winget packaging when distribution matters
