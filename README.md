# Windows DFIR Lab 66 — WinRM Activity Investigation

## Overview

This lab investigates Windows Remote Management (WinRM) from a DFIR perspective. WinRM is a legitimate Windows remote-management mechanism that can be used for administration, automation, and PowerShell remoting, but unexpected configuration changes or remote-management activity can also become indicators of lateral movement.

The investigation began with a WinRM configuration query that failed because the WinRM service was not yet responding correctly to WS-Man requests. The resulting troubleshooting process led to the execution of `winrm quickconfig`, after which WinRM became operational.

The investigation then examined the resulting WinRM configuration, service state, listener configuration, firewall rules, process telemetry, PowerShell logging, network telemetry, and command history.

The main finding was not confirmed malicious lateral movement. Instead, the lab demonstrated how a legitimate WinRM configuration change can generate useful forensic evidence and how the investigator must distinguish **WinRM configuration** from **actual remote WinRM activity**.

## Investigation Objectives

- Establish the original WinRM service state.
- Identify whether WinRM is running and determine its startup configuration.
- Examine the WinRM service account and process information.
- Determine whether a WinRM listener is configured.
- Identify the listener transport and port.
- Examine WinRM client and service configuration.
- Review WinRM Operational events for configuration changes and WS-Man activity.
- Investigate firewall rules associated with WinRM.
- Review Sysmon Event ID 1 for PowerShell and `wsmprovhost.exe`.
- Review Sysmon Event ID 3 for WinRM-related network connections.
- Review PowerShell Event ID 4104 for WinRM-related commands.
- Review Security Event ID 4688 for process evidence.
- Examine PowerShell command history for WinRM-related activity.
- Distinguish local WinRM configuration from evidence of remote execution.
- Document the troubleshooting steps required to make WinRM operational.
- Correlate configuration, process, and network telemetry before making a security conclusion.

## Investigation Scenario

A Windows workstation is being assessed for unexpected WinRM activity. Initial WS-Man queries fail, raising the question of whether WinRM is disabled, misconfigured, or unavailable.

The analyst investigates the local service, listener, firewall rules, and WinRM configuration. After the service is configured, the analyst examines the resulting WinRM Operational events and checks for signs of PowerShell remoting or remote execution.

The investigation must determine whether the observed evidence represents:

- Normal WinRM configuration and local administration.
- A configuration change that requires review.
- Actual remote-management activity.
- Evidence of suspicious lateral movement.

The exercise is performed without connecting to another machine or executing commands remotely.

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Windows |
| Investigation Type | Windows DFIR / Remote Management |
| WinRM Service | Running |
| WinRM Start Mode | Automatic |
| WinRM Account | `NT AUTHORITY\NetworkService` |
| WinRM Process ID | 1672 |
| HTTP Listener | Enabled |
| HTTP Port | 5985 |
| HTTPS Port | 5986 configured as default port |
| WinRM Operational Log | Enabled |
| Sysmon Event ID 1 | Available |
| Sysmon Event ID 3 | Available |
| Sysmon Event ID 10 | Not investigated as primary evidence |
| PowerShell Event ID 4104 | Available |
| Security Event ID 4688 | Queried |

## Initial WinRM Problem

The initial WinRM configuration query failed with a WS-Man connection error.

The investigation then used:

```powershell
winrm quickconfig
```

This resulted in WinRM configuration changes recorded in the WinRM Operational log.

The troubleshooting phase is an important part of the lab because it demonstrates that an operational change can itself generate forensic evidence.

## WinRM Service Baseline After Configuration

The final service state was:

```text
Name      : WinRM
State     : Running
StartMode : Auto
StartName : NT AUTHORITY\NetworkService
ProcessId : 1672
```

The service `PathName` was:

```text
C:\Windows\System32\svchost.exe -k NetworkService -p
```

As with previous service investigations, the executable portion is:

```text
C:\Windows\System32\svchost.exe
```

and the remaining portion contains service parameters.

## WinRM Listener

The configured listener was:

