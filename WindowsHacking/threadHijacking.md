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

## Creating The Target Thread

El requisito previo para realizar el hijacking de hilos es encontrar un hilo en ejecución para ser hijackeado. Es importante destacar que no es posible hijackear el hilo principal de un proceso local porque el hilo objetivo debe ser colocado primero en un estado suspendido. Esto es problemático cuando se intenta apuntar al hilo principal, ya que es el que ejecuta el código y no puede ser suspendido. Por lo tanto, no se debe apuntar al hilo principal cuando se realiza un hijacking de hilos local.

Este módulo demostrará cómo hijackear un hilo recién creado. Se llamará inicialmente a `CreateThread` para crear un hilo y establecer una función benigna como entrada del hilo. Posteriormente, el handle del hilo será utilizado para realizar los pasos necesarios para hijackear el hilo y ejecutar el payload en su lugar.

## Modifying The Thread's Context

El siguiente paso es obtener el contexto del hilo para modificarlo y hacer que apunte a un payload. Cuando el hilo reanude su ejecución, el payload será ejecutado.

Como se mencionó anteriormente, se utilizará `GetThreadContext` para obtener la estructura `CONTEXT` del hilo objetivo. Ciertos valores de esta estructura serán modificados para alterar el contexto actual del hilo utilizando `SetThreadContext`. Los valores que se modifican en la estructura son aquellos que determinan qué ejecutará el hilo a continuación. Estos valores son los registros `RIP` (para procesadores de 64 bits) o `EIP` (para procesadores de 32 bits).

Los registros `RIP` y `EIP`, también conocidos como registros de puntero de instrucción, indican la próxima instrucción a ejecutar. Estos se actualizan después de que se ejecuta cada instrucción.

## Setting ContextFlags

El segundo parámetro de `GetThreadContext`, `lpContext`, está marcado como un parámetro de entrada y salida (IN & OUT). La sección de *Remarks* en la documentación de Microsoft indica:

La función recupera un contexto selectivo basado en el valor del miembro `ContextFlags` de la estructura de contexto.

En esencia, Microsoft establece que el campo `CONTEXT.ContextFlags` debe establecerse con un valor antes de llamar a la función. `ContextFlags` se configura con la bandera `CONTEXT_CONTROL` para obtener el valor de los registros de control.

Por lo tanto, establecer `CONTEXT.ContextFlags` en `CONTEXT_CONTROL` es necesario para realizar el hijacking de hilos. Alternativamente, también se puede usar `CONTEXT_ALL` para llevar a cabo el hijacking de hilos.

## Thread Hijacking Function

`RunViaClassicThreadHijacking` es una función personalizada que realiza el hijacking de hilos. La función requiere 3 argumentos:

- `hThread`: Un handle a un hilo suspendido que será hijackeado.
- `pPayload`: Un puntero a la dirección base del payload.
- `sPayloadSize`: El tamaño del payload.

# Creating The Sacrificial Thread

Dado que `RunViaClassicThreadHijacking` requiere un handle a un hilo, la función principal debe proporcionarlo. Como se mencionó anteriormente, el hilo objetivo debe estar en un estado suspendido para que `RunViaClassicThreadHijacking` pueda hijackear el hilo con éxito.

Se utilizará la WinAPI `CreateThread` para crear un nuevo hilo. El nuevo hilo debe parecer lo más benigno posible para evitar su detección. Esto se puede lograr creando una función benigna que sea ejecutada por este hilo recién creado.

El siguiente paso es suspender el hilo recién creado para que `GetThreadContext` tenga éxito. Esto se puede realizar de dos maneras:

- Pasando la bandera `CREATE_SUSPENDED` en el parámetro `dwCreationFlags` de `CreateThread`. Esta bandera creará el hilo en un estado suspendido.
- Creando un hilo normal, pero suspendiéndolo posteriormente utilizando la WinAPI `SuspendThread`.

Se utilizará el primer método, ya que requiere menos llamadas a la WinAPI. Sin embargo, ambos métodos necesitarán que el hilo sea reanudado después de ejecutar `RunViaClassicThreadHijacking`. Esto se logrará utilizando la WinAPI `ResumeThread`, que solo requiere el handle del hilo suspendido.