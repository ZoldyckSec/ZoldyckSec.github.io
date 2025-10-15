---
title: PENETRATION TESTING REPORT - EXPRESSWAY MACHINE
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

Date: Sep 27, 2025
Machine: Expressway (Linux)
Difficulty: Easy
Prepared by: Allan Espinoza(Aka.ZoldyckSec)

## EXECUTIVE SUMMARY

A security assessment was conducted against the Expressway machine, identifying critical vulnerabilities in the IPSec VPN service implementation. Complete system compromise was achieved through exploitation of an IKE/ISAKMP service configured in aggressive mode, followed by privilege escalation leveraging misconfigured sudo permissions on multiple executable scripts.

#### Key Findings

- `Critical` Service in agressive mode allowing PSK brute-forcing
- `High` Weak Pre-Shared Key susceptible to dictionary attacks
- `Critical`  Multiple scripts with sudo permissions allowing privilege escalation

## 1. INTRODUCTION

#### 1.1 Scope and Objectives
The assessment focused on the Expressway machine with the objective of identifying attack vectors through non-conventional services and vulnerable IPSec VPN configurations.

#### 1.2 Methodology
A testing methodology centered on UDP enumeration and IPSec service exploitation was employed.

- Exhaustive UDP service enumeration
- IKE/ISAKMP service analysis on UDP port 500
- Offline Pre-Shared Key (PSK) brute-forcing
- Exploitation of vulnerable sudo configuration on multiple scripts

## 2. TECHNICAL METHODOLOGY

#### 2.1 Reconnaissance Phase
- `Tools` nmap, ike-scan
- `Techniques` Exhaustive UDP scanning, IKE service fingerprinting

#### 2.2 Vulnerability Analysis Phase
- `Tools` ike-scan, psk-crack, custom wordlists
- `Techniques` Aggressive mode detection, IKE hash extraction

#### 2.3 Exploitation Phase
- `Techniques` Offline PSK brute-forcing, sudo privilege abuse through multiple scripts


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





  
