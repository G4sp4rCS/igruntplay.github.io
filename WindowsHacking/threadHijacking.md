# Thread Hijacking - Local Thread Creation
Thread Execution Hijacking es una técnica que permite ejecutar un payload sin necesidad de crear un nuevo hilo. Esta técnica funciona suspendiendo el hilo y actualizando el registro que apunta a la siguiente instrucción en memoria para que apunte al inicio del payload. Cuando el hilo reanuda su ejecución, se ejecuta el payload.

## Contexto del hilo

Antes de explicar la técnica, se debe entender el contexto del hilo. Cada hilo tiene una prioridad de scheduling y mantiene un conjunto de estructuras que el sistema guarda en el contexto del hilo. El contexto del hilo incluye toda la información que el hilo necesita para reanudar la ejecución sin problemas, incluyendo el conjunto de registros de CPU del hilo y la pila.
`GetThreadContext` y `SetThreadContext` son dos WinAPIs que pueden usarse para obtener y establecer el contexto de un hilo, respectivamente.

- `GetThreadContext` puebla una estructura `CONTEXT` que contiene toda la información sobre el hilo.
- Mientras que `SetThreadContext` toma una estructura `CONTEXT` poblada y la establece en el hilo especificado.

## Thread Hijacking vs Thread Creation

La primera pregunta que debe abordarse es por qué secuestrar un hilo creado para ejecutar un payload en lugar de ejecutar el payload usando un hilo recién creado.
La principal diferencia es **la exposición del payload y el sigilo**. Crear un nuevo hilo para la ejecución del payload expondrá la dirección base del payload, y por lo tanto el contenido del payload, porque el punto de entrada de un nuevo hilo debe apuntar a la dirección base del payload en memoria. Este no es el caso con thread hijacking porque el punto de entrada del hilo estaría apuntando a una función normal del proceso y por lo tanto el hilo parecería benigno.