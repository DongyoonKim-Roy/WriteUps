# Resolute (Medium)

# Machine Info
```
Resolute is an medium difficulty Windows machine that features Active Directory. 
The Active Directory anonymous bind is used to obtain a password that the sysadmins set for new user accounts, although it seems that the password for that account has since changed. 
A password spray reveals that this password is still in use for another domain user account, which gives us access to the system over WinRM. 
A PowerShell transcript log is discovered, which has captured credentials passed on the command-line. This is used to move laterally to a user that is a member of the DnsAdmins group. 
This group has the ability to specify that the DNS Server service loads a plugin DLL. 
After restarting the DNS service, we achieve command execution on the domain controller in the context of NT_AUTHORITY\SYSTEM.
```
