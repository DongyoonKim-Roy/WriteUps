# Resolute (Medium)

# Machine Info
Resolute is a medium-difficulty Windows machine that involves Active Directory enumeration and privilege escalation.

Anonymous LDAP bind reveals a default password used for newly created accounts. Although the original account’s password has changed, a password spray shows that the same password is still valid for another domain user. This grants WinRM access to the system.

A PowerShell transcript log is then discovered, which contains credentials passed on the command line. These credentials are used to move laterally to a user in the `DnsAdmins` group. Members of this group can configure the DNS Server service to load a malicious plugin DLL.

After restarting the DNS service, command execution is achieved on the domain controller as `NT AUTHORITY\SYSTEM`.

# Reconnaissance
## nmap
```
nmap -sCV 10.129.100.9 -o resol.out --min-rate 10000 -p- -Pn                    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-04 09:05 EST
Nmap scan report for 10.129.100.9
Host is up (0.067s latency).
Not shown: 65511 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Simple DNS Plus
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-11-04 14:04:31Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49684/tcp open  msrpc        Microsoft Windows RPC
49788/tcp open  msrpc        Microsoft Windows RPC
49817/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: RESOLUTE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-11-04T14:05:24
|_  start_date: 2026-11-04T13:58:20
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Resolute
|   NetBIOS computer name: RESOLUTE\x00
|   Domain name: megabank.local
|   Forest name: megabank.local
|   FQDN: Resolute.megabank.local
|_  System time: 2026-11-04T06:05:23-08:00
|_clock-skew: mean: 2h38m35s, deviation: 4h37m09s, median: -1m26s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 76.80 seconds
```
From namp scan, the hostname `resolute` and the domain `megabank.local` were identified. Both were added both to the `/etc/hosts` file to allow proper name resolution: 
```
10.129.100.9  RESOLUTE.megabank.local megabank.local RESOLUTE
```

# Enumeration
## smb - TCP 445
```
nxc smb 10.129.100.9 --generate-hosts-file smb.hosts  
SMB         10.129.100.9    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
```
```
I used nxc smb IP_ADDR --generate-hosts-file smb.hosts to make sure the hostname and domain both are correct.
```
