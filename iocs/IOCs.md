# 🚨 Indicadores de Compromiso (IOCs)
## Incidente: 4geeks-server — Junio 2025

---

## Red / Network

| Tipo | Indicador | Puerto | Protocolo | Rol |
|---|---|---|---|---|
| IPv4 | `192.168.1.100` | 8080/tcp | HTTP | Host C2 — origen del dropper y destino de exfiltración |
| URL | `http://192.168.1.100/install.sh` | 8080 | HTTP | Descarga del dropper de primera etapa |
| URL | `http://192.168.1.100/payload.bin` | 8080 | HTTP | Descarga del binario de segunda etapa (LPE) |
| URL | `http://192.168.1.100:8080/upload` | 8080 | HTTP POST | Endpoint de recepción de datos exfiltrados |

### Patrón de tráfico
```
Origen  : 192.168.0.133 (víctima)
Destino : 192.168.1.100:8080
Método  : HTTP POST multipart/form-data
Payload : archivo secrets.tgz (compresión de /etc/passwd)
Frecuencia: cada 15 minutos (activado por cron)
```

---

## Filesystem

### Archivos maliciosos (ya eliminados)
| Ruta | Descripción | Estado |
|---|---|---|
| `/etc/cron.d/sys-maintenance` | Tarea cron maliciosa (`*/15 * * * * root /usr/local/bin/backup2.sh`) | **ELIMINADO** |
| `/usr/local/bin/backup2.sh` | Script de exfiltración (tar + curl POST) | **ELIMINADO** |
| `/tmp/secrets.tgz` | Archivo comprimido con /etc/passwd listo para exfiltrar | **ELIMINADO** |
| `/tmp/.temp/payload` | Binario de segunda etapa — LPE. No recuperado. | **ELIMINADO** (no recuperable) |

### Archivos de evidencia (preservados)
| Ruta | Descripción | Propiedad |
|---|---|---|
| `/home/reports/install.sh` | Dropper de primera etapa (preservado como evidencia) | reports:reports |
| `/home/reports/chat.txt` | Mensaje de phishing de "unknown@externalmail.com" | reports:reports |
| `/home/reports/backup.log` | Log de cobertura fabricado manualmente (typo en IP revela fabricación) | reports:reports |
| `/home/reports/.note` | Nota referenciando `/opt/.archive` — escrita como root post-escalada | **root:root** |
| `/opt/.archive/credentials.txt` | Credenciales `reports:reports123` en texto plano | root:root |

---

## Cuentas

| Cuenta | UID | GID | Shell | Origen | Uso real | Estado |
|---|---|---|---|---|---|---|
| `reports` | 1001 | 1001 | /bin/bash | /dev/tty1 (21-jun-2025 19:54) | 1 sesión (23-jun-2025 14:07) | **ELIMINADA** |
| `hacker` | 1002 | 1002 | /bin/bash | /dev/tty1 (23-jun-2025 15:02:47) | Ninguno | **ELIMINADA** |
| `www-data` | 33 | 33 | ~~`/bin/bash`~~ | Modificada por payload | N/A | **CORREGIDA** → `/usr/sbin/nologin` |

### Credencial comprometida
```
Usuario : reports
Password: reports123 (texto plano en /opt/.archive/credentials.txt)
Hash    : $6$fFYfdILCOsztfi3e$... (sha512crypt, activo al momento del hallazgo)
```

---

## Hashes SHA-256 de evidencia preservada

> Los siguientes hashes corresponden a los backups de evidencia almacenados en `/root/ir-evidence/` para cadena de custodia.

```
# Archivo                                          SHA-256
home-backups/reports-home_20260618.tar.gz         [ejecutar sha256sum para obtener]
home-backups/hacker-home_20260618.tar.gz          [ejecutar sha256sum para obtener]
credentials.txt.evidence                          [ejecutar sha256sum para obtener]
```

---

## Cronología de modificaciones de archivos del sistema

| Archivo | mtime comprometido | Significado |
|---|---|---|
| `/etc/passwd` | 2025-06-23 15:02:47 | Adición de cuenta `hacker` + shell de www-data |
| `/etc/shadow` | 2025-06-23 15:03:53 | Alta de contraseña para `hacker` (66s post-creación) |
| `/home/hacker/` | 2025-06-23 15:02:47 | Creación del directorio home de backdoor |
| `/home/reports/` | 2025-06-23 14:40:35 | Última modificación durante sesión del atacante |
| `/etc/cron.d/sys-maintenance` | 2025-06-23 ≈15:02 | Fecha anómala (vecinos: Feb-2020 / Mar-2023) |

---

## Descartados / Negativos

Los siguientes IOCs potenciales fueron investigados y descartados:

| IOC investigado | Resultado |
|---|---|
| Exfiltración de `/etc/shadow` | ❌ Fabricada — log de cobertura manual con typo de IP |
| Módulos de kernel maliciosos | ❌ lsmod limpio — solo módulos legítimos |
| Reglas de firewall modificadas | ❌ UFW/iptables/nftables sin cambios |
| Binarios del sistema troyanizados | ❌ dpkg -V sin diferencias |
| LD_PRELOAD backdoor | ❌ /etc/ld.so.preload vacío |
| Timers systemd maliciosos | ❌ Solo timers estándar de Ubuntu |
| Uso de la cuenta `hacker` | ❌ Sin entradas en lastlog, .bash_history vacío |
| Acceso web/FTP como vector de entrada | ❌ access.log 0 bytes, vsftpd.log vacío |

---

*Generado durante la investigación Live IR — 4geeks-server — Junio 2026*