```text
Address    = *
Transport  = HTTP
Port       = 5985
Enabled    = true
URLPrefix  = wsman
```

The listener was reported as listening on:

```text
127.0.0.1
192.168.203.128
::1
fe80::469c:6992:a61d:ff19%4
```

This demonstrates that WinRM HTTP was configured and listening on the endpoint.

## WinRM Configuration

The WinRM configuration showed:

```text
Client AllowUnencrypted = false
Service AllowUnencrypted = false
```

Client authentication included:

```text
Basic
Digest
Kerberos
Negotiate
Certificate
```

The service authentication configuration included:

```text
Kerberos
Negotiate
```

with:

```text
Certificate = false
CredSSP = false
```

The service also showed:

```text
IPv4Filter = *
IPv6Filter = *
AllowRemoteAccess = true
```

and:

```text
AllowRemoteShellAccess = true
```

These settings are important configuration evidence but do not independently establish malicious use.

## Firewall Configuration

The following WinRM firewall rule was enabled:

```text
Windows Remote Management (HTTP-In)
Enabled   : True
Direction : Inbound
Action    : Allow
Profile   : Domain, Private
```

The corresponding Public profile rule was disabled.

The compatibility-mode rules were also disabled.

## WinRM Operational Events

The WinRM Operational log was enabled:

```text
Microsoft-Windows-WinRM/Operational
IsEnabled : True
```

Important events were recorded during configuration and testing.

At approximately:

```text
30-08-2026 06:42:24
```

the WinRM service started successfully.

Additional events recorded:

```text
WinRM service type changed successfully.
Enable the WinRM firewall exception.
Configured LocalAccountTokenFilterPolicy to grant administrative rights remotely to local users.
WinRM firewall exception enabled.
WSMan operation Invoke completed successfully.
```

These events document the configuration process.

## Earlier WS-Man Failures

Before WinRM became operational, the WinRM Operational log contained failed requests.

Examples occurred at:

```text
30-08-2026 06:39:59
30-08-2026 06:37:42
29-08-2026 06:35:05
29-08-2026 06:17:40
28-08-2026 07:40:04
28-08-2026 07:35:39
```

The failures contained:

```text
WSMan operation Get failed
WSMan operation Enumeration failed
The client cannot connect to the destination specified in the request
```

These failures were consistent with WinRM not yet being fully operational.

## Successful WS-Man Operations

After configuration, successful operations were recorded.

Examples included:

```text
30-08-2026 06:44:33
30-08-2026 06:45:43
30-08-2026 06:59:00
30-08-2026 06:59:01
```

These included:

```text
WSMan operation Get completed successfully
WSMan operation Enumeration completed successfully
```

This showed that WinRM functionality became operational after configuration.

## Process Investigation

Current process enumeration identified:

```text
powershell.exe
PID 7724
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

The search for:

```text
wsmprovhost.exe
```

did not return a relevant Sysmon Event ID 1 result.

This was significant because there was no supplied process evidence demonstrating the creation of a remote PowerShell provider host.

## Security Event ID 4688

Security Event ID 4688 was searched for:

```text
powershell.exe
wsmprovhost.exe
```

No result was returned in the supplied evidence.

Therefore, Security process-creation telemetry did not provide direct evidence of remote PowerShell execution.

## PowerShell Event ID 4104

PowerShell Script Block Logging was searched for:

```text
WinRM
Enter-PSSession
Invoke-Command
New-PSSession
```

The targeted search did not return a matching result.

A broader Event ID 4104 review showed that Script Block Logging was active and contained historical events, but the supplied evidence did not establish WinRM remote-command execution through 4104.

## Sysmon Event ID 3

Sysmon Event ID 3 was available.

The investigation specifically searched for:

```text
5985
5986
```

No matching result was returned in the supplied evidence.

Therefore, no WinRM-port network connection was established through the supplied Sysmon Event ID 3 search.

## PowerShell Command History

The current PowerShell history contained WinRM-related commands including:

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
Get-Content (Get-PSReadLineOption).HistorySavePath
```

