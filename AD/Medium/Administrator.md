# Administrator

---
# Machine Info
`Administrator` is a medium-difficulty Windows machine designed around a complete domain compromise scenario, where credentials for a low-privileged user are provided. To gain access to the michael account, ACLs (Access Control Lists) over privileged objects are enumerated, leading us to discover that the user olivia has GenericAll permissions over michael, allowing us to reset his password. With access as michael, it is revealed that he can force a password change on the user benjamin, whose password is reset. This grants access to FTP where a backup.psafe3 file is discovered, cracked, and reveals credentials for several users. These credentials are sprayed across the domain, revealing valid credentials for the user emily. Further enumeration shows that emily has GenericWrite permissions over the user ethan, allowing us to perform a targeted Kerberoasting attack. The recovered hash is cracked and reveals valid credentials for ethan, who is found to have DCSync rights ultimately allowing retrieval of the Administrator account hash and full domain compromise.

---
# Reconnaissance
## nmap
```
nmap -sCV 10.129.105.119 -o administrator.out --min-rate 10000 -p- -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-18 18:12 EST
Warning: 10.129.105.119 giving up on port because retransmission cap hit (10).
Nmap scan report for DC.administrator.htb (10.129.105.119)
Host is up (0.082s latency).
Not shown: 64201 closed tcp ports (reset), 1308 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-18 23:14:30Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
58177/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
58182/tcp open  msrpc         Microsoft Windows RPC
58189/tcp open  msrpc         Microsoft Windows RPC
58194/tcp open  msrpc         Microsoft Windows RPC
58208/tcp open  msrpc         Microsoft Windows RPC
63472/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-11-18T23:15:28
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 1m32s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.40 seconds
```
From `nmap` scan, the domain,`administrator.htb` was identified and it was added to `/etc/hosts` file to allow proper name resolution.
