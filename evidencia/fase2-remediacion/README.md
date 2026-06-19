# 📸 Evidencia — Fase 2: Remediación y Recuperación

Esta carpeta contiene las capturas que documentan cada acción de contención, erradicación y recuperación aplicada.

## Contenido

| Captura | Fecha | Acción |
|---|---|---|
| `01-eliminacion-artefactos.png` | 10-jun-2026 | rm de cron, script y secrets.tgz — verificación con ls |
| `02-preservacion-evidencia.png` | 18-jun-2026 | mkdir ir-evidence + tar backups de homes + sha256sum |
| `03-userdel-reports-hacker.png` | 18-jun-2026 | userdel -r reports y hacker — mensajes de mail spool |
| `04-verificacion-cuentas-eliminadas.png` | 18-jun-2026 | grep /etc/passwd y /etc/group — resultado vacío |
| `05-shred-credentials.png` | 18-jun-2026 | shred -u credentials.txt + rmdir /opt/.archive |
| `06-vsftpd-deshabilitado.png` | 18-jun-2026 | systemctl stop/disable vsftpd — confirmación systemd |

> Las acciones de remediación están documentadas en detalle en el Capítulo 7 del informe técnico.
