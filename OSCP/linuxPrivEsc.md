# Linux Privilege Escalation

## Enumeration
- Generalmente vamos a empezar utilizando LinEnum, Linpeas, exploitSuggester, linux-exploit-suggester, etc.
- Hay varios puntos importantes a enumerar:
    - Version del OS
    - Version del Kernel
    - Servicios ejecutandose
- También podemos revisar `sudo -l` para ver que comandos podemos ejecutar con sudo.
- Podemos revisar los cronjobs `ls -la /etc/cron.daily/` y `ls -la /etc/cron.hourly/`
- Encontrar directorios con acceso a escritura: `find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null`
- Encontrar archivos con acceso a escritura: `find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null`
- Encontrar scripts: `find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"`
- Corriendo servicios como root: `ps aux | grep root`