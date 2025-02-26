# Buffer Overflow
- Buffer overflow escausado cuando un programa grandes cantidades de datos que no caben en el buffer que se le asignó.
- Un buffer overflow puede causar que el programa crashee, corrompa datos o dañe estrucuturas del runtime del programa.
- Esto puede causar que se pueda sobreescribir la dirección de retorno con datos arbitrarios. Permitiendo a un at acante ejecutar comandos con el privilegio de ese programa.
- La mayor causa de los buffer overflows son los lenguajes de programación que no monitorean los limites de la memoria asignada a un buffer o stack.

## [Buffer Overflow en Linux](./bufferOverflowLinux.md)
## [Buffer Overflow en Windows](./bufferOverflowWindows.md)