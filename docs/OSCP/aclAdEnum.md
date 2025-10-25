# AD ACL Enum 

## Introducción
PowerView es una herramienta de PowerShell para obtener conocimiento situacional de la red en dominios de Windows. Contiene un conjunto de reemplazos en PowerShell para varios comandos de Windows "net *", que utilizan hooks de AD de PowerShell y funciones subyacentes de la API de Win32 para realizar funcionalidades útiles de dominio de Windows.

## Comandos de PowerView

### Find-InterestingDomainAcl
Encuentra ACLs de objetos en el dominio actual (o especificado) con derechos de modificación establecidos en objetos no integrados.

- `Import-Module .\PowerView.ps1`
- `Find-InterestingDomainAcl`

#### Ejemplo:
- Encuentra ACLs interesantes en el dominio actual.
    ```powershell
    Find-InterestingDomainAcl
    ```

### Targeted Enum
- `$sid = Convert-NameToSid USER`
- `Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}`

#### Mapeo de Valor GUID
- Example:
    - `$guid= "00299570-246d-11d0-a768-00aa006e0529"`
    - `Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl`
    - `-ResolverGUID` flag
        - `Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}`

### Creación de lista de usuarios del dominio AD
- `Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt`

#### Chequeo de cada usuario
- `foreach($line in [System.IO.File]::ReadLines("C:\Users\SOME-USER\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'DOMAIN\\USER'}}`

#### Ejemplo:
- Enumerando derechos del usuario "damundsen"
    ```powershell
    $sid2 = Convert-NameToSid damundsen
    Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose
    ```

    ```powershell
    AceType               : AccessAllowed
    ObjectDN              : CN=Help Desk Level 1,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
    ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
    OpaqueLength          : 0
    ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-4022
    InheritanceFlags      : ContainerInherit
    BinaryLength          : 36
    IsInherited           : False
    IsCallback            : False
    PropagationFlags      : None
    SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1176
    AccessMask            : 131132
    AuditFlags            : None
    AceFlags              : ContainerInherit
    AceQualifier          : AccessAllowed
    ```

    - Este usuario tiene **GenericWrite** sobre el grupo **Help Desk Level 1**, esto significa que podemos agregar a cualquier usuario a este grupo con este usuario y también que si podemos agregar un usuario a este grupo va a tomar los privilegios de **GenericWrite**.
    - Podemos chequear si este grupo está "anidado" a otro grupo.
    - `Get-DomainGroup -Identity "Help Desk Level 1" | select memberof`
    ```powershell
    memberof                                                                      
    --------                                                                      
    CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
    ```

## Bloodhound
![](https://academy.hackthebox.com/storage/modules/143/wley_damundsen.png)

- También se puede hacer todo esto mucho más fácil con Bloodhound en vez de con Powershell.

## Funciones adicionales de PowerView

### Export-PowerViewCSV
Exporta objetos a un archivo CSV de manera segura para hilos.
- `Get-DomainUser | Export-PowerViewCSV -Path "users.csv"`

### Get-DomainSPNTicket
Solicita un ticket de servicio Kerberos para un SPN especificado.
- `Get-DomainSPNTicket -SPN "HTTP/web.testlab.local"`
- `Get-DomainUser -SPN | Get-DomainSPNTicket -OutputFormat Hashcat`

### Get-DomainTrust
Retorna todas las relaciones de confianza de dominio para el dominio actual o un dominio especificado.
- `Get-DomainTrust`

### Get-ForestTrust
Retorna todas las relaciones de confianza de bosque para el bosque actual o un bosque especificado.
- `Get-ForestTrust`

### Get-PathAcl
Enumera las ACL para una ruta de archivo especificada.
- `Get-PathAcl "\\SERVER\Share"`

### Find-DomainUserLocation
Encuentra máquinas de dominio donde usuarios específicos están conectados.
- `Find-DomainUserLocation -UserName "jdoe"`

### Find-DomainShare
Encuentra comparticiones accesibles en máquinas de dominio.
- `Find-DomainShare`

### Get-DomainGroupMember
Retorna los miembros de un grupo de dominio específico.
- `Get-DomainGroupMember -Identity "Domain Admins"`