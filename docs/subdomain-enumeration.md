## Sirve para CTFs o "entornos controlados"

```
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -H Host:FUZZ.ejemplo.com -u http://ejemplo.com -fw 125
```

### Pro tip:
#### Para CTFS con sublist3r se puede cambiar el .htb // .thm por .com y es probable que encuentre verdaderos subdominios