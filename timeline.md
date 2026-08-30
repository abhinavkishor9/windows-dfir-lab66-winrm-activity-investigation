# Timeline — Lab 66 WinRM Activity Investigation

## Timeline Purpose

This timeline documents the WinRM troubleshooting sequence, configuration changes, successful WS-Man activity, and subsequent investigation for evidence of remote execution.

## Investigation Timeline

| Time | Source | Activity | Result |
|---|---|---|---|
| 28-08-2026 07:35:34 | WinRM Operational | WS-Man enumeration started | Earlier failed operation |
| 28-08-2026 07:35:39 | WinRM Operational | Enumeration failed | Client could not connect |
| 28-08-2026 07:39:59 | WinRM Operational | WS-Man enumeration started | Earlier failed operation |
| 28-08-2026 07:40:04 | WinRM Operational | Enumeration failed | Client could not connect |
| 29-08-2026 06:35:01 | WinRM Operational | Enumeration started | Earlier failed operation |
| 29-08-2026 06:35:05 | WinRM Operational | Enumeration failed | Client could not connect |
| 30-08-2026 06:37:37 | WinRM Operational | Enumeration started | Failed WS-Man activity |
| 30-08-2026 06:37:42 | WinRM Operational | Enumeration failed | Client could not connect |
| 30-08-2026 06:39:55 | WinRM Operational | Get operation started | Failed sequence |
| 30-08-2026 06:39:59 | WinRM Operational | Get operation failed | Client could not connect |
| 30-08-2026 06:42:19 | WinRM Operational | Service type configured | Delayed automatic start configured |
| 30-08-2026 06:42:24 | WinRM Operational | WinRM service starting | Service startup initiated |
| 30-08-2026 06:42:24 | WinRM Operational | WinRM service started | Service became operational |
| 30-08-2026 06:42:24 | WinRM Operational | Service type changed | Configuration completed |
| 30-08-2026 06:42:25 | WinRM Operational | Firewall exception enabled | WinRM network access configured |
| 30-08-2026 06:42:27 | WinRM Operational | WS-Man Invoke started | Configuration operation |
| 30-08-2026 06:42:28 | WinRM Operational | Remote token policy configured | LocalAccountTokenFilterPolicy changed |
| 30-08-2026 06:42:28 | WinRM Operational | Firewall exception enabled | Configuration confirmed |
| 30-08-2026 06:42:28 | WinRM Operational | Invoke completed | Configuration operation successful |
| 30-08-2026 06:44:33 | WinRM Operational | WS-Man Get completed | Successful operation |
| 30-08-2026 06:45:43 | WinRM Operational | Enumeration completed | Successful operation |
| 30-08-2026 06:59:00 | WinRM Operational | Enumeration completed | Successful operation |
| 30-08-2026 06:59:01 | WinRM Operational | Enumeration completed | Successful operation |
| Investigation | Process | PowerShell searched | PID 7724 identified |
| Investigation | Sysmon Event ID 1 | `wsmprovhost.exe` searched | No relevant result |
| Investigation | Security 4688 | PowerShell/WSMan process search | No relevant result |
| Investigation | PowerShell 4104 | Remote-management terms searched | No relevant result |
| Investigation | Sysmon Event ID 3 | 5985/5986 searched | No relevant result |
| Investigation | PowerShell History | WinRM commands reviewed | Local activity documented |

## Phase 1 — Initial Failure

Before the final WinRM configuration, the endpoint generated failed WS-Man operations.

Examples included:

```text
28-08-2026 07:35:39
28-08-2026 07:40:04
29-08-2026 06:35:05
30-08-2026 06:37:42
30-08-2026 06:39:59
```

The failures stated that the client could not connect to the requested destination.

## Phase 2 — WinRM Configuration

### 06:42:19

The WinRM Operational log recorded:

```text
Set the WinRM service type to delayed auto start.
```

### 06:42:24

The WinRM service started.

The log recorded:

