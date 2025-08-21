# Local Payload Execution - DLL

- En este apunte se documentan las técnicas de ejecución de payloads locales en Windows utilizando DLLs.

### Crear una DLL

- Para crear una DLL en C podemos armar un proyecto en Visual Studio y utilizar el siguiente código de ejemplo:

Esta demostración utilizará un cuadro de mensaje que aparece cuando la DLL se carga exitosamente. Crear un cuadro de mensaje se puede hacer fácilmente con la API de Windows MessageBox. El fragmento de código a continuación ejecutará MsgBoxPayload cada vez que la DLL sea cargada en un proceso. Nota que los encabezados precompilados fueron removidos de la configuración C/C++ del proyecto como se muestra en el módulo introductorio de Biblioteca de Enlace Dinámico.


```c
#include <Windows.h>
#include <stdio.h>

VOID MsgBoxPayload() {
    MessageBoxA(NULL, "Hacking", "Wow !", MB_OK | MB_ICONINFORMATION);
}


BOOL APIENTRY DllMain (HMODULE hModule, DWORD dwReason, LPVOID lpReserved){

    switch (dwReason){
        case DLL_PROCESS_ATTACH: {
            MsgBoxPayload();
            break;
        };
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
        case DLL_PROCESS_DETACH:
            break;
    }

    return TRUE;
}
```

----


### Compilar la DLL

- Para compilar la DLL, podemos utilizar el comando `cl` de Visual Studio:

```powershell
cl /LD /Fe:payload.dll payload.c user32.lib
```


## Inyección local de la DLL

Recordemos que la **WinAPI** `LoadLibrary` se utiliza para cargar una DLL. Esta función toma la ruta de una DLL en disco y la carga en el espacio de direcciones del proceso que la llama, que en nuestro caso será el proceso actual. Cargar la DLL ejecutará su punto de entrada y, por lo tanto, ejecutará la función `MsgBoxPayload`, haciendo que aparezca el cuadro de mensaje. Aunque el concepto es simple, será útil en módulos posteriores para entender técnicas más complejas.

El código a continuación tomará el nombre de la DLL como argumento de línea de comandos, la cargará usando **LoadLibraryA**, y realizará algunas verificaciones de errores para asegurar que la DLL se cargó exitosamente.

```c
#include <Windows.h>
#include <stdio.h>


int main(int argc, char* argv[]) {

	if (argc < 2){
		printf("[!] Missing Argument; Dll Payload To Run \n");
		return -1;
	}

	printf("[i] Injecting \"%s\" To The Local Process Of Pid: %d \n", argv[1], GetCurrentProcessId());
	
	
	printf("[+] Loading Dll... ");
	if (LoadLibraryA(argv[1]) == NULL) {
		printf("[!] LoadLibraryA Failed With Error : %d \n", GetLastError());
		return -1;
	}
	printf("[+] DONE ! \n");

	
	printf("[#] Press <Enter> To Quit ... ");
	getchar();

	return 0;
}
``` 

----

# Local Payload Execution - Shellcode

## Windows API necesarios
- **[VirtualAlloc](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc)**: Asigna memoria en el proceso actual que será utilizada para almacenar el payload/shellcode.
- **[VirtualProtect](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect)**: Modifica los permisos de protección de la memoria asignada para hacerla ejecutable y poder ejecutar el payload.
- **[CreateThread](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread)**: Crea un nuevo hilo de ejecución que ejecutará el payload almacenado en la memoria asignada.

### Allocating memory

```c
LPVOID VirtualAlloc(
  [in, optional] LPVOID lpAddress,          // The starting address of the region to allocate (set to NULL)
  [in]           SIZE_T dwSize,             // The size of the region to allocate, in bytes
  [in]           DWORD  flAllocationType,   // The type of memory allocation
  [in]           DWORD  flProtect           // The memory protection for the region of pages to be allocated
);
```

El tipo de asignación de memoria se especifica como `MEM_RESERVE | MEM_COMMIT`, que reserva un rango de páginas en el espacio de direcciones virtuales del proceso que realiza la llamada y confirma la memoria física para esas páginas reservadas. Las flags combinadas se discuten por separado a continuación:

- **`MEM_RESERVE`**: Se utiliza para reservar un rango de páginas sin confirmar realmente la memoria física.
- **`MEM_COMMIT`**: Se utiliza para confirmar un rango de páginas en el espacio de direcciones virtuales del proceso.

El último parámetro de `VirtualAlloc` establece los permisos en la región de memoria. La forma más fácil sería establecer la protección de memoria a `PAGE_EXECUTE_READWRITE`, pero esto generalmente es un indicador de actividad maliciosa para muchas soluciones de seguridad. Por lo tanto, la protección de memoria se establece a `PAGE_READWRITE` ya que en este punto solo se requiere escribir el payload, pero no ejecutarlo. Finalmente, `VirtualAlloc` retornará la dirección base de la memoria asignada.



### Writing Payload to memory

A continuación, los bytes del payload desobfuscado se copian en la región de memoria recién asignada en `pShellcodeAddress` y luego se limpia `pDeobfuscatedPayload` sobrescribiéndolo con 0s. `pDeobfuscatedPayload` es la dirección base de un heap asignado por la función `UuidDeobfuscation` que retorna los bytes del shellcode en bruto. Ha sido sobrescrito con ceros ya que ya no se requiere y por lo tanto esto reducirá la posibilidad de que las soluciones de seguridad encuentren el payload en memoria.
