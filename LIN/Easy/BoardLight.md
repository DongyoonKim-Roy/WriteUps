# BoardLight

# Machine Info
BoardLight is an easy difficulty Linux machine that features a Dolibarr instance vulnerable to CVE-2023-30253. This vulnerability is leveraged to gain access as www-data. After enumerating and dumping the web configuration file contents, plaintext credentials lead to SSH access to the machine. Enumerating the system, a SUID binary related to enlightenment is identified which is vulnerable to privilege escalation via CVE-2022-37706 and can be abused to leverage a root shell.

# Enumeration
## nmap
```
nmap -sCV 10.129.231.37 -o board.out --min-rate 10000 -p- -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-23 20:26 EST
Warning: 10.129.231.37 giving up on port because retransmission cap hit (10).
Nmap scan report for board.htb (10.129.231.37)
Host is up (0.088s latency).
Not shown: 65442 closed tcp ports (reset), 91 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.14 seconds
```

The `nmap` scan revealed that only ports `22` (SSH) and `80` (HTTP) were open on the target.

# Enumeration
## HTTP - TCP 80
<img width="1665" height="714" alt="image" src="https://github.com/user-attachments/assets/3f801da3-266b-493c-8800-46d877047943" />
<img width="912" height="637" alt="image" src="https://github.com/user-attachments/assets/12766956-6959-48cd-b444-1768f9f542fa" />
<img width="468" height="77" alt="image" src="https://github.com/user-attachments/assets/e3ea831c-42f4-42e4-9b4e-7a2ab2e80a13" />

