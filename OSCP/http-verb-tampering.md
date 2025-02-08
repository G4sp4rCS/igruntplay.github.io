# HTTP verb tampering

- hay 9 verbos HTTP, los más comunes son:
    - GET
    - POST
    - PUT
    - DELETE
    - PATCH
    - HEAD
    - OPTIONS
    - TRACE
    - CONNECT
- Entonces si una aplicación web no valida los verbos HTTP, se puede hacer un ataque de HTTP verb tampering.
- Un ataque de HTTP verb tampering es un ataque que se basa en la manipulación de los verbos HTTP para realizar acciones no autorizadas.
- Por ejemplo, si una aplicación web permite el uso del verbo DELETE, un atacante podría enviar una solicitud DELETE a un recurso que no debería ser eliminado.
- Para prevenir este tipo de ataques, se deben restringir los verbos HTTP permitidos en la aplicación web.
- O con PUT se puede subir un archivo a un servidor.
- O con TRACE se puede hacer un ataque de Cross-Site Tracing.
- O con CONNECT se puede hacer un ataque de HTTP tunneling, etc etc.
