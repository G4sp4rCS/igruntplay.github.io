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