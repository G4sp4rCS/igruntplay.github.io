# Run As C#
- RunAsCS es una herramienta de post-explotación que permite ejecutar un programa como un usuario diferente en un sistema Windows. Esto es útil para obtener acceso a recursos restringidos o para ejecutar tareas con privilegios elevados.
- Si tenemos credenciales de un usuario dentro de una maquina pero no podemos acceder a ella, podemos usar RunAsCS para ejecutar un programa como ese usuario. Esto puede ser útil para obtener acceso a recursos restringidos o para ejecutar tareas con privilegios elevados.
- [Repo](https://github.com/antonioCoco/RunasCs)
- [Como utilizar](https://arttoolkit.github.io/wadcoms/RunasCs/)
    - `.\RunasCS.exe USER PASSWORD -l 2 "c:\Users\Public\reverse.exe"`