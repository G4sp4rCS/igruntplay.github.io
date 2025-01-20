# AD ACL Enum 

## Find-InterestingDomainAcl
- `Import-Module .\PowerView.ps1`
- `Find-InterestingDomainAcl`

### Targeted Enum
- `$sid = Convert-NameToSid USER`
- `Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}`

### Mapping GUID Value
- Example:
    - `$guid= "00299570-246d-11d0-a768-00aa006e0529"`
    - `Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl`
    - `-ResolverGUID` flag
        - `Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}`


### Creating list of AD Domain users
`Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt`

#### Check each user
`foreach($line in [System.IO.File]::ReadLines("C:\Users\SOME-USER\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'DOMAIN\\USER'}}`

#### Example:
- Enumerating rights of user "damundsen"

```powershell
PS C:\htb> $sid2 = Convert-NameToSid damundsen
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

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