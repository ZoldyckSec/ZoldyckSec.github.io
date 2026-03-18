---
title: Conversor - HackTheBox WriteUp
description: This write-up for the retired machine Conversor was completed while the machine was still active on the HackTheBox platform. Publication was held until after its official retirement to comply with platform guidelines.
author: ZoldyckSec
date: 2026-01-05 00:00:00 -0800
categories: [HackTheBox]
tags: [Conversor, Linux, Path-transversal, SSTI, sqlite]
render_with_liquid: false
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





