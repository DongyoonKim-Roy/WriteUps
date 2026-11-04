# Resolute (Medium)

# Machine Info
Resolute is a medium-difficulty Windows machine that involves Active Directory enumeration and privilege escalation.

Anonymous LDAP bind reveals a default password used for newly created accounts. Although the original account’s password has changed, a password spray shows that the same password is still valid for another domain user. This grants WinRM access to the system.

A PowerShell transcript log is then discovered, which contains credentials passed on the command line. These credentials are used to move laterally to a user in the `DnsAdmins` group. Members of this group can configure the DNS Server service to load a malicious plugin DLL.

After restarting the DNS service, command execution is achieved on the domain controller as `NT AUTHORITY\SYSTEM`.
