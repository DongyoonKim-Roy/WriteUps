# sauna

# Machine Info
Sauna is an easy difficulty Windows machine that features Active Directory enumeration and exploitation. Possible usernames can be derived from employee full names listed on the website. With these usernames, an ASREPRoasting attack can be performed, which results in hash for an account that doesn&amp;#039;t require Kerberos pre-authentication. This hash can be subjected to an offline brute force attack, in order to recover the plaintext password for a user that is able to WinRM to the box. Running WinPEAS reveals that another system user has been configured to automatically login and it identifies their password. This second user also has Windows remote management permissions. BloodHound reveals that this user has the DS-Replication-Get-Changes-All extended right, which allows them to dump password hashes from the Domain Controller in a DCSync attack. Executing this attack returns the hash of the primary domain administrator, which can be used with Impacket psexec.py in order to gain a shell on the box as NT_AUTHORITY\SYSTEM.

---
# Reconnaissance

## nmap
```
nmap -sCV 10.129.95.180 -o sauna.out --min-rate 10000 -p- -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-23 14:35 EST
Nmap scan report for 10.129.95.180
Host is up (0.065s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Egotistical Bank :: Home
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-23 19:35:33Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
49700/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows
```
The `nmap` scan identified the domain `egotistical-bank.local` and it was added to `/etc/hosts` file.

# Enumeration
## HTTP - TCP 80
<img width="1452" height="1250" alt="image" src="https://github.com/user-attachments/assets/3a042247-7fc6-432f-a3a8-7e5784dc4a78" />
<img width="1191" height="805" alt="image" src="https://github.com/user-attachments/assets/10ae0723-fe20-4bcb-b50b-988af1d291e2" />

The application features multiple navigation tabs, including `Home`, `About Us`, `Blogs`, `Dropdown`, `Contact Us`, and `Apply Now`.\
While the `Home`, `Blogs`, `Dropdown`, and `Contact Us` sections did not contain any sensitive information, the `About Us` page revealed several usernames. The usernames are saved into `users.txt` file.

```
/smb/username-anarchy/username-anarchy --input-file users.txt
fergus
fergussmith
fergus.smith
fergussm
fergsmit
ferguss
f.smith
fsmith
sfergus
s.fergus
smithf
smith
smith.f
smith.fergus
fs
shaun
shauncoins
shaun.coins
shauncoi
shaucoin
shaunc
s.coins
scoins
cshaun
c.shaun
coinss
coins
coins.s
coins.shaun
sc
hugo
hugobear
hugo.bear
hugob
h.bear
hbear
bhugo
b.hugo
bearh
bear
bear.h
bear.hugo
hb
bowie
bowietaylor
bowie.taylor
bowietay
bowitayl
bowiet
b.taylor
btaylor
tbowie
t.bowie
taylorb
taylor
taylor.b
taylor.bowie
bt
sophie
sophiedriver
sophie.driver
sophiedr
sophdriv
sophied
s.driver
sdriver
dsophie
d.sophie
drivers
driver
driver.s
driver.sophie
sd
steven
stevenkerb
steven.kerb
stevenke
stevkerb
stevenk
s.kerb
skerb
ksteven
k.steven
kerbs
kerb
kerb.s
kerb.steven
sk
```
Subsequently, the `username-anarchy` tool was used to generate additional username permutations based on the names identified on the `About Us` page.

## Kerberos - TCP 88 
<img width="1369" height="171" alt="image" src="https://github.com/user-attachments/assets/b710659b-d2ec-4540-ae2b-5deacea44783" />

Using the enumerated usernames, `impacket-GetNPUsers` was run against the domain controller. This resulted in the successful retrieval of a TGT for the account `fsmith`, and saved into `hash.txt` file.

<img width="153" height="162" alt="image" src="https://github.com/user-attachments/assets/331eac78-2577-4ccd-aebe-2d795e2eaf31" />

Using `John the Ripper`, the password was recovered as `Thestrokes23`.

# Shell as fsmith
<img width="871" height="105" alt="image" src="https://github.com/user-attachments/assets/d7013543-1f45-429a-b1c4-f7a0a22e7ee1" />
<img width="486" height="313" alt="image" src="https://github.com/user-attachments/assets/89cbd2be-868b-4742-bf31-a905ddcb9ebb" />

Using the compromised credentials, remote access was established through `evil-winrm` as `fsmith`. This session allowed retrieval of the `user.txt` flag.

## Enumeration
<img width="1047" height="626" alt="image" src="https://github.com/user-attachments/assets/f345b003-b8e6-488e-9f36-15ec6fb0dfa3" />
<img width="587" height="112" alt="image" src="https://github.com/user-attachments/assets/f38a4525-26ce-410d-80c1-dc67a3adb6f6" />

The account `fsmith` held no special `privileges` or `group memberships`. After uploading and executing `WinPEAS`, the credentials for the service account `svc_loanmgr` (`Moneymakestheworldgoround!`) were identified.

# Shell as Administrator
## Evil-Winrm
<img width="684" height="757" alt="image" src="https://github.com/user-attachments/assets/6fbd4b57-c7a7-4057-9e6d-eb54da1ae70e" />

The service account `svc_loanmgr` possessed the `GetChanges` and `GetChangesAll` rights on the domain object `egotistical-bank.local`.

<img width="717" height="117" alt="image" src="https://github.com/user-attachments/assets/fa3638bf-69d3-452c-bda4-9253e3300301" />
<img width="500" height="173" alt="image" src="https://github.com/user-attachments/assets/b9d59216-54bb-4f7e-b0ef-473ad3e510c3" />

Leveraging the `GetChanges` and `GetChangesAll` privileges, a DCSync attack was performed with `impacket-secretsdump`, resulting in the extraction of the `administrator` NTLM hash. Pass-the-hash authentication as `administrator` was then successful, allowing retrieval of the `root.txt` flag.
