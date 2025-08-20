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