```text
The WinRM service is starting
The WinRM service started successfully
WinRM service type changed successfully
```

### 06:42:25

The WinRM firewall exception was enabled.

### 06:42:27–06:42:28

WS-Man configuration operations completed.

The log recorded:

```text
Configured LocalAccountTokenFilterPolicy to grant administrative rights remotely to local users.
WinRM firewall exception enabled.
WSMan operation Invoke completed successfully.
```

## Phase 3 — Successful WinRM Operations

### 06:44:33

A WS-Man Get operation completed successfully.

### 06:45:43

A WS-Man enumeration completed successfully.

### 06:59:00–06:59:01

Additional enumeration operations completed successfully.

These events demonstrated that WinRM became operational.

## Phase 4 — Final Service State

The final service state was:

```text
Name      : WinRM
State     : Running
StartMode : Auto
StartName : NT AUTHORITY\NetworkService
ProcessId : 1672
```

## Phase 5 — Listener State

The listener was:

```text
Transport : HTTP
Port      : 5985
Enabled   : true
Address   : *
```

## Phase 6 — Process Investigation

The current PowerShell process was:

```text
powershell.exe
PID 7724
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

No relevant `wsmprovhost.exe` process event was established.

## Phase 7 — PowerShell Investigation

The targeted Event ID 4104 search did not return:

```text
WinRM
Enter-PSSession
Invoke-Command
New-PSSession
```

No direct Script Block Logging evidence of remote PowerShell execution was established.

## Phase 8 — Security Investigation

The Security Event ID 4688 search did not return a relevant:

```text
powershell.exe
wsmprovhost.exe
```

event.

## Phase 9 — Network Investigation

The Sysmon Event ID 3 search for:

```text
5985
5986
```

returned no relevant results.

No WinRM-port network activity was established from the supplied evidence.

## Phase 10 — Command History

The command history documented local WinRM administration and investigation commands.

The sequence included:

```text
Get-Service WinRM
winrm get winrm/config
winrm quickconfig
winrm enumerate winrm/config/listener
Get-WinEvent -ListLog *WinRM*
Get-WinEvent -LogName "Microsoft-Windows-WinRM/Operational"
Get-NetFirewallRule
Get-CimInstance Win32_Service
Get-Process
```

## Final Evidence Chain

```text
Initial WS-Man Failure
        |
        v
winrm quickconfig
        |
        v
WinRM Service Started
        |
        v
Firewall Exception
        |
        v
HTTP Listener on 5985
        |
        v
Successful WS-Man Operations
        |
        v
Process Investigation
        |
        v
PowerShell Investigation
        |
        v
Network Investigation
        |
        v
Remote Activity Assessment
```

## Final Evidence Summary

| Evidence Source | Finding |
|---|---|
| WinRM Service | Running |
| Start Mode | Automatic |
| Service Account | `NT AUTHORITY\NetworkService` |
| Process ID | 1672 |
| HTTP Listener | Enabled |
| Listener Port | 5985 |
| AllowRemoteAccess | True |
| AllowRemoteShellAccess | True |
| Firewall Rule | Domain/Private HTTP-In enabled |
| WinRM Operational Log | Enabled |
| Successful WS-Man Operations | Observed |
| `wsmprovhost.exe` | No relevant event established |
| Security 4688 | No relevant result |
| PowerShell 4104 | No relevant targeted result |
| Sysmon 3 for 5985/5986 | No relevant result |
| Malicious Remote Execution | Not established |

## Final Assessment

The timeline demonstrates a clear transition:

```text
WinRM Not Operational
        |
        v
Configuration
        |
        v
WinRM Operational
```

The subsequent evidence confirms local WinRM functionality but does not establish a remote PowerShell session or malicious lateral movement.

## Investigation Conclusion

The final DFIR conclusion is:

> WinRM configuration and WS-Man activity were observed, including service startup, firewall configuration, listener creation, and successful local operations. However, the supplied process, PowerShell, Security, and network telemetry does not establish suspicious remote execution or lateral movement.
