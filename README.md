# HTB Write-ups

Write-ups from Hack The Box machines and HTB Academy modules I've completed. Flags are redacted.

## Machines

| Machine | Difficulty | Summary |
|---|---|---|
| [GreenHorn](greenhorn-writeup/) | Easy (Linux) | Leaked hash in a public Gitea repo, cracked with Hashcat, Pluck CMS module upload to a shell, password reuse for the user flag |
| [Soccer](soccer-writeup/) | Easy (Linux) | Default creds on Tiny File Manager, PHP upload to a shell, hidden vhost, and blind SQL injection over a WebSocket for the user flag |

## HTB Academy

| Module | Write-up | Topics |
|---|---|---|
| Getting Started | [GetSimple CMS](getsimple-cms.md) | Web enumeration, default creds, PHP web shell, sudo privilege escalation |
| Using the Metasploit Framework | [Metasploit](metasploit-framework.md) | Module search, payloads, sessions, Meterpreter, Kiwi |
| Password Attacks | [Network Services](network-services-password-attacks.md) | Password spraying against WinRM, SSH, RDP, and SMB |

I only publish retired machines and Tier 0 Academy modules, per HTB's rules.
