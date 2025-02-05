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
