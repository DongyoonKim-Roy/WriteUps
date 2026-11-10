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

<img width="887" height="677" alt="image" src="https://github.com/user-attachments/assets/cf735812-8a6c-4b1a-a2b7-5751937cef59" />
Default credentials `(kevin:iNa2we6haRj2gaw!)` were provided for the box. After successfully authenticating with these credentials, a RID brute-force attack was conducted. Discovered user names were saved into `users.txt` file. 


