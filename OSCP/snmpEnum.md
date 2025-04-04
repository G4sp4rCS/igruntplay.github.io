# SNMP Enumeration

## Que es SNMP
- SNMP (Simple Network Management Protocol) es un protocolo de red utilizado para la gestión y supervisión de dispositivos en una red IP. Permite a los administradores de red recopilar información sobre el estado y el rendimiento de los dispositivos, así como realizar configuraciones remotas.
- SNMP se basa en un modelo cliente-servidor, donde los dispositivos de red actúan como agentes SNMP y los sistemas de gestión SNMP actúan como clientes. Los agentes SNMP recopilan información sobre el dispositivo y la envían al sistema de gestión SNMP cuando se solicita.

## Como enumerar SNMP
- SNMP puede ser utilizado para enumerar información sobre dispositivos de red, como routers, switches, servidores y otros dispositivos. La información que se puede obtener incluye:
  - Información del sistema: nombre del host, ubicación, contacto, etc.
  - Información de la interfaz: estadísticas de tráfico, estado de la interfaz, etc.
  - Información de la memoria y CPU: uso de memoria, carga de CPU, etc.
  - Información de la configuración: configuraciones de red, rutas, etc.

### Primeros pasos

- El protocolo SNMP es UDP, por lo tanto para que NMAP lo pueda escanear, se debe utilizar el flag `-sU` para escanear puertos UDP.
    - `nmap -sU -p 161 <IP>`: Escanea el puerto 161 (puerto SNMP) en la dirección IP especificada.
- Para enumerar SNMP, se puede utilizar la herramienta `snmpwalk`, que permite realizar consultas SNMP a un dispositivo y obtener información sobre él. La sintaxis básica de `snmpwalk` es la siguiente:
- En primer lugar abrir: `sudo mousepad /etc/snmp/snmp.conf` y descomentar la linea `mibdirs /usr/share/snmp/mibs:/usr/share/snmp/mibs/iana:/usr/share/snmp/mibs/ietf` dentro del kali
- Luego descargar sudo apt install snmp-mibs-downloader
- Ahora por ejemplo podemos hacer: `snmpwalk -v2c -c public 10.10.11.248 -m all`
- Para utilizar una versión multi threading podemos usar snmpbulkwalk, que es más rápido que snmpwalk. La sintaxis básica de `snmpbulkwalk` es la siguiente:
- `snmpbulkwalk -v2c -c public 10.10.11.248 -m all | tee snmp.txt`
    - Esto lo que hace es guardar toda la info obtenida en un txt porque los logs son muy extensos.


### Extras
### Comandos útiles para SNMP Enumeration

- `snmpwalk -v2c -c public 10.80.50.8`  
    Devuelve información sobre interfaces de red, nombres de grupos, información de hardware y programas en ejecución en la máquina.

- `snmpwalk -v2c -c public 10.80.50.8 hrSWInstalledName`  
    Muestra los programas en ejecución en la máquina.

- `snmpwalk -v2c -c public 10.80.50.8 hrMemorySize`  
    Obtiene el tamaño de la memoria de la máquina.

- `snmpwalk -v2c -c public 10.80.50.8 sysContact`  
    Recupera la información de contacto del sistema.

### Comandos sobre SNMP en Windows

- `nmap -sU -p 161 --script snmp-win32-services 10.80.50.8`  
    Enumera los servicios de Windows a través de SNMP.

- `nmap -sU -p 161 --script snmp-win32-users 10.80.50.8`  
    Enumera los usuarios en la máquina objetivo.

- `nmap -sU -p 161 --script smb-brute 10.80.50.8`  
    Realiza fuerza bruta sobre la cadena de comunidad SNMP.

    ### Fuerza bruta de cadenas de comunidad SNMP
    - `onesixtyone -c /home/liodeus/wordlist/SecLists/Discovery/SNMP/common-snmp-community-strings-onesixtyone.txt <IP>`
    - `snmpbulkwalk -c <COMMUNITY_STRING> -v<VERSION> <IP>`
    - `snmp-check <IP>`

    #### Valores MIB

    | **MIB Value**            | **Parámetro de Windows**      |
    |---------------------------|-------------------------------|
    | 1.3.6.1.2.1.25.1.6.0     | Procesos del Sistema          |
    | 1.3.6.1.2.1.25.4.2.1.2   | Programas en Ejecución        |
    | 1.3.6.1.2.1.25.4.2.1.4   | Rutas de Procesos             |
    | 1.3.6.1.2.1.25.2.3.1.4   | Unidades de Almacenamiento    |
    | 1.3.6.1.2.1.25.6.3.1.2   | Nombre del Software           |
    | 1.3.6.1.4.1.77.1.2.25    | Cuentas de Usuario            |
    | 1.3.6.1.2.1.6.13.1.3     | Puertos TCP Locales           |

    #### Fuerza Bruta de Cadenas de Comunidad

    ```bash
    # Crear archivo con cadenas de comunidad comunes
    echo public > community
    echo private >> community
    echo manager >> community

    # Generar lista de IPs
    for ip in $(seq 1 254); do echo 10.11.1.$ip; done > snmp-ips

    # Ejecutar fuerza bruta con onesixtyone
    onesixtyone -c community -i snmp-ips
    ```

    #### Enumeración del Árbol MIB Completo

    ```bash
    # Enumerar todo el árbol MIB
    snmpwalk -c public -v1 $ip
    ```

    #### Enumeración de un Valor MIB Específico

    ```bash
    # Enumerar un valor MIB específico
    snmpwalk -c public -v1 $ip $MIB_Value
    ```

    #### Comprobación SNMP

    ```bash
    # Utilizar snmp-check para obtener información detallada
    snmp-check $ip
    ```
