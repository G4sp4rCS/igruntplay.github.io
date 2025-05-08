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
