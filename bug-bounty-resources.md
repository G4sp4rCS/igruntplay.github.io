# Bug bounty Resources

## Recon
- Assetfinder - [Find domains and subdomains potentially related to a given domain.](https://github.com/tomnomnom/assetfinder)  
- Subfinder - [subdomain discovery tool](https://github.com/projectdiscovery/subfinder)  
- Prips - [tool that can be used to print all of the IP address on a given range](https://gitlab.com/prips/prips)  
- JQ - [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)  
- Hakrevdns - [simple tool for performing reverse DNS lookups](https://github.com/hakluke/hakrevdns)  
- Massdns - [simple high-performance DNS stub resolver](https://github.com/blechschmidt/massdns)
- Shuffledns - [https://github.com/projectdiscovery/shuffledns](https://github.com/projectdiscovery/shuffledns)  
- Project Discovery Chaos - [https://chaos.projectdiscovery.io/#/](https://chaos.projectdiscovery.io/#/)  
- AMASS - [https://github.com/OWASP/Amass](https://github.com/OWASP/Amass) 
- LazyRecon - [https://github.com/nahamsec/lazyrecon](https://github.com/nahamsec/lazyrecon)
- HTTPX - [fast and multi-purpose HTTP toolkit that allows running multiple](https://github.com/projectdiscovery/httpx)

## Magic Checklist
[Bug bounty ToDo list by soyelmago](https://github.com/alanbriangh/Magic-CheckList-for-Web-Applications)  
[Workflows compilation](https://pentester.land/cheatsheets/2019/03/25/compilation-of-recon-workflows.html)

## Subdomain live checker
```
# toma como parametro el archivo con las urls
# ejemplo: python3 list_websites.py subfinder_starlink.txt
import sys
import subprocess

with open(sys.argv[1]) as fp:
    for line in fp:
        print("voy a probar:" + line.strip())
        subprocess.call(["curl", "-i", "--connect-timeout", "5", line.strip()])
        sys.stdout.flush()
```