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

### Cloud services
- [Buckets gray hat warfare](https://buckets.grayhatwarfare.com/)

### DNS
- [DNS Enumeration](./dnsEnum.md)