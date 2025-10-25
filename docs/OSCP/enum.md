# General Enumeration stuff

## Overview
![Overview](./enum-method3.png)

## OSINT

| Resource                       | Examples                                                                                                                                            |
|--------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| ASN / IP registrars            | IANA, ARIN for searching the Americas, RIPE for searching in Europe, BGP Toolkit                                                                    |
| Domain Registrars & DNS        | Domaintools, PTRArchive, ICANN, manual DNS record requests against the domain in question or against well-known DNS servers, such as 8.8.8.8        |
| Social Media                   | Searching LinkedIn, Twitter, Facebook, your region's major social media sites, news articles, and any relevant info you can find about the organization |
| Public-Facing Company Websites | Often, the public website for a corporation will have relevant info embedded. News articles, embedded documents, and the "About Us" and "Contact Us" pages can also be gold mines |
| Cloud & Dev Storage Spaces     | GitHub, AWS S3 buckets & Azure Blob storage containers, Google searches using "Dorks"                                                               |
| Breach Data Sources            | HaveIBeenPwned to determine if any corporate email accounts appear in public breach data, Dehashed to search for corporate emails with cleartext passwords or hashes we can try to crack offline. We can then try these passwords against any exposed login portals (Citrix, RDS, OWA, 0365, VPN, VMware Horizon, custom applications, etc.) that may use AD authentication |

## AD Enumeration
- [Kerbrute](https://github.com/ropnop/kerbrute)
    - `root@kali:~# ./kerbrute_linux_amd64 userenum -d lab.ropnop.com usernames.txt`
    - `kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users`
    - `kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt`
    - `kerbrute passwordspray -d inlanefreight.local --dc 172.16.7.3 users2.txt Welcome1`
- [BloodHound](./bloodhound.md)
- [AD ACL Enumeration](./aclAdEnum.md)
- [CA & CN Enumeration](./caEnum.md)
- [AD User Enumeration with lookupsid.py](./lookupsid.md)
- [Enumerating AD DNS RECORDS TOOL](https://github.com/dirkjanm/adidnsdump)
    - `adidnsdump -u DOMAIN\\user ldap://DC-IP`
        - flag `-r` resuelve registros desconocidos
- [LDAP Enumeration](./ldapEnum.md)


#### AD: Get installed programs via powershell & registry keys

```powershell
PS C:\htb> $INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize
```

## [SNMP enumeration](./snmpEnum.md)

## Detailed User Enumeration
- [LinkedIn to username](https://github.com/initstring/linkedin2username)
- Any script that combines the firstname and lastname
    - firstname.lastname
    - firstnamelastname
    - lastname.firstname
    - lastnamefirstname
    - lastname
    - firstname
    - initialFirstName.lastname
    - etc
    - [Script One](https://github.com/yuyudhn/osintname)
    - [Script two](https://github.com/haicenhacks/username-generator)


### Cloud services
- [Buckets gray hat warfare](https://buckets.grayhatwarfare.com/)

### DNS
- [DNS Enumeration](./dnsEnum.md)


#### vhost enum
- `ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://domain:port/ -H 'Host: FUZZ.domain' -fs 986`

#### http POST fuzzing
- `ffuf -w /ust/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://domain:PORT/something.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx`