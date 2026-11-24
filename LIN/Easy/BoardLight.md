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

---
# Enumeration
## HTTP - TCP 80
<img width="1665" height="714" alt="image" src="https://github.com/user-attachments/assets/3f801da3-266b-493c-8800-46d877047943" />
<img width="912" height="637" alt="image" src="https://github.com/user-attachments/assets/12766956-6959-48cd-b444-1768f9f542fa" />
<img width="468" height="77" alt="image" src="https://github.com/user-attachments/assets/e3ea831c-42f4-42e4-9b4e-7a2ab2e80a13" />

The web application features several navigation sections, including `Home`, `About`, `What we do`, `Contact us`, and `Search`.\
A user input form was present, and basic `XSS` testing was performed, which yielded no results.\
The domain `board.htb` was also identified in the page footer, and it was added to `/etc/hosts` file.

## Directory Brute Force
<img width="865" height="902" alt="image" src="https://github.com/user-attachments/assets/fb9967da-ae02-4057-a3bc-a04a406bdbfd" />
<img width="876" height="425" alt="image" src="https://github.com/user-attachments/assets/9585ad73-37bc-4d69-a40b-3d6605b3b553" />

Directory brute-forcing revealed several accessible directories; however, none of them contained anything of interest.\
On the other hand, fuzzing identified a subdomain named `crm`.

## crm.board.htb
<img width="1258" height="677" alt="image" src="https://github.com/user-attachments/assets/fbde49fe-35a3-4c3e-8efd-3664c98116e8" />
<img width="1904" height="376" alt="image" src="https://github.com/user-attachments/assets/e3a0440a-31cc-4e8d-ac8a-99c879084fc3" />

There was login page was found and `Dolibarr v17.0.0` was running on the subdomain. Authentication with `admin:admin` was successful.

## CVE-2023-30253
<img width="1238" height="558" alt="image" src="https://github.com/user-attachments/assets/fb21a233-f17c-42a4-ad18-1c1004397db6" />
<img width="991" height="218" alt="image" src="https://github.com/user-attachments/assets/c27a9e5d-1a45-47f8-bfe8-11c434f40118" />

An exploit was found for `CVE-2023-30253` on GitHub. After running it, successfully got a shell as the `www-data` user.

---
# Shell as larissa
## Enumeration
<img width="279" height="46" alt="image" src="https://github.com/user-attachments/assets/6bf615a1-5268-47b9-acbd-4cca38cf85dc" />
<img width="639" height="314" alt="image" src="https://github.com/user-attachments/assets/920903ff-bd63-4559-ab60-302da5eb5dc3" />
<img width="548" height="54" alt="image" src="https://github.com/user-attachments/assets/054113b5-5949-46ef-bc48-61c6cfa13a31" />
<img width="684" height="44" alt="image" src="https://github.com/user-attachments/assets/2d409aad-43bc-4429-bf57-df23d5030420" />

`LinPEAS` enumeration yielded no useful findings.\ 
`sudo -l` required the `www-data` password, so it could not be run.
Checking `/var/www/html/crm.board.htb/htdocs/conf/conf.php` revealed the database credentials.\
With the recovered password, changing user to `larissa` was successful.\
In home directory, `user.txt` flag was found.

---
# Shell as root
<img width="780" height="146" alt="image" src="https://github.com/user-attachments/assets/5c808efe-069c-4bbe-afe7-e372e573e7bd" />
<img width="710" height="340" alt="image" src="https://github.com/user-attachments/assets/35959cae-fbbe-4bf7-bfd7-1a06a9f18e28" />

