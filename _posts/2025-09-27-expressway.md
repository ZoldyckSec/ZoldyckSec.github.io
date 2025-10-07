---
title: Expressway - HTB
description:  Está maquina fue resuelta mientras estaba activa. Publicado tras su retiro oficial según las normas de HackTheBox.
author: ZoldyckSec
date: 2025-09-28 00:00:00 +0800
categories: [HackTheBox, Expressway]
tags: [CTF, UDP, IKE, Easy, PSK,]
render_with_liquid: false
image:
  path: /assets/img/Expressway/Card-Express.png
---

## Description:

Expressway is an Easy difficulty Linux machine that focuses on enumerating non-conventional services and exploiting misconfigured IPSec VPN setups. The machine initially shows only a single TCP port (SSH), but the real attack surface lies in UDP services, specifically a vulnerable IKE/ISAKMP (IPSec VPN) implementation configured in aggressive mode.

## Main Themes:

- UDP service enumeration (port 500/udp - IKE/ISAKMP)
- PSK brute-forcing (Pre-Shared Keys)
- IPSec VPN attacks with aggressive mode enabled

## Skills Practiced:

- Advanced nmap scanning (UDP)
- Specialized tool usage (ike-scan, psk-crack)
- Offline IKE hash brute-forcing
- Sudo privilege escalation vulnerability exploitation

## Reconnaissance

#### Initial TCP Scan

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.87 -oG allPorts
```
- `-p-` Scans all ports(1-65535) instead of just the most common ones
- `-sS` Only shows open ports, ignoring closed or filtered ones.
- `--min-rate 5000` Sets a minimum of 5000 packets per seconds, speeding up the scan.
- `-vvv` Extreme verbose mode(shows more details during the scan).
- `-n` No DNS resolution(avoids wasting time resolving domain names).
- `Pn` Skips host discovery and assumes all hosts are active

#### Result

```markdown
# Nmap 7.95 scan initiated Fri Sep 26 20:22:45 2025 as: /usr/lib/nmap/nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -oG allPorts 10.10.11.87
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.11.87 ()    Status: Up
Host: 10.10.11.87 ()    Ports: 22/open/tcp//ssh///  Ignored State: closed (65534)
# Nmap done at Fri Sep 26 20:23:01 2025 -- 1 IP address (1 host up) scanned in 15.65 seconds
```
Only SSH `22` is visible on standar TCP Scan

##  Service and Version Enumeration

A service enumeration scan was performed against port 22 using Nmap's default scripts (-sC) and version detection (-sV) to gather detailed information about the SSH service."

```bash
nmap -p22 -sCV 10.10.11.87 -oN targeted
```
 - `-p22` Targeted port scanning focused exclusively on port `22(SSH)`
 - `-sC` Default NSE scripts execution for comprehensive service interrogation
 - `-sV` Version detection to indentify service/application versions and banners

#### Result 

```markdown
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The single open port (SSH) running a recent version presented limited immediate attack surface. Without valid credentials,this vector offered low exploitaion porbability, promping exploration of alternative approaches.

#### Initial UDP Scan

Complenting the port enumeration phase, a `UDP` scan was performed to uncover services exclusive to the UDP protocol that might represent addiotional attack vectors.

```bash
nmap -sU --top-ports 100 -T4 -v -n --reason -oN UDP 10.10.11.87
```
- `-sU` **UDP Scan**. Sends UDP probes to ports and waits for either a response or an ICMP "port unreachable" message. UDP lacks a handshake, which is why it's typically slower and less accurate than TCP.

- `--top-ports` Scans **the top 100 most common UDP ports** according to the Nmap database (instead of 1-65535). Good for quick reconnaissance, but may miss services on non-standard ports.

- `-T4` **Timing Template**: Aggressive/Fast. Reduces timeouts and waits less for responses → faster scan but higher risk of **false negatives** and more network noise (detectable by IDS/IPS). Often reasonable for UDP, but if a firewall delays responses, prefer `-T3` or `-T4` with retries.

- `-v` **Verbosity**: More detailed console output. Shows progress and useful information. (You can use `-vv`/`-vvv` for more detail, but `-v` is usually sufficient.)

- `-n` **No DNS resolution**. Nmap will not attempt to resolve IP → hostname (faster, less DNS traffic).

- `--reason` Shows **why** Nmap classified a port (e.g., "ICMP port unreachable" → closed; "no response" → open|filtered). Very useful for interpreting UDP results.





  
