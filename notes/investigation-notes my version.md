# Investigation Notes

## Initial WinRM State

The initial WinRM configuration query failed because the WS-Man client could not connect to the destination.

The WinRM Operational log subsequently documented multiple failed WS-Man operations before the service became operational.

## Troubleshooting and Configuration

The command:

```powershell
winrm quickconfig
```

was executed.

The WinRM Operational log recorded configuration activity around:

```text
30-08-2026 06:42:19
30-08-2026 06:42:24
30-08-2026 06:42:25
30-08-2026 06:42:27
30-08-2026 06:42:28
```

Observed events included:

```text
Set the WinRM service type to delayed auto start.
WinRM service started.
WinRM service type changed successfully.
Enable the WinRM firewall exception.
WinRM firewall exception enabled.
Configured LocalAccountTokenFilterPolicy to grant administrative rights remotely to local users.
WSMan operation Invoke completed successfully.
```

This represents the central configuration-change sequence of the lab.

## WinRM Service

The final service configuration was:

```text
Name      : WinRM
State     : Running
StartMode : Auto
StartName : NT AUTHORITY\NetworkService
ProcessId : 1672
```

The configured service path was:

```text
C:\Windows\System32\svchost.exe -k NetworkService -p
```

The executable component was:

```text
C:\Windows\System32\svchost.exe
```

The remaining text consisted of service arguments.

## WinRM Listener

The configured listener was:

```text
Address   : *
Transport : HTTP
Port      : 5985
Enabled   : true
URLPrefix : wsman
```

The listener addresses included:

```text
127.0.0.1
192.168.203.128
::1
fe80::469c:6992:a61d:ff19%4
```

## WinRM Security Configuration

The service configuration included:

```text
AllowRemoteAccess = true
AllowRemoteShellAccess = true
```

Unencrypted communication was disabled:

```text
Client AllowUnencrypted  = false
Service AllowUnencrypted = false
```

Service authentication included:

```text
Kerberos
Negotiate
```

## Firewall Configuration

The primary enabled firewall rule was:

```text
Windows Remote Management (HTTP-In)
Enabled   : True
Direction : Inbound
Action    : Allow
Profile   : Domain, Private
```

The Public profile rule was disabled.

## WinRM Operational Logging

The WinRM log was enabled:

```text
Microsoft-Windows-WinRM/Operational
IsEnabled : True
```

## Failed WS-Man Operations

Before configuration, failed operations included:

```text
30-08-2026 06:39:59
30-08-2026 06:37:42
29-08-2026 06:35:05
29-08-2026 06:17:40
28-08-2026 07:40:04
28-08-2026 07:35:39
28-08-2026 07:32:06
```

The associated messages indicated:

```text
The client cannot connect to the destination specified in the request.
```

These records were treated as evidence of unsuccessful WinRM operations rather than malicious activity.

## Successful WS-Man Operations

After configuration, successful operations were observed:

```text
30-08-2026 06:44:33
30-08-2026 06:45:43
30-08-2026 06:59:00
30-08-2026 06:59:01
```

The events included:

```text
WSMan operation Get completed successfully
WSMan operation Enumeration completed successfully
```

This confirmed that WinRM became operational.

## Process Investigation

Current process enumeration identified:

```text
powershell
PID 7724
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

The search for:

```text
wsmprovhost.exe
```

returned no relevant Sysmon Event ID 1 result.

This meant no remote PowerShell provider-host process was established from the supplied evidence.

## Security Event ID 4688

The 4688 search targeted:

```text
powershell.exe
wsmprovhost.exe
```

No relevant result was returned.

Therefore, 4688 did not provide process evidence of remote PowerShell execution.

## PowerShell Event ID 4104

The targeted 4104 query searched for:

```text
WinRM
Enter-PSSession
Invoke-Command
New-PSSession
```

No matching output was returned.

A broader 4104 query showed that Script Block Logging was active.

## Sysmon Event ID 3

The targeted Sysmon Event ID 3 query searched for:

```text
5985
5986
```

No matching output was returned.

Therefore, a network connection associated with the standard WinRM ports was not established from the supplied evidence.

## Command History

Current-session history contained WinRM investigation and configuration commands.

Important commands included:

```text
Get-Service WinRM | Select-Object Name, DisplayName, Status, StartType
winrm get winrm/config
winrm quickconfig
winrm enumerate winrm/config/listener
Get-CimInstance Win32_Service
Get-WinEvent -ListLog *WinRM*
Get-WinEvent -LogName "Microsoft-Windows-WinRM/Operational"
Get-NetFirewallRule
Get-Process
```

This provides a chronological record of the local investigation.

## PSReadLine Troubleshooting

The command:

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath
```

could not be used in one session because:

```text
Get-PSReadLineOption
```

was not recognized.

The investigation therefore used:

```powershell
Get-History
```

as the available current-session history source.

## Evidence Correlation

The investigation chain was:

```text
Initial WinRM Failure
        |
        v
winrm quickconfig
        |
        v
Service Started
        |
        v
Firewall Exception
        |
        v
Listener Enabled
        |
        v
WinRM Configuration
        |
        v
Successful WS-Man Operations
        |
        v
Process Search
        |
        v
PowerShell Search
        |
        v
Network Search
        |
        v
Remote Activity Assessment
```

