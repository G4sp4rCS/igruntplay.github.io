### En una maquina que pwnee que tenía la vuln shellshock, fue clave la enumeración para poder explotar la misma
```
❯ gobuster dir -u http://10.10.10.56/cgi-bin -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 400 -x cgi,php,sh --add-slash
```

#### el -x sh me dió la enumeración correcta