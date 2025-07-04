# Técnicas Prácticas de Enumeración en AWS

Este apunte está diseñado para pentesters y red teamers que buscan identificar recursos en AWS de manera efectiva y ética. Incluye técnicas externas e internas, ejemplos prácticos y comandos específicos para maximizar su utilidad.

---

## 1. Enumeración Externa: Sin Credenciales

### 1.1. Descubrimiento de Subdominios
- **Objetivo**: Encontrar subdominios asociados al dominio objetivo (ej., `example.com`).
- **Comando**:
  ```bash
  dnsenum example.com --threads 50
  ```
- **Resultado Esperado**: Lista de subdominios como `app.example.com` o `dev.example.com`.
- **Valor**: Revela servicios expuestos (EC2, S3, etc.) vinculados a subdominios.

#### Ejemplo de offsec:

```bash
❯ sudo nano /etc/resolv.conf
❯ sudo systemctl restart NetworkManager
❯ host www.offseclab.io 3.208.18.82
Using domain server:
Name: 3.208.18.82
Address: 3.208.18.82#53
Aliases: 

www.offseclab.io has address 54.90.147.109

❯ host -t ns offseclab.io
offseclab.io name server ns-1536.awsdns-00.co.uk.
offseclab.io name server ns-1024.awsdns-00.org.
offseclab.io name server ns-512.awsdns-00.net.
offseclab.io name server ns-0.awsdns-00.com.

❯ dnsenum offseclab.io --threads 100
dnsenum VERSION:1.3.1

-----   offseclab.io   -----


Host's addresses:
__________________

offseclab.io.                            60       IN    A        54.90.147.109


Name Servers:
______________



Mail (MX) Servers:
___________________



Trying Zone Transfers and getting Bind Versions:
_________________________________________________

unresolvable name: ns-0.awsdns-00.com at /usr/bin/dnsenum line 892 thread 5.

Trying Zone Transfer for offseclab.io on ns-0.awsdns-00.com ... 
AXFR record query failed: no nameservers
unresolvable name: ns-1024.awsdns-00.org at /usr/bin/dnsenum line 892 thread 6.

Trying Zone Transfer for offseclab.io on ns-1024.awsdns-00.org ... 
AXFR record query failed: no nameservers
unresolvable name: ns-1536.awsdns-00.co.uk at /usr/bin/dnsenum line 892 thread 7.

Trying Zone Transfer for offseclab.io on ns-1536.awsdns-00.co.uk ... 
AXFR record query failed: no nameservers
unresolvable name: ns-512.awsdns-00.net at /usr/bin/dnsenum line 892 thread 8.

Trying Zone Transfer for offseclab.io on ns-512.awsdns-00.net ... 
AXFR record query failed: no nameservers


Brute forcing with /usr/share/dnsenum/dns.txt:
_______________________________________________


mail.offseclab.io.                       60       IN    A        54.90.147.109

www.offseclab.io.                        60       IN    A        54.90.147.109

``` 

-----




### 1.2. Identificación de Buckets S3 Públicos
- **Técnica**: Buscar buckets accesibles probando nombres predecibles.
- **Comando Manual**:
  ```bash
  curl http://example-lab-assets-public.s3.amazonaws.com/
  ```
  - Respuesta XML = bucket público.
  - "Access Denied" = bucket existe pero está protegido.
  - "NoSuchBucket" = bucket no existe.
- **Automatización con `cloud_enum`**:
  ```bash
  cloud_enum -k example-lab -qs --disable-azure --disable-gcp
  ```
- **Valor**: Encuentra almacenamiento público con datos sensibles (ej., backups, configs).

#### Ejemplo de offsec

```bash

kali@kali:~$ cloud_enum -k offseclab-assets-public-axevtewi --quickscan --disable-azure --disable-gcp

...

Keywords:    offseclab-assets-public-axevtewi
Mutations:   NONE! (Using quickscan)
Brute-list:  /usr/lib/cloud-enum/enum_tools/fuzz.txt

[+] Mutated results: 1 items

++++++++++++++++++++++++++
      amazon checks
++++++++++++++++++++++++++

[+] Checking for S3 buckets
  OPEN S3 BUCKET: http://offseclab-assets-public-axevtewi.s3.amazonaws.com/
      FILES:
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/offseclab-assets-public-axevtewi
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/amethyst-expanded.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/amethyst.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/logo.svg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/pic02.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/pic05.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/pic13.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/ruby-expanded.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/ruby.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/saphire-expanded.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/saphire.jpg
                            
                            
 Elapsed time: 00:00:00

[+] Checking for AWS Apps
[*] Brute-forcing a list of 1 possible DNS names
                            
 Elapsed time: 00:00:00


[+] All done, happy hacking!


```

----

Hay muchas formas en las que podríamos lograr esto. Usaremos un script de Bash para ejecutar un bucle en una sola línea que itera sobre algunas palabras clave y muestra la palabra clave, insertando un prefijo (offseclab-assets) y un sufijo (-axevtewi) alrededor de ella. Finalmente, usaremos el comando tee para mostrar el resultado en la consola, así como en el archivo /tmp/keyfile.txt. Como resultado, tendremos el archivo clave con los nombres de los buckets para validar si existen.

```bash

kali@kali:~$ for key in "public" "private" "dev" "prod" "development" "production"; do echo "offseclab-assets-$key-axevtewi"; done | tee /tmp/keyfile.txt
offseclab-assets-public-axevtewi
offseclab-assets-private-axevtewi
offseclab-assets-dev-axevtewi
offseclab-assets-prod-axevtewi
offseclab-assets-development-axevtewi
offseclab-assets-production-axevtewi

```

Ahora podemos volver a ejecutar `cloud_enum` con el archivo de claves que acabamos de crear:

```bash

kali@kali:~$ cloud_enum -kf /tmp/keyfile.txt -qs --disable-azure --disable-gcp

...

Keywords:    offseclab-assets-public-axevtewi, offseclab-assets-private-axevtewi, offseclab-assets-dev-axevtewi, offseclab-assets-prod-axevtewi, offseclab-assets-development-axevtewi, offseclab-assets-production-axevtewi
Mutations:   NONE! (Using quickscan)
Brute-list:  /usr/lib/cloud-enum/enum_tools/fuzz.txt

[+] Mutated results: 6 items

++++++++++++++++++++++++++
      amazon checks
++++++++++++++++++++++++++

[+] Checking for S3 buckets
  OPEN S3 BUCKET: http://offseclab-assets-public-axevtewi.s3.amazonaws.com/
      FILES:
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/offseclab-assets-public-axevtewi
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/amethyst-expanded.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/amethyst.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/logo.svg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/pic02.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/pic05.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/pic13.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/ruby-expanded.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/ruby.jpg
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/saphire-expanded.png
      ->http://offseclab-assets-public-axevtewi.s3.amazonaws.com/sites/www/images/saphire.jpg
  Protected S3 Bucket: http://offseclab-assets-private-axevtewi.s3.amazonaws.com/
                            
 Elapsed time: 00:00:06

[+] Checking for AWS Apps
[*] Brute-forcing a list of 6 possible DNS names
                            
 Elapsed time: 00:00:00


[+] All done, happy hacking!

``` 


----


A partir del resultado, podemos confirmar que hay otro bucket, pero está protegido, lo que significa que no es públicamente legible.
También podríamos intentar validar si existen otros buckets utilizando otra información que encontramos durante la fase de reconocimiento. Por ejemplo, podrían ser buckets que incluyan el nombre de los proyectos de offseclab como offseclab-assets-ruby-axevtewi o offseclab-ruby-axevtewi. Dejaremos esto como un ejercicio para el lector.





### 1.3. Análisis DNS y WHOIS
- **Comando DNS**:
  ```bash
  host -t ns example.com
  ```
  - Salida: `ns-1536.awsdns-00.co.uk` (indica Route53).
- **Comando WHOIS**:
  ```bash
  whois example.com | grep "Registrant Organization"
  ```
- **Valor**: Confirma uso de AWS y expone infraestructura asociada.

---

## 2. Enumeración Interna: Con Credenciales

### 2.1. Listado de Recursos con `awscli`
- **Configuración**:
  ```bash
  aws configure --profile pentester
  ```

----

```bash
kali@kali:~$ aws configure --profile attacker
AWS Access Key ID []: AKIAQO...
AWS Secret Access Key []: cOGzm...
Default region name []: us-east-1
Default output format []: json

kali@kali:~$ aws --profile attacker sts get-caller-identity
{
    "UserId": "AIDAQOMAIGYU5VFQCHOI4",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/attacker"
}

```

#### Comandos Útiles

- `aws --profile attacker ec2 describe-images --owners amazon --executable-users all`: Lista imágenes de Amazon disponibles para todos los usuarios ejecutables.
- `aws --profile attacker ec2 describe-images --executable-users all --filters "Name=description,Values=*<keyword>*"`: Filtra imágenes por palabras clave en la descripción.
- `aws --profile attacker ec2 describe-images --executable-users all --filters "Name=name,Values=*<keyword>*"`: Filtra imágenes por palabras clave en el nombre.


  - Ingresa Access Key, Secret Key y región.
- **Listar Buckets S3**:
  ```bash
  aws s3 ls --profile pentester
  ```
- **Listar Instancias EC2**:
  ```bash
  aws ec2 describe-instances --profile pentester
  ```
- **Valor**: Mapea la infraestructura interna rápidamente.

## Obtención de IDs de Cuentas desde Buckets S3 Públicos

Para realizar esta técnica, se necesita una cuenta de AWS para interactuar con la API de AWS. Se utilizará un perfil configurado en AWS CLI para simular este escenario. Además, la cuenta objetivo debe tener un bucket S3 público que permita acceso de lectura.
Primero, se debe crear un usuario IAM que, por defecto, no tendrá permisos para ejecutar acciones. Luego, se añadirá una política que otorgue acceso de lectura al bucket, con la condición de que el permiso solo se aplique si el ID de la cuenta propietaria del bucket comienza con un dígito específico. Si no se puede leer el bucket, se probarán otros números hasta que se logre acceder, lo que permitirá identificar el primer dígito del ID de la cuenta propietaria. Este proceso se puede repetir para obtener todos los dígitos del ID de la cuenta.
El primer paso es seleccionar un bucket o un objeto público dentro de la cuenta objetivo. Dado que el bucket u objeto es público, debería ser posible listar su contenido con cualquier usuario IAM de cualquier cuenta de AWS.
A continuación, se crea un nuevo usuario IAM en la cuenta propia. Por defecto, los usuarios IAM no tienen permisos para ejecutar acciones, por lo que el nuevo usuario no podrá listar el contenido del recurso público incluso si es accesible públicamente.
Después, se define una política que permita listar buckets y leer objetos. Sin embargo, se añade una condición para que el permiso de lectura solo se aplique si el ID de la cuenta propietaria del bucket comienza con un dígito específico.
Finalmente, se prueba si el nuevo usuario puede listar el bucket utilizando sus credenciales. Se prueba con valores de dígitos del 0 al 9 hasta que se logre listar el bucket, lo que confirma el primer dígito del ID de la cuenta.

