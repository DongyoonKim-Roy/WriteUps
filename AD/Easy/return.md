# Machine Info
`Return` is an easy difficulty Windows machine featuring a network printer administration panel that stores LDAP credentials. These credentials can be captured by inputting a malicious LDAP server which allows obtaining foothold on the server through the WinRM service. User found to be part of a privilege group which further exploited to gain system access.

---
# Reconnaissance
## nmap
```
# Nmap 7.95 scan initiated Tue Nov 17 14:41:08 2026 as: /usr/lib/nmap/nmap --privileged -sCV -o return.out --min-rate 10000 -p- -Pn 10.129.95.241
Warning: 10.129.95.241 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.95.241
Host is up (0.069s latency).
Not shown: 65510 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: HTB Printer Admin Panel
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-17 13:00:04Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: return.local0., Site: Default-First-Site-Name)
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
49671/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         Microsoft Windows RPC
49678/tcp open  msrpc         Microsoft Windows RPC
49681/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: PRINTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -6h41m23s
| smb2-time: 
|   date: 2026-11-17T13:01:00
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Nov 17 14:42:33 2026 -- 1 IP address (1 host up) scanned in 85.23 seconds
```
The `nmap` scan identified the domain `return.local`, which was added to `/etc/hosts`.

---
# Enumeration
## HTTP - TCP 80
<img width="1164" height="679" alt="image" src="https://github.com/user-attachments/assets/dfe1e0bc-21e9-48b0-83a1-a1b2a85726d4" />
<img width="1034" height="548" alt="image" src="https://github.com/user-attachments/assets/dc877217-f6a1-49f7-9cef-336d249203da" />

The website includes Home, Settings, Fax, and Troubleshooting tabs, but the Fax and Troubleshooting tabs don’t respond when clicked.\
In the `Settings` tab, with the given information, `Server Address`, `Server Port`, `Username`, `Password`, clicking `Update` button doesn't respond.

<img width="1302" height="342" alt="image" src="https://github.com/user-attachments/assets/1d89649b-3a68-487e-939a-9a7e8f81aa8b" />
However, it responses with username : `svc-printer` and password : `1edFg43012!!` after `Server Address` with attacker's `ip address`. 




