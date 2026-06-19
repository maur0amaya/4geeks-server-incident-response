# 📸 Evidencia — Fase 1: Reconocimiento y Recolección

Esta carpeta contiene las capturas de pantalla de la investigación activa en vivo, organizadas por área de análisis.

## Contenido

| Captura | Fecha | Hallazgo |
|---|---|---|
| `01-netstat-conexion-c2.png` | 05-jun-2026 | Conexión SYN_SENT activa a 192.168.1.100:8080 (curl PID 2020) |
| `02-ssh-acceso-legitimo.png` | 10-jun-2026 | Sesión SSH del analista (sysadmin desde 192.168.0.147) |
| `03-netstat-sesion-analista.png` | 10-jun-2026 | Verificación post-escalada — solo conexión del analista visible |
| `04-ps-auxf-kernel-threads.png` | 10-jun-2026 | Árbol de procesos parte 1 — hilos del kernel |
| `05-ps-auxf-servicios.png` | 10-jun-2026 | Árbol de procesos parte 2 — servicios, vsftpd, sshd, sesión analista |
| `06-ps-auxf-apache-wazuh.png` | 10-jun-2026 | Árbol de procesos parte 3 — apache2, agente Wazuh, sysadmin en tty1 |
| `07-crontab-sistema.png` | 10-jun-2026 | /etc/crontab estándar, crontab -l vacío, /var/spool vacío |
| `08-cron-d-sys-maintenance.png` | 10-jun-2026 | /etc/cron.d/ — sys-maintenance con fecha anómala (23-jun-2025) |
| `09-tmp-secrets-tgz.png` | 10-jun-2026 | find en /tmp — hallazgo de secrets.tgz |
| `10-syslog-cron-ejecucion.png` | 10-jun-2026 | syslog — CRON[6088] y CRON[6125] ejecutando backup2.sh |
| `11-backup2sh-contenido.png` | 10-jun-2026 | Contenido de /usr/local/bin/backup2.sh (tar + curl POST) |
| `12-secrets-tgz-contenido.png` | 10-jun-2026 | tar -tvf secrets.tgz — solo /etc/passwd |
| `13-cron-grep-confirmacion.png` | 10-jun-2026 | grep de backup2.sh en cron confirma /etc/cron.d/sys-maintenance |
| `14-etc-passwd-shadow-keys.png` | 16-jun-2026 | Auditoría — sin UID 0 adicionales, sin passwords vacíos, sin authorized_keys |
| `15-suid-systemd-rclocal.png` | 16-jun-2026 | SUID estándar, sin unidades systemd modificadas, sin rc.local |
| `16-bash-history-root-sysadmin.png` | 16-jun-2026 | bash_history de root y sysadmin — solo comandos del analista |
| `17-cuentas-shell-interactiva.png` | 16-jun-2026 | grep /bin/bash en passwd — reports y hacker con shell interactiva |
| `18-etc-passwd-completo.png` | 16-jun-2026 | Listado completo de /etc/passwd con UID/shell |
| `19-grupo-sudo.png` | 16-jun-2026 | Solo sysadmin en grupo sudo |
| `20-shadow-hashes-activos.png` | 16-jun-2026 | Hashes $6$ activos para hacker y reports en /etc/shadow |
| `21-authorized-keys-vacios.png` | 16-jun-2026 | authorized_keys de hacker y reports vacíos |
| `22-home-contenido.png` | 16-jun-2026 | Contenido de /home/hacker y /home/reports con fechas |
| `23-bash-history-reports.png` | 16-jun-2026 | bash_history de reports — cadena de ataque completa |
| `24-lastlog-sessions.png` | 16-jun-2026 | lastlog y last — única sesión de reports, hacker sin sesiones |
| `25-stat-homes-chage.png` | 16-jun-2026 | stat de /home/hacker y /home/reports + chage de ambas cuentas |
| `26-auth-log-cronologia.png` | 16-jun-2026 | auth.log — cronología completa de creación de cuentas + journalctl |
| `27-install-sh.png` | 16-jun-2026 | Contenido de /home/reports/install.sh (dropper stage 1) |
| `28-credentials-txt.png` | 16-jun-2026 | /opt/.archive/credentials.txt → reports:reports123 |
| `29-chat-note-backuplog.png` | 16-jun-2026 | chat.txt (phishing) + .note + backup.log fabricado |
| `30-sudoers-verificacion.png` | 16-jun-2026 | id reports, grupos, /etc/sudoers y /etc/sudoers.d/ |
| `31-sudoers-d-readme.png` | 16-jun-2026 | Contenido README estándar de /etc/sudoers.d/ |

> Las capturas originales se encuentran en el informe técnico (`docs/Informe_IR_4geeks-server.docx`) con numeración y descripción detallada.
