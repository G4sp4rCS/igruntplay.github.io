# Buffer Overflow en Linux

### [Apunte de Arquitectura y Organización del computador 2](https://apuntes.grunt.ar/24hw1vPASvGpoM-rouQTcg)
#### Entender la pila 
- ![](https://www.cs.virginia.edu/~evans/cs216/guides/stack-convention.png)
- ![](https://cdn.acunetix.com/wp_content/uploads/2019/06/buffer-overflow.png)

## Programa vulnerable en C
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int bowfunc(char *string) {

	char buffer[1024];
	strcpy(buffer, string);
	return 1;
}

int main(int argc, char *argv[]) {

	bowfunc(argv[1]);
	printf("Done.\n");
	return 1;
}
```

- Acá la función bowfunc toma de parametro un puntero a un string y lo copia a un buffer de 1024 bytes.
- Esto es vulnerable pues no se chequea si el tamaño del string es mayor a 1024 bytes. Permitiendo que se sobreescriba la dirección de retorno de la función bowfunc.
- De todas formas las distribuciones GNU/Linux modernas tienen protecciones contra este tipo de ataques.
    - Conocida como Address Space Layout Randomization (ASLR) que randomiza la dirección de memoria de las funciones y variables.
    - De todas formas como root se puede desactivar.

```bash
student@nix-bow:~$ sudo su
root@nix-bow:/home/student# echo 0 > /proc/sys/kernel/randomize_va_space
root@nix-bow:/home/student# cat /proc/sys/kernel/randomize_va_space

0
```

#### Compilar a 32 bits
```bash
student@nix-bow:~$ sudo apt install gcc-multilib
student@nix-bow:~$ gcc bow.c -o bow32 -fno-stack-protector -z execstack -m32
student@nix-bow:~$ file bow32 | tr "," "\n"
```

## Funciones de C a tener en cuenta que pueden ser vulnerables
- strcpy
- gets
- sprintf
- strcat
- scanf
- Estas funciones no protegen la memoria independientemente. Por lo tanto si se les pasa un string mayor al buffer que se les asignó, se puede sobreescribir la dirección de retorno de la función.

## GNU Debugger (GDB)
- `set disassembly-flavor intel`
- `disassemble main`
- `echo 'set disassembly-flavor intel' > ~/.gdbinit`
- GDB Dashboard: https://github.com/cyrus-and/gdb-dashboard
- Object dump con intel: `objdump -d -M intel bow32`