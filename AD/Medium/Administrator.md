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

# Enumeration
## FTP - TCP 21
<img width="314" height="159" alt="image" src="https://github.com/user-attachments/assets/e75f729a-064c-414b-8c05-97320a8e5662" />
<img width="259" height="169" alt="image" src="https://github.com/user-attachments/assets/a335e0e5-6561-4351-bf08-91f4a7a5b12c" />
<img width="432" height="157" alt="image" src="https://github.com/user-attachments/assets/60cf3054-8dd5-405b-9ab6-e2ac3aa3fa40" />

The `nmap` scan identified an open `FTP` service. Attempts to authenticate using the `olivia`, `anonymous` and `ftp` accounts were unsuccessful.

## SMB - TCP 445
<img width="854" height="79" alt="image" src="https://github.com/user-attachments/assets/3da1e869-5e12-440c-8808-0a66112cb3b0" />
<img width="848" height="332" alt="image" src="https://github.com/user-attachments/assets/282faaea-de65-4a4b-b6b7-1345dd962520" />
<img width="853" height="202" alt="image" src="https://github.com/user-attachments/assets/ae598900-13ca-40a8-93af-5ce08e21628e" />
<img width="148" height="132" alt="image" src="https://github.com/user-attachments/assets/19e54af3-1966-4bcb-b467-a95bb4ef0046" />

Authentication to `SMB` succeeded with the credentials `olivia`:`ichliebedich`. The enumerated usernames were written to `users.txt`. Multiple shares were identified, though none contained anything of interest.

---
# Shell as Olivia
<img width="408" height="40" alt="image" src="https://github.com/user-attachments/assets/7b5b7c34-7437-451e-a441-5e78d0631fea" />

Successfully authenticated to the target machine using the provided credentials. However, no user flag was found.

## Enumeration
<img width="1012" height="655" alt="image" src="https://github.com/user-attachments/assets/f8efba31-6b3b-4c0c-b4e9-b065da854a4e" />
<img width="529" height="209" alt="image" src="https://github.com/user-attachments/assets/81bf876a-03a0-4d1e-97bd-61cb7302c3a0" />
<img width="618" height="624" alt="image" src="https://github.com/user-attachments/assets/ae1a7621-4ddd-43f9-8623-21e6b4fd23e8" />

The authenticated user did not possess useful privileges and was not a member of any privileged groups. No items of interest were discovered in the home directory. Although the user `emily` was identified under `Users`, access was not permitted.

### BloodHound
<img width="1096" height="401" alt="image" src="https://github.com/user-attachments/assets/a2343002-6664-4255-b84e-0963a9ba120c" />
<img width="653" height="545" alt="image" src="https://github.com/user-attachments/assets/c7160f65-f144-4965-a386-50c092924abf" />
<img width="662" height="137" alt="image" src="https://github.com/user-attachments/assets/ba0a3503-c6f1-4230-a0c6-efb9153c8874" />

`Olivia` possessed `GenericAll` privileges on `Michael`. The `Michael` account was Kerberoasted and a hash was obtained, though password recovery was unsuccessful.

<img width="1168" height="163" alt="image" src="https://github.com/user-attachments/assets/78a8472f-63d4-4d38-b809-bf1f0b5f1786" />

Since `Olivia` has `GenericAll` rights over the `Michael` account, she can reset his password. The new password was set to `password-1`.

<img width="674" height="173" alt="image" src="https://github.com/user-attachments/assets/72d25b39-2d54-49f6-a132-7495803a5290" />
<img width="1004" height="40" alt="image" src="https://github.com/user-attachments/assets/26ed97ec-adee-437f-b74a-576e24ae9fd1" />
<img width="1152" height="75" alt="image" src="https://github.com/user-attachments/assets/f82ab0c0-bc39-45e9-8fc0-f3fb0d77ac80" />

Then, `Michael` has `ForceChangePassword` Privilege over the `Benjamin` account. The new password was set to `password-1` as well. The user was able to access to `SMB` service.

# Shell as Emily
