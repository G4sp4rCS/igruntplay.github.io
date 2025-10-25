# IDOR (Insecure Direct Object Reference)
- IDOR es una vulnerabilidad de seguridad que se produce cuando una aplicación web expone referencias directas a objetos internos.
- Por ejemplo, si una aplicación web utiliza un parámetro numérico para acceder a un recurso, un atacante podría modificar ese parámetro para acceder a recursos no autorizados.

## Identificando IDORs
- Para identificar IDORs, se pueden seguir los siguientes pasos:
    - Identificar los parámetros que se utilizan para acceder a recursos.
    - Modificar esos parámetros para acceder a recursos no autorizados.
    - Observar si la aplicación web permite el acceso a recursos no autorizados.
### Llamadas AJAX
- En las llamadas AJAX, se pueden identificar IDORs de la siguiente manera:
    - Inspeccionar las llamadas AJAX en la pestaña Network de las herramientas de desarrollo del navegador.
    - Identificar los parámetros que se utilizan en las llamadas AJAX.
    - Modificar esos parámetros para acceder a recursos no autorizados.
    - Observar si la aplicación web permite el acceso a recursos no autorizados.

## Enumeración masiva con IDORs
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
