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

## Path Abuse
- Definición: PATH es una variable de entorno que contiene una lista de directorios a los que el sistema busca ejecutables.
    - Ejemplo: cuando hacemos `cat /etc/passwd` el sistema busca el ejecutable `cat` en `/bin/cat` en los directorios especificados en la variable PATH.
- Podemos chequear el contenido de la variable PATH con `echo $PATH`
- Crear un script o programa especificado en el PATH lo hace ejecutable en cualquier lugar del sistema.
- Ejemplo de abuso:
```bash
htb_student@NIX02:~$ touch ls
htb_student@NIX02:~$ echo 'echo "PATH ABUSE!!"' > ls
htb_student@NIX02:~$ chmod +x ls
htb_student@NIX02:~$ ls
PATH ABUSE!!
```

### Setuid permissions
- Definición: Setuid permissions permite a un proceso ejecutar comandos como el usuario que lo ejecuta. **Set User ID upon Execution (setuid)**
- El setuid bit aparece como una `s` en el lugar del bit de ejecución en el owner.
- `find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null`
### Setgid permissions
- Definición: Setgid permissions permite a un proceso ejecutar comandos como el grupo que lo ejecuta.
- `find / -uid 0 -perm -6000 -type f 2>/dev/null`