# Remote/Reverse Port Forwarding with SSH

- SSH puede escuchar en nuestro host local y reenviar un servicio en el host remoto a nuestro puerto, y el reenvío de puertos dinámico, donde podemos enviar paquetes a una red remota a través de un host pivote. Pero a veces, puede que queramos reenviar un servicio local al puerto remoto también. Consideremos el escenario en el que podemos RDP en el host de Windows Windows A. Como se puede ver en la imagen de abajo, en nuestro caso anterior, podríamos pivotar en el host de Windows a través del servidor Ubuntu.
![](https://academy.hackthebox.com/storage/modules/158/33.png)


- meterpreter (maquina atacante) <=> maquina pivot => windows A
- En nuestra maquina atacante: `ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN`

![](https://academy.hackthebox.com/storage/modules/158/44.png)