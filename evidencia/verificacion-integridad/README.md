# 📸 Evidencia — Verificación Final de Integridad

Esta carpeta contiene las capturas de las verificaciones de integridad exhaustivas realizadas tras la remediación.

## Contenido

| Captura | Fecha | Verificación |
|---|---|---|
| `01-getcap-capacidades.png` | 17-jun-2026 | getcap -r / — solo capacidades estándar (ping, traceroute6) |
| `02-lsmod-parte1.png` | 17-jun-2026 | lsmod — módulos de kernel parte 1 (netfilter, SCSI, audio) |
| `03-lsmod-ld-preload.png` | 17-jun-2026 | lsmod parte 2 + cat /etc/ld.so.preload (vacío) |
| `04-dpkg-integridad.png` | 17-jun-2026 | dpkg -V — sin salida (ningún binario del sistema alterado) |
| `05-systemd-timers.png` | 17-jun-2026 | systemctl list-timers — 11 timers todos estándar de Ubuntu |
| `06-atq-profile-motd.png` | 17-jun-2026 | atq vacío, /etc/profile.d/ y /etc/update-motd.d/ limpios |
| `07-bashrc-hacker.png` | 17-jun-2026 | .bashrc y .profile de hacker — idénticos a /etc/skel |
| `08-bashrc-reports.png` | 17-jun-2026 | .bashrc y .profile de reports — sin modificaciones |
| `09-home-inventario.png` | 17-jun-2026 | ls /home/ — solo hacker, reports, sysadmin |
| `10-apache-logs.png` | 17-jun-2026 | Apache access.log (0 bytes), vsftpd.log vacío, /var/www limpio |
| `11-apache-directorio.png` | 17-jun-2026 | ls /var/log/apache2/ — access.log 0 bytes desde 21-jun-2025 |
| `12-hacker-sin-sesiones.png` | 17-jun-2026 | .bash_history vacío + lastlog — hacker sin sesiones |
| `13-ss-tunap.png` | 17-jun-2026 | ss -tunap — sin conexiones activas al C2 |
| `14-wazuh-sin-alertas-locales.png` | 17-jun-2026 | grep en /var/ossec/logs/ — sin directorio local (solo agente) |
| `15-iptables-1.png` | 18-jun-2026 | iptables -L -n -v — cadenas UFW estándar, INPUT/FORWARD DROP |
| `16-iptables-2.png` | 18-jun-2026 | Continuación — ufw-user-input solo puertos 22, 80 (21 ya cerrado) |
| `17-iptables-3.png` | 18-jun-2026 | Cadenas ufw-before-input/output/forward — sin reglas extra |
| `18-iptables-nat-nftables.png` | 18-jun-2026 | NAT con 0 paquetes/bytes + nft list ruleset vacío |
| `19-vsftpd-config.png` | 18-jun-2026 | vsftpd.conf — ssl_enable=NO + local_enable=YES (vector de riesgo) |

> Estas verificaciones están documentadas en el Capítulo 8 del informe técnico.
