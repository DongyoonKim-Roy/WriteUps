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

cat smb.hosts 
10.129.100.9     RESOLUTE.megabank.local megabank.local RESOLUTE
```

I used the command to make sure the hostname and domain both are correct.

---
```
nxc smb 10.129.100.9 -u '' -p '' --shares           
SMB         10.129.100.9    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.100.9    445    RESOLUTE         [+] megabank.local\: 
SMB         10.129.100.9    445    RESOLUTE         [-] Error enumerating shares: STATUS_ACCESS_DENIED

nxc smb 10.129.100.9 -u 'anonymous' -p '' --shares
SMB         10.129.100.9    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\anonymous: STATUS_LOGON_FAILURE

nxc smb 10.129.100.9 -u 'guest' -p '' --shares
SMB         10.129.100.9    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\guest: STATUS_ACCOUNT_DISABLED
```

Null session authentication was accepted, but share listing was denied.
Anonymous login failed, and the guest account was disabled.

---
```
nxc smb 10.129.100.9 -u '' -p '' --users
SMB         10.129.100.9    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.100.9    445    RESOLUTE         [+] megabank.local\: 
SMB         10.129.100.9    445    RESOLUTE         -Username-                    -Last PW Set-       -BadPW- -Description-         
SMB         10.129.100.9    445    RESOLUTE         Administrator                 2026-11-04 14:29:03 0       Built-in account for administering the computer/domain                                                                                                    
SMB         10.129.100.9    445    RESOLUTE         Guest                         <never>             0       Built-in account for guest access to the computer/domain                                                                                                  
SMB         10.129.100.9    445    RESOLUTE         krbtgt                        2019-09-25 13:29:12 0       Key Distribution Center Service Account                                                                                                                   
SMB         10.129.100.9    445    RESOLUTE         DefaultAccount                <never>             0       A user account managed by the system.                                                                                                                     
SMB         10.129.100.9    445    RESOLUTE         ryan                          2026-11-04 14:28:02 0        
SMB         10.129.100.9    445    RESOLUTE         marko                         2019-09-27 13:17:14 0       Account created. Password set to Welcome123!                                                                                                              
SMB         10.129.100.9    445    RESOLUTE         sunita                        2019-12-03 21:26:29 0        
SMB         10.129.100.9    445    RESOLUTE         abigail                       2019-12-03 21:27:30 0        
SMB         10.129.100.9    445    RESOLUTE         marcus                        2019-12-03 21:27:59 0        
SMB         10.129.100.9    445    RESOLUTE         sally                         2019-12-03 21:28:29 0        
SMB         10.129.100.9    445    RESOLUTE         fred                          2019-12-03 21:29:01 0        
SMB         10.129.100.9    445    RESOLUTE         angela                        2019-12-03 21:29:43 0        
SMB         10.129.100.9    445    RESOLUTE         felicia                       2019-12-03 21:30:53 0        
SMB         10.129.100.9    445    RESOLUTE         gustavo                       2019-12-03 21:31:42 0        
SMB         10.129.100.9    445    RESOLUTE         ulf                           2019-12-03 21:32:19 0        
SMB         10.129.100.9    445    RESOLUTE         stevie                        2019-12-03 21:33:13 0        
SMB         10.129.100.9    445    RESOLUTE         claire                        2019-12-03 21:33:44 0        
SMB         10.129.100.9    445    RESOLUTE         paulo                         2019-12-03 21:34:46 0        
SMB         10.129.100.9    445    RESOLUTE         steve                         2019-12-03 21:35:25 0        
SMB         10.129.100.9    445    RESOLUTE         annette                       2019-12-03 21:36:55 0        
SMB         10.129.100.9    445    RESOLUTE         annika                        2019-12-03 21:37:23 0        
SMB         10.129.100.9    445    RESOLUTE         per                           2019-12-03 21:38:12 0        
SMB         10.129.100.9    445    RESOLUTE         claude                        2019-12-03 21:39:56 0        
SMB         10.129.100.9    445    RESOLUTE         melanie                       2026-11-04 14:29:03 0        
SMB         10.129.100.9    445    RESOLUTE         zach                          2019-12-04 10:39:27 0        
SMB         10.129.100.9    445    RESOLUTE         simon                         2019-12-04 10:39:58 0        
SMB         10.129.100.9    445    RESOLUTE         naoki                         2019-12-04 10:40:44 0        
SMB         10.129.100.9    445    RESOLUTE         [*] Enumerated 27 local users: MEGABANK

cat users.txt 
ryan
marko
sunita
abigail
marcus
sally
fred
angela
felicia
gustavo
ulf
stevie
claire
paulo
steve
annette
annika
per
claude
melanie
zach
simon
naoki
```

Since null session authentication was accepted, I tried to enumerate users. 
This returned a list of domain users, and the description for the user `marko` indicated that the password was set to `Welcome123!`.

---
