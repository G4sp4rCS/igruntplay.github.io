# DNS Enumeration with dig

## Basic DNS Query
- `dig [domain] @<DNS_Server>`
    - `dig inlanefreight.htb @10.129.221.34`
## Zone transfer
- `dig axfr [domain] @<DNS_Server>`
## Reverse DNS Lookup
- `dig -x <IP_Address>`
## Enumerate subdomains
- `for sub in $(cat subdomains.txt); do dig $sub.inlanefreight.htb @10.129.221.34; done`
## Check DNS Server Version (if enabled)
- `dig CHAOS VERSION.BIND @<DNS_Server>`
