# Troubleshooting Notes 

## 1. Initial `winrm get winrm/config` Failure

### Problem

The initial WinRM configuration query failed because the client could not connect to the destination.

The WinRM Operational log recorded corresponding failed WS-Man operations.

### Resolution

The WinRM configuration was initialized using:

```powershell
winrm quickconfig
```

### DFIR Lesson

The configuration process itself generated useful forensic events.

## 2. WinRM Service Was Initially Not Operational

### Observation

The initial WS-Man requests failed.

After configuration, the service was:

```text
Running
Automatic
```

### Resolution

`winrm quickconfig` configured the required local WinRM components.

## 3. WinRM Listener

### Observation

The final listener was:

```text
Transport : HTTP
Port      : 5985
Enabled   : true
```

### Interpretation

WinRM was listening for HTTP-based WS-Man communication.

This does not prove that a remote host actually connected.

## 4. Firewall Rule

### Observation

The following rule was enabled:

```text
Windows Remote Management (HTTP-In)
```

for:

```text
Domain, Private
```

### Interpretation

The firewall permitted inbound WinRM traffic on the applicable profiles.

This is configuration evidence, not proof of remote use.

## 5. LocalAccountTokenFilterPolicy Event

The WinRM Operational log recorded:

```text
Configured LocalAccountTokenFilterPolicy to grant administrative rights remotely to local users.
```

### Interpretation

This event is part of the configuration process observed after `winrm quickconfig`.

It should not automatically be interpreted as attacker activity.

## 6. Service Command Line

The WinRM service path was:

```text
C:\Windows\System32\svchost.exe -k NetworkService -p
```

### Important

The entire string should not be passed to file-analysis commands as though it were a filename.

The executable is:

```text
C:\Windows\System32\svchost.exe
```

The remaining values are service arguments.

This is consistent with the parsing lesson from Lab 65.

## 7. No `wsmprovhost.exe` Evidence

### Observation

The Sysmon Event ID 1 search for:

```text
wsmprovhost.exe
```

returned no relevant result.

### Interpretation

No remote PowerShell provider-host process was established from the available evidence.

### DFIR Lesson

The absence of `wsmprovhost.exe` does not prove that remote management never occurred, but it prevents us from using that artifact as supporting evidence.

## 8. No Relevant Security Event ID 4688

### Observation

The search for:

```text
powershell.exe
wsmprovhost.exe
```

returned no result.

### Interpretation

Security process-creation telemetry did not provide direct evidence of remote PowerShell execution.

## 9. No Relevant PowerShell Event ID 4104

### Observation

The targeted search for:

```text
WinRM
Enter-PSSession
Invoke-Command
New-PSSession
```

returned no output.

### Interpretation

No targeted Script Block Logging evidence was identified.

## 10. Sysmon Event ID 3

### Observation

The query searched for:

```text
5985
5986
```

No matching result was returned.

### Interpretation

No WinRM-port network activity was established from the supplied search.

## 11. PowerShell History Problem

### Problem

The command:

```powershell
Get-PSReadLineOption
```

was not available in one session.

### Result

Persistent history could not be accessed using that command.

### Resolution

Current-session history was checked with:

```powershell
Get-History
```

### DFIR Lesson

Current-session command history and persistent PSReadLine history are different evidence sources.

## 12. WinRM Configuration vs WinRM Activity

One of the most important troubleshooting distinctions was:

```text
WinRM Enabled
```

does not mean:

```text
Remote WinRM Session
```

Likewise:

```text
Listener on 5985
```

does not mean:

```text
Malicious Lateral Movement
```

Both require additional process, authentication, source-host, and network evidence.

