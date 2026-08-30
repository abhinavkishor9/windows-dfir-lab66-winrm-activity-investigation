# Timeline 

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

