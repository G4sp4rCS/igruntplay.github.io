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