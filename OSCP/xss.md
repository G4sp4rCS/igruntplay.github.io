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