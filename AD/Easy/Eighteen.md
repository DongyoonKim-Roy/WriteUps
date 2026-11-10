# Eighteen

# Machine Info
`Eighteen` is an easy difficulty Windows machine that demonstrates common weaknesses in database access control and Active Directory misconfigurations. Initial access is obtained by identifying valid MSSQL credentials and abusing impersonation privileges to access backend application data. Extracted password hashes are cracked offline and used for password spraying, leading to WinRM access as a domain user. Privilege escalation is achieved by abusing delegated permissions in Active Directory. Specifically, membership in a group with CreateChild rights over an Organisational Unit allows exploitation of the BadSuccessor technique to create a delegated Managed Service Account (dMSA) linked to the Administrator account, ultimately leading to full domain compromise.

---
# Reconnaissance
## nmap
```
nmap -sCV 10.129.102.80 -o eighteen.out --min-rate 10000 -p- -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-10 14:52 EST
Nmap scan report for 10.129.102.80
Host is up (0.085s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Microsoft IIS httpd 10.0
|_http-title: Did not follow redirect to http://eighteen.htb/
|_http-server-header: Microsoft-IIS/10.0
1433/tcp open  ms-sql-s Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.129.102.80:1433: 
|     Target_Name: EIGHTEEN
|     NetBIOS_Domain_Name: EIGHTEEN
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: eighteen.htb
|     DNS_Computer_Name: DC01.eighteen.htb
|     DNS_Tree_Name: eighteen.htb
|_    Product_Version: 10.0.26100
|_ssl-date: 2026-11-10T22:54:23+00:00; +3h01m43s from scanner time.
| ms-sql-info: 
|   10.129.102.80:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-11-10T22:51:48
|_Not valid after:  2056-11-10T22:51:48
5985/tcp open  http     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 3h01m42s, deviation: 0s, median: 3h01m42s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.50 seconds
```
The nmap scan identified the domain `eighteen.htb`, which was added to `/etc/hosts`. Additionally, MSSQL was found to be listening on the target.

---
# Enumeration
## HTTP - TCP 80
<img width="1332" height="1147" alt="image" src="https://github.com/user-attachments/assets/7f94c025-8b2b-4913-b7b1-5e56fe8d92a8" />
<img width="1203" height="875" alt="image" src="https://github.com/user-attachments/assets/d8913cb7-fb62-4270-b7e9-15733862dca8" />
<img width="1380" height="1213" alt="image" src="https://github.com/user-attachments/assets/5c26c4de-cfd0-436f-80b1-706375c24fe1" />
<img width="1190" height="239" alt="image" src="https://github.com/user-attachments/assets/b1d1c2f8-dd7d-4b10-a110-88c7120cd70f" />

The site includes login and registration functionality. After registering an account, nothing of interest was discovered. Access to the admin page was denied. To access the page, it needs admin credentials.

---
### Directory Bruteforce

<img width="825" height="567" alt="image" src="https://github.com/user-attachments/assets/5a2b5ab4-e0fa-4a90-912c-be7f059e0b48" />
<img width="873" height="401" alt="image" src="https://github.com/user-attachments/assets/68bf7c76-361d-40c8-96eb-b53f65eab57f" />

Directory Bruteforce was conducted, but nothing interesting was found.

---
## MSSQL - TCP 1433
<img width="886" height="69" alt="image" src="https://github.com/user-attachments/assets/6e492bd1-b940-4206-a6d3-a85da3253657" />

<img width="887" height="677" alt="image" src="https://github.com/user-attachments/assets/cf735812-8a6c-4b1a-a2b7-5751937cef59" />

Default credentials `kevin:iNa2we6haRj2gaw!` were provided for the box. After successfully authenticating with these credentials, a RID brute-force attack was conducted. Discovered user names were saved into `users.txt` file. 


<img width="896" height="794" alt="image" src="https://github.com/user-attachments/assets/4e65eaf1-cb1c-4281-988e-7e40b746df8b" />
<img width="862" height="378" alt="image" src="https://github.com/user-attachments/assets/0f0e14bb-cc9c-4a64-bbb8-516ed6f6819e" />
<img width="776" height="134" alt="image" src="https://github.com/user-attachments/assets/74a73303-396c-47af-b735-892c569b8b43" />

User kevin lacks the privileges required to enable or run xp_cmdshell. A hash was also captured via the SMB server, but cracking attempts were unsuccessful.

<img width="891" height="188" alt="image" src="https://github.com/user-attachments/assets/e4c736ea-8444-439a-968e-515baff5b777" />

Among the available databases, `financial_planner` stood out, but `kevin` did not have permission to access it. No interesting data was found in the remaining databases.

<img width="914" height="187" alt="image" src="https://github.com/user-attachments/assets/f6d9ab25-8c83-40d8-8164-ba3a85538082" />
<img width="387" height="43" alt="image" src="https://github.com/user-attachments/assets/930a86bd-20d2-41f5-aa18-42c0ca0b2610" />

During user enumeration, the `appdev` account was identified. Authentication as `appdev` succeeded.

<img width="570" height="195" alt="image" src="https://github.com/user-attachments/assets/f26bd8ce-5d17-418d-add9-1054d244e7c7" />
<img width="1183" height="147" alt="image" src="https://github.com/user-attachments/assets/4682a4c3-d6e4-4895-9e0b-73a9e204b664" />

With access to the `financial_planner` database as `appdev`, an admin password hash was recovered from the `users` table.

# Shell as adam.scott
## Recover Hash
<img width="825" height="43" alt="image" src="https://github.com/user-attachments/assets/9c590a7b-31d2-4d90-9930-5ab8c316ed1c" />

Recovered hash was saved in the `hash.txt` file. It is identified as PBKDF2-HMAC-SHA256 hash.

<img width="810" height="103" alt="image" src="https://github.com/user-attachments/assets/0344addf-30fa-46d3-bd3e-db1c06a4bab0" />
<img width="407" height="49" alt="image" src="https://github.com/user-attachments/assets/52f0d911-783b-4b84-98a4-51d004956d43" />
<img width="757" height="45" alt="image" src="https://github.com/user-attachments/assets/75f773c8-2c76-44eb-9888-89d1d67b44f2" />

Research led to a useful reference (https://notes.benheater.com/books/hash-cracking/page/pbkdf2-hmac-sha256) on cracking `PBKDF2-HMAC-SHA256` hashes. Following the guidance, the plaintext password, `iloveyou1`, was recovered.

---
## Login Admin Portal
<img width="1256" height="1117" alt="image" src="https://github.com/user-attachments/assets/dffa107d-ff43-4269-bbe7-ccd264303f03" />
Access to the admin portal with the recovered password was successful, but no useful information was found.

---
## Password Spray
<img width="810" height="55" alt="image" src="https://github.com/user-attachments/assets/36e88e07-b0b5-411d-a7fa-ccfbe7abf881" />
After trying a few approaches, no foothold was obtained. A password spray was then conducted using the enumerated usernames and the recovered password, which successfully authenticated as `adam.scott`.

## Evil-winrm
<img width="428" height="40" alt="image" src="https://github.com/user-attachments/assets/7334d164-dd19-418c-a15e-eecc0e9c3708" />
<img width="485" height="174" alt="image" src="https://github.com/user-attachments/assets/305d86e4-66ed-4b50-b7af-72a0def331c8" />/
Authentication via Evil-WinRM as adam.scott was successful, and the user flag was retrieved.
