# Windows Active directory

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

### Pot-Compromise AD enumeration
- [Bloudhound](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/bloodhound)
- Plumhound
- Ldapdomaindump
    - `sudo ldapdomaindump ldaps://$IP -u 'DOMAIN\username' -p $PASSWORD`
- PingCastle
- etc