# SSH Local Port Forwarding.
- Caso de uso:
    - Tenés una maquina victima con acceso SSH, tiene servicios locales como una base de datos MySQL la cual no se puede acceder desde el exterior, podés reenviar el acceso a ese puerto mediante **ssh tunneling**

`ssh -L 3306:127.0.0.1:3306 ubuntu@IP`
- Para chequear: `nmap -p3306 localhost`

- También se pueden reenviar varios puertos a la vez:
`ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64`

## Pivoting
- Pivoting es un poco más complejo, tenemos que usar una interface completa junto con un proxy

![](https://academy.hackthebox.com/storage/modules/158/22.png)

- Acá el servidor ssh empieza escuchando en el puerto 9050 de nuestra maquina atacante y va a ser redirigida.

## Dynamic Port forwarding SSH & Socks tunneling

`ssh -D 9050 user@host`

- El argumento `-D` peticiona al servidor SSH que se habilite el reenvío de puertos dinamirocs.
- Una vez que esté esto habilitado utilizamos una herramienta que routee los paquetes a ese puerto como proxychains con SOCKS4/5.

```
GNT@htb[/htb]$ tail -4 /etc/proxychains.conf
# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

- Luego para todos los comandos tenés que poner antes proxychains, ejemplo:
`proxychains nmap -v -sn 172.16.5.1-200`
- Esto es un ping sweep para ver todos los host de la red
