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





  