This demonstrated the sequence of local investigative and configuration commands.

## PSReadLine Investigation

An attempt was made to use:

```powershell
Get-PSReadLineOption
```

to retrieve the persistent history path.

The command was not available in one session, so the investigation used:

```powershell
Get-History
```

instead.

This demonstrated an important evidence-source distinction between current-session history and persistent PSReadLine history.

## Evidence Correlation

The investigation followed:

```text
WinRM Configuration
       |
       v
WinRM Service
       |
       v
Listener
       |
       v
Firewall
       |
       v
WinRM Operational Events
       |
       v
PowerShell Activity
       |
       v
Process Telemetry
       |
       v
Network Telemetry
       |
       v
Remote Execution Assessment
```

## Key Findings

- WinRM was initially not responding correctly to WS-Man configuration queries.
- `winrm quickconfig` was used to make WinRM operational.
- The WinRM service became Running and Automatic.
- WinRM ran under `NT AUTHORITY\NetworkService`.
- WinRM process ID was 1672.
- An HTTP listener was enabled on port 5985.
- The listener was configured for all addresses.
- Remote access was enabled.
- Remote shell access was enabled.
- A Domain/Private WinRM firewall exception was enabled.
- WinRM Operational logging was enabled.
- WinRM logs recorded the configuration changes.
- Successful WS-Man operations occurred after configuration.
- No `wsmprovhost.exe` process evidence was established.
- No relevant 4688 result was identified.
- No relevant 4104 result was identified for the targeted remote-management terms.
- No 5985/5986 Sysmon Event ID 3 result was established.
- The evidence does not establish malicious lateral movement.

## DFIR Interpretation

The most important distinction in this investigation is:

```text
WinRM Enabled
    !=
WinRM Used for Lateral Movement
```

The endpoint was configured for WinRM, and the configuration process generated clear WinRM Operational events.

However, the supplied evidence did not establish:

```text
Remote Source
    +
Remote Authentication
    +
wsmprovhost.exe
    +
Remote PowerShell Command
    +
WinRM Network Connection
```

Therefore, suspicious remote execution was not confirmed.

## MITRE ATT&CK Relevance

Potentially relevant techniques in a real incident include:

**T1021.006 — Windows Remote Management**

Relevant when WinRM is used for remote services or lateral movement.

**T1059.001 — PowerShell**

Relevant when PowerShell is used during remote execution.

The controlled lab did not establish malicious use of either technique.

## Evidence Limitations

- The initial configuration failure reflected the endpoint's state before WinRM configuration.
- Security Event ID 4688 did not provide relevant evidence.
- The targeted PowerShell 4104 search did not establish remote-command execution.
- No `wsmprovhost.exe` process event was identified.
- No 5985/5986 Sysmon Event ID 3 result was established.
- The WinRM Operational log primarily documented local configuration and WS-Man operations.
- No remote source host or remote user was established from the supplied evidence.

## Conclusion

This lab demonstrated how WinRM configuration and activity can be investigated without performing remote lateral movement.

The endpoint's WinRM service became operational after `winrm quickconfig`, and the resulting WinRM Operational events clearly documented service startup, firewall configuration, and successful WS-Man operations.

Although WinRM remote access and remote shell access were enabled, the available process, PowerShell, Security, and network evidence did not establish a remote PowerShell session or malicious lateral movement.

The final conclusion is:

> WinRM was successfully configured and operational on the endpoint, but the available evidence does not establish suspicious remote execution or lateral movement.

## Skills Demonstrated

- WinRM Investigation
- Windows Remote Management
- WS-Man Analysis
- WinRM Service Analysis
- Listener Analysis
- Firewall Analysis
- WinRM Operational Log Analysis
- PowerShell Investigation
- Sysmon Event ID 1
- Sysmon Event ID 3
- Security Event ID 4688
- PowerShell Event ID 4104
- Command History Analysis
- Configuration Change Analysis
- Telemetry Correlation
- Lateral Movement Assessment
