# Windows Active directory

[**internal pentesting "all the things" notes**](https://swisskyrepo.github.io/InternalAllTheThings/)

### [Tools](./adTools.md)

### Attack Vectors & Post Compromise Attacks too
- [LLMNR POISONING](./llmnr.md)
    - This is a quick view
    - [Attack tool "Responder"](https://www.kali.org/tools/responder/)
- [SMB Relay Attacks](https://medium.com/@aniswersighni/active-directory-attacks-smb-relay-attacks-ea7d8cf9a8f8)
    - [This guide is a little bit better than the other one](https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/adversary-in-the-middle/smb-relay)
- [Pass the hash](./passTheHash.md)
- [Password attacks & Cracking](./passAttacks.md)
- [IPv6 AD Attack (mitm)](https://stridergearhead.medium.com/ipv6-attack-ad-attack-ea50476dccee)
- [Passback attacks](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack)
- [Attacking LSASS](./lsass.md)
- [DCsync Attack](./dcsync.md)
- [Group Policy Preferences Passwords](./gpp.md)
- others that i didn't take notes yet

### [More Post compromise attacks](./postCompromiseAttacks.md)



## Quick notes

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
- [ENUMERATION HERE](./enum.md)
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
- Trevor Spray tool
- [SeatBelt Tool](https://github.com/GhostPack/Seatbelt)


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
