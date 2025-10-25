# DNS spoofing or Local DNS Cache Poisoning

### Desde una red local un atacante podría envenenar el servidor DNS local haciendo un MITM con Ettercap o Bettercap

- Para esto hay que editar `/etc/ettercap/etter.dns`
- Mappear el nombre de dominio que queremos spoofear

Ejemplo de HTB academy
```
GNT@htb[/htb]# cat /etc/ettercap/etter.dns

inlanefreight.com      A   192.168.225.110
*.inlanefreight.com    A   192.168.225.110
```

- Inicias y le das a escanear
- La IP de la victima va a target1 y el gateway a target2
- Plugins > Manage Plugins y dns_spoof activado.
    - Esto manda a la maquina objetivo la resolución DNS falsa.

[Video corto y explicativo](https://youtu.be/ermP_fmasdc)