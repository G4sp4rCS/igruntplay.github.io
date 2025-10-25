# Cross-Site Scripting XSS

- Son un ataque de tipo inyección de código a través de la web que ejecuta javascript.
- Basicamente es inyección de código javascript en la web.

## Tipos de XSS

### Reflejado
- Ocurre cuando los datos proporcionados del usuario se reflejan inmediatamente en la respuesta del servidor sin una validación decente.
- Un atacante podría enviar en unlace malicioso a un usuario y cuando este hace click se ejecuta en el navegador y le puede robar las cookies por ejemplo.
- Impacto:
    - Robo de cookies/sesión (account take over).
    - Filtración de datos sensibles.
    - Redireccionamiento a sitios maliciosos.

### Persistente
- En este caso el código malicioso queda fijo en el servidor y se ejecuta cada vez que un usuario accede a la página afectada.
- Un atacante inserta un script malicioso dentro de un comentario.
- Impacto:
    - Robo de información sensible.
    - Secuestro de sesiones.
    - Modificación del sitio web.

### DOM XSS
- Este tipo de XSS ocurre cuando la vulnerabilidad del código JS se encuentra unicamente del lado del cliente.
- El atacante podría manipular el DOM para ejecutar codigo malicioso.
- Impacto:
    - Manipulación del contenido de la página del lado del cliente.
    - Ejecución de acciones no autorizadas por el navegador (se podría utilizar BeeF como PoC).
    - Distribución de malware, kinda botnet, etc.

## Automatización
- [XSS STRIKE](https://github.com/s0md3v/XSStrike)
- [Payloads + fuzzing](./../xss.md)

## Un payload interesante
![](https://pbs.twimg.com/media/GkNmgp8bUAEKpBc?format=jpg&name=4096x4096)
- `((_)=>m=>_[m]|| (confirm(m),_[m]=1))({})`
- este payload está basado en una **función autoejecutable (IIFE)** y una **técnica de memoización** (ver programación dinamica) para evitar ejecuciones repetidas.

## **El Payload**
```js
((_)=>m=>_[m]|| (confirm(m), _[m]=1))({})
```

## **Explicación paso a paso**

### 1. **Función autoejecutable (IIFE)**
El código comienza con una **Immediately Invoked Function Expression (IIFE)**, es decir, una función que se ejecuta inmediatamente:
```js
((_) => m => _[m] || (confirm(m), _[m] = 1))({})
```
- La función externa `(_) => ...` recibe un objeto `{}` como argumento.
- Este objeto se usa como almacenamiento temporal para guardar valores.

### 2. **Función interna con `m` como parámetro**
```js
m => _[m] || (confirm(m), _[m] = 1)
```
- Cuando se llama con un valor `m`, la función intenta acceder a `_ [m]`.
- Si `_ [m]` es `undefined`, se ejecuta `confirm(m)` y luego se asigna `_ [m] = 1`.
- Si `_ [m]` ya existe, la función simplemente devuelve su valor, evitando que `confirm(m)` se ejecute nuevamente.

### 3. **Uso del operador OR (`||`) y la coma (` , `)**
```js
_[m] || (confirm(m), _[m] = 1)
```
- **`||` (Operador OR)**: Si `_ [m]` es `undefined`, evalúa la segunda parte `(confirm(m), _[m] = 1)`.
- **Operador `,` (coma)**: Permite ejecutar `confirm(m)` antes de asignar `_ [m] = 1`.

### 4. **Ejecución del código**
El resultado es que `confirm(m)` solo se ejecutará la primera vez que se pase un valor `m`. Para valores repetidos, el código devolverá `_ [m]` sin ejecutar `confirm(m)` nuevamente.

## **Uso en ataques XSS**
Este payload puede inyectarse en sitios vulnerables para ejecutar código arbitrario en el navegador de la víctima.

**Ejemplo de inyección XSS:**
```html
<input onfocus="((_)=>m=>_[m]|| (confirm(m), _[m]=1))({})('XSS')" autofocus>
```
- Cuando el usuario hace foco en el `input`, se ejecuta `confirm('XSS')`.