![](https://static.offsec.com/offsec-courses/PEN-200/imgs/cloud_enum_1/6c8d389ae9d0506cf0c1303c50ba75b7-cldenum_s3_accountID.gif)

----

### Creación de un Usuario IAM para Enumeración

- `kali@kali:~$ aws --profile attacker s3 ls offseclab-assets-public-kaykoour`

Ahora, vamos a crear un nuevo usuario IAM con el comando `iam create-user --user-name enum`. Recordemos que este usuario reside en la cuenta de AWS controlada por el atacante.

A continuación, también crearemos claves de acceso para el usuario IAM, de modo que podamos interactuar con la API de AWS a través de la herramienta AWS CLI. Ejecutaremos el comando `iam create-access-key --user-name enum` y tomaremos nota de los valores de `AccessKeyId` y `SecretAccessKey` en la salida.

```bash
kali@kali:~$ aws --profile attacker iam create-user --user-name enum
{
    "User": {
        "Path": "/",
        "UserName": "enum",
        "UserId": "AIDAQOMAIGYU4HTPEJ32K",
        "Arn": "arn:aws:iam::123456789012:user/enum",
    }
}

kali@kali:~$ aws --profile attacker iam create-access-key --user-name enum
{
    "AccessKey": {
        "UserName": "enum",
        "AccessKeyId": "AKIAQOMAIGYURE7QCUXU",
        "Status": "Active",
        "SecretAccessKey": "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",
    }
}
``` 

----


### 2.2. Enumeración de Usuarios y Roles IAM
- **Listar Usuarios**:
  ```bash
  aws iam list-users --profile pentester
  ```
- **Listar Roles**:
  ```bash
  aws iam list-roles --profile pentester
  ```
- **Valor**: Identifica privilegios excesivos o roles asumibles.

### 2.3. Detección de Configuraciones Erróneas
- **Buckets S3 Públicos**:
  ```bash
  aws s3api get-bucket-policy --bucket example-lab-assets-public --profile pentester
  ```
- **Valor**: Encuentra puntos de entrada para escalar privilegios.

---

## 3. Ejemplo Práctico
1. **Escenario**: Dominio `example.com`.
2. **Paso 1**: Enumerar subdominios:
   ```bash
   dnsenum example.com
   ```
   - Encuentra `dev.example.com`.
3. **Paso 2**: Resolver IP:
   ```bash
   host dev.example.com
   ```
   - Salida: `52.70.117.69` (EC2).
4. **Paso 3**: Probar buckets:
   ```bash
   curl http://example-lab-dev.s3.amazonaws.com/
   ```
   - XML devuelto = bucket público.

---

## 4. Herramientas Recomendadas
- **`awscli`**: Interacción directa con la API de AWS.
- **`cloud_enum`**: Automatiza búsqueda de recursos públicos.
- **`dnsenum`**: Enumeración DNS eficiente.
- **pacu**: Herramienta de enumeración y explotación de AWS.

---

## Obtención de IDs de Cuentas desde Buckets S3

### Descripción
Esta técnica aprovecha las características de la API de AWS para determinar el ID de una cuenta a partir de un bucket S3 público o un objeto accesible, incluso si no hay recursos compartidos públicamente de manera evidente.

### Pasos Clave
1. **Crear un Usuario IAM en la Cuenta del Atacante:**
   - Genera un usuario IAM sin permisos iniciales en tu propia cuenta AWS.
   - Usa el comando:  
     ```bash
     aws iam create-user --user-name enum
     ```
   - Crea claves de acceso para este usuario:  
     ```bash
     aws iam create-access-key --user-name enum
     ```

2. **Configurar las Claves en AWS CLI:**
   - Configura un perfil en AWS CLI con las claves generadas:  
     ```bash
     aws configure --profile enum
     ```

3. **Definir una Política con Condición:**
   - Crea una política que permita leer el bucket solo si el ID de la cuenta que posee el bucket comienza con un dígito específico (por ejemplo, "1"). Ejemplo de política en `policy-s3-read.json`:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowResourceAccount",
                 "Effect": "Allow",
                 "Action": [
                     "s3:ListBucket",
                     "s3:GetObject"
                 ],
                 "Resource": "*",
                 "Condition": {
                     "StringLike": {"s3:ResourceAccount": ["1*"]}
                 }
             }
         ]
     }
     ```
   - Asocia la política al usuario:  
     ```bash
     aws iam put-user-policy --user-name enum --policy-name s3-read --policy-document file://policy-s3-read.json
     ```

4. **Probar el Acceso:**
   - Intenta listar el contenido del bucket con el perfil del usuario:  
     ```bash
     aws --profile enum s3 ls s3://<nombre-del-bucket>
     ```
   - Si falla con "AccessDenied", ajusta el dígito en la condición (de 0 a 9) y repite hasta que tengas éxito, revelando el primer dígito del ID.

5. **Iterar para Dígitos Siguientes:**
   - Modifica la condición para incluir más dígitos (por ejemplo, `"12*"`), y continúa hasta obtener el ID completo.

### Ejemplo Práctico
- Si el bucket es `s3://ejemplo-bucket-publico`, prueba con diferentes condiciones:
  - `"0*"` → AccessDenied
  - `"1*"` → Éxito (el ID comienza con 1)
  - `"12*"` → Éxito (siguiente dígito es 2)
- Automatiza el proceso con un script que itere sobre los dígitos.

---

## Enumeración de Usuarios IAM en Otras Cuentas

### Descripción
Esta técnica abusa de la validación de políticas IAM para identificar usuarios existentes en una cuenta objetivo, utilizando el ID de cuenta previamente obtenido.

### Pasos Clave
1. **Crear un Bucket S3 en la Cuenta del Atacante:**
   - Genera un bucket único:  
     ```bash
     aws s3 mb s3://mi-bucket-$RANDOM-$RANDOM-$RANDOM
     ```

2. **Definir una Política de Acceso:**
   - Crea una política que otorgue acceso a un usuario específico en la cuenta objetivo. Ejemplo en `grant-s3-bucket-read.json`:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "AllowUserToListBucket",
                 "Effect": "Allow",
                 "Resource": "arn:aws:s3:::mi-bucket-12345-67890-54321",
                 "Principal": {
                     "AWS": ["arn:aws:iam::123456789012:user/cloudadmin"]
                 },
                 "Action": "s3:ListBucket"
             }
         ]
     }
     ```
   - Aplica la política al bucket:  
     ```bash
     aws s3api put-bucket-policy --bucket mi-bucket-12345-67890-54321 --policy file://grant-s3-bucket-read.json
     ```

3. **Verificar Existencia:**
   - Si la política se aplica sin errores, el usuario existe.
   - Si retorna "Invalid principal in policy", el usuario no existe.

### Ejemplo Práctico
- Prueba con `arn:aws:iam::123456789012:user/admin`:
  - Éxito → El usuario "admin" existe.
  - Error → Prueba otro nombre como "nonexistent".
- Automatiza con una lista de nombres de usuarios potenciales.

---

## Herramientas Útiles

### Pacu
- **Descripción:** Herramienta para automatizar la enumeración en AWS, incluyendo roles IAM.
- **Instalación en Kali:**
  ```bash
  sudo apt update
  sudo apt install pacu
  ```
- **Uso Básico:**
  1. Inicia Pacu:  
     ```bash
     pacu
     ```
  2. Importa claves:  
     ```
     import_keys <perfil>
     ```
  3. Ejecuta la enumeración de roles:  
     ```
     run iam__enum_roles --word-list /ruta/a/lista.txt --account-id <ID_de_cuenta>
     ```

### AWSBucketDump
- **Descripción:** AWSBucketDump es una herramienta diseñada para enumerar y descargar datos de buckets S3 públicos en AWS.
- https://github.com/jordanpotti/AWSBucketDump.git
- **Uso Básico:**
  1. Ejecuta el script para buscar buckets públicos:
     ```bash
     python AWSBucketDump.py -l lista_de_buckets.txt
     ```
  2. Descarga los datos de los buckets encontrados:
     ```bash
     python AWSBucketDump.py -d -l lista_de_buckets.txt
     ```
- **Valor:** Automatiza la identificación y extracción de datos sensibles de buckets S3 públicos, facilitando la enumeración en entornos AWS.

### Ejemplo con Lista de Roles
- Crea una lista en `/tmp/role-names.txt`:
  ```
  lab_admin
  security_auditor
  content_creator
  ```
- Ejecuta:  
  ```
  run iam__enum_roles --word-list /tmp/role-names.txt --account-id 123456789012
  ```
