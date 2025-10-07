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
- `-p-` Scans all ports(2-65535) instead of just the most common ones
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





  
