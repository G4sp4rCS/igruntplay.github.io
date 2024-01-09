## Sirve para CTFs o "entornos controlados"

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -H Host:FUZZ.ejemplo.com -u http://ejemplo.com -fw 125
```