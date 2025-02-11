# Linux Privilege Escalation

## Enumeration
- Generalmente vamos a empezar utilizando LinEnum, Linpeas, exploitSuggester, linux-exploit-suggester, etc.
- Hay varios puntos importantes a enumerar:
    - Version del OS
    - Version del Kernel
    - Servicios ejecutandose
- También podemos revisar `sudo -l` para ver que comandos podemos ejecutar con sudo.
- Podemos revisar los cronjobs `ls -la /etc/cron.daily/` y `ls -la /etc/cron.hourly/`