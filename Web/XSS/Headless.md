# Headless (Easy)

# Machine Info
Headless is an easy-difficulty Linux machine that features a Python Werkzeug server hosting a website. The website has a customer support form, which is found to be vulnerable to blind Cross-Site Scripting (XSS) via the User-Agent header. This vulnerability is leveraged to steal an admin cookie, which is then used to access the administrator dashboard. The page is vulnerable to command injection, leading to a reverse shell on the box. Enumerating the user’s mail reveals a script that does not use absolute paths, which is leveraged to get a shell as root.

# Reconnaissance
## nmap
```
nmap -sCV 10.129.100.124 -o headless.out --min-rate 10000 -p- -Pn
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-05 15:35 EST
Nmap scan report for 10.129.100.124
Host is up (0.067s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 90:02:94:28:3d:ab:22:74:df:0e:a3:b2:0f:2b:c6:17 (ECDSA)
|_  256 2e:b9:08:24:02:1b:60:94:60:b3:84:a9:9e:1a:60:ca (ED25519)
5000/tcp open  http    Werkzeug httpd 2.2.2 (Python 3.11.2)
|_http-server-header: Werkzeug/2.2.2 Python/3.11.2
|_http-title: Under Construction
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.06 seconds
```
From the nmap scan, `port 5000` is running Werkzeug / Python

# Enumeration
## HTTP - TCP 5000

### Site
<img width="455" height="314" alt="image" src="https://github.com/user-attachments/assets/614b727e-adc3-43d7-83d5-2f7af30466be" />\
<img width="291" height="712" alt="image" src="https://github.com/user-attachments/assets/af670b0e-f643-4921-b551-687af2a8df29" />

The button redirects to `/support`, where there is a contact form. However, no feedback is displayed after submitting the form.

---
### Directory Brute-Force
<img width="651" height="578" alt="image" src="https://github.com/user-attachments/assets/1dfd6d08-0404-4ec8-aa88-502e656a2cbb" />

```
feroxbuster -u http://10.129.100.124:5000/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -C 404,403 -x php,html,txt,py
```

The scan found two accessible directories — `/support` and `/dashboard`.

---
### XSS
<img width="316" height="729" alt="image" src="https://github.com/user-attachments/assets/d0810392-ea67-467c-b562-7bc0818e2863" />\
<img width="655" height="518" alt="image" src="https://github.com/user-attachments/assets/514aeaeb-4b0d-4cc1-af69-68a6db53bb4c" />


```
<script>alert(10)</script>
```

Since there is a field that accepts user input, a simple XSS payload was inserted into the message field.
