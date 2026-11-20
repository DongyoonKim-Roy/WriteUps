# Monteverde
# Machine Info
Monteverde is a Medium Windows machine that features Azure AD Connect. The domain is enumerated and a user list is created. Through password spraying, the SABatchJobs service account is found to have the username as a password. Using this service account, it is possible to enumerate SMB Shares on the system, and the $users share is found to be world-readable. An XML file used for an Azure AD account is found within a user folder and contains a password. Due to password reuse, we can connect to the domain controller as mhope using WinRM. Enumeration shows that Azure AD Connect is installed. It is possible to extract the credentials for the account that replicates the directory changes to Azure (in this case the default domain administrator).

---
# Reconnaissance
## nmap
```
nmap -sCV 10.129.228.111 -o monte.out --min-rate 10000 -p- -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-20 13:50 EST
Nmap scan report for MONTEVERDE.MEGABANK.LOCAL (10.129.228.111)
Host is up (0.067s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-20 11:49:56Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49750/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows
```
From `nmap` scan, the domain, megabank.local, and Domain Controller, `Monteverde` were found.\
They were added to `/etc/hosts` file to allow proper name resolution.

---
## LDAP - TCP 389
<img width="890" height="269" alt="image" src="https://github.com/user-attachments/assets/37fcd468-bf22-444f-b26e-4b42fbbad6cd" />
<img width="163" height="160" alt="image" src="https://github.com/user-attachments/assets/5b95e8f2-2001-49da-9fef-d5ff7d75d8c8" />


`impacket-GetADUsers` successfully enumerated multiple users, which were then saved to `users.txt`.

---
# Enumeration
## SMB - TCP 445
<img width="969" height="282" alt="image" src="https://github.com/user-attachments/assets/9092e468-3f04-499f-833b-a7ed347391f5" />

`Null session`, `guest`, and `anonymous` authentication were unsuccessful.\
Since there were no known passwords, the `users.txt` file was used as the password list.\
This successfully authenticated the user `SABatchJobs` with the password `SABatchJobs`.

<img width="981" height="246" alt="image" src="https://github.com/user-attachments/assets/1f1ee72e-f232-423b-bb1a-c07c02455cdd" />
<img width="625" height="162" alt="image" src="https://github.com/user-attachments/assets/e5afe75a-443c-4ee6-b52c-a3fae578c587" />
<img width="615" height="76" alt="image" src="https://github.com/user-attachments/assets/4fdb2679-a73a-4014-aceb-3ed3f6228a6a" />
<img width="731" height="265" alt="image" src="https://github.com/user-attachments/assets/59a142f0-dc05-4fa6-80ef-66bf51d178b5" />
<img width="194" height="54" alt="image" src="https://github.com/user-attachments/assets/4291ca50-2c48-4935-affa-1a2383a915ec" />

The user `SABatchJobs` had access to several shares.\
Nothing interesting was found in `IPC$`, `SYSVOL`, or `NETLOGON`.\
However, in the `users$` share, there was a file named `azure.xml` inside the `mhope` directory.\
In `azure.xml`, there is a password, `4n0therD4y@n0th3r$`, and it was saved into `passwords.txt` file.

---
# Shell as mhope
<img width="487" height="169" alt="image" src="https://github.com/user-attachments/assets/32ebbc8b-64b8-4292-afdd-8ed0db0a001a" />

Using the discovered password, we successfully logged into the host as `mhope` via `evil-winrm` and obtained the `user.txt` flag.

## Enumeration
<img width="1283" height="664" alt="image" src="https://github.com/user-attachments/assets/398e1ba6-6ac1-467c-90d8-e3435c8956e6" />

The user `mhope` doesn't have special privilege, but it is in `Azure Admins` group.

<img width="704" height="218" alt="image" src="https://github.com/user-attachments/assets/8b0bf9dd-fe74-463b-a1b0-267fe2cb3873" />

In the `mhope` directory, there was a `.Azure` folder containing several files, but nothing interesting was found.

<img width="805" height="199" alt="image" src="https://github.com/user-attachments/assets/59847b0d-6770-4b8e-b503-b95b3c8c3dd8" />
