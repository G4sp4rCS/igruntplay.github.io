# Resumen de Técnicas de Cifrado de Payloads

Cifrar payloads es una técnica fundamental en malware para evadir detección estática, dificultar el análisis y mantener el código ofuscado hasta el momento de ejecución.

## 📌 Objetivos comunes:
- Evadir detección por firmas (AV/EDR).
- Ocultar shellcode o cadenas sensibles.
- Ejecutar el payload en memoria de forma segura.

---

## 1. 🧪 XOR Encryption

### ✨ Características:
- Algoritmo extremadamente simple y reversible.
- Usa el operador lógico XOR (`^`) entre cada byte del payload y una clave.
- Mismo código para cifrar y descifrar.
- Muy liviano, no requiere WinAPI ni librerías externas.

### 🔧 Implementaciones vistas:
```c
// XOR con clave de 1 byte
p[i] = p[i] ^ key;

// XOR con clave dinámica
p[i] = p[i] ^ (key + i);

// XOR con array de clave
p[i] = p[i] ^ key[i % key_size];
```

### ✅ Ventajas:
- Fácil de implementar.
- Bajo consumo de recursos.
- Puede usarse para ofuscar cadenas embebidas o shellcodes cortos.

### ❌ Desventajas:
- Fácilmente detectable por herramientas de análisis.
- Espacio de clave pequeño si no se hace bien.
- No criptográficamente seguro.

---

## 2. RC4 Encryption

### ✨ Características:
- Cifrado de flujo, simétrico y reversible.
- Más fuerte que XOR, pero deprecated en criptografía moderna.
- Puede usarse con librerías propias o funciones ocultas de Windows.

### 🔧 Implementaciones vistas:

#### Método 1: RC4 manual (con estructura `Rc4Context`)
- Inicialización con `rc4Init`.
- Cifrado con `rc4Cipher`.

#### Método 2: SystemFunction032 (WinAPI, Advapi32.dll)
```c
typedef NTSTATUS (NTAPI *fnSystemFunction032)(USTRING* Data, USTRING* Key);
```

#### Método 3: SystemFunction033
- Idéntico al anterior, distinta función.
- Misma estructura, diferente nombre para evadir detección.

### ✅ Ventajas:
- Rápido y fácil de usar en código real.
- Implementación con WinAPI es compacta y puede aprovechar funciones del sistema.
- Ideal para payloads medianos.

### ❌ Desventajas:
- Puede dejar rastros en el IAT (`advapi32.dll`).
- Puede ser detectable por AV si no se ofusca.
- Menos seguro que AES.

---

## 3. AES Encryption

### ✨ Características:
- Algoritmo seguro y moderno, por bloques (128 bits).
- AES-256-CBC requiere:
    - Clave de 32 bytes.
    - IV (Initialization Vector) de 16 bytes.
- Requiere padding si el tamaño del payload no es múltiplo de 16.

### 🔧 Implementaciones vistas:

#### 🔹 Con WinAPI (`bcrypt`):
Usa funciones como:
- `BCryptOpenAlgorithmProvider`
- `BCryptEncrypt` / `BCryptDecrypt`
- Padding automático con `BCRYPT_BLOCK_PADDING`.

#### 🔹 Con Tiny-AES-C:
- Librería portable, sin dependencias.
- No deja trazas en el IAT.
- No soporta padding → se requiere implementación manual.

### ✅ Ventajas:
- Muy seguro.
- Compatible con estándares modernos.
- Ideal para proteger payloads grandes o sensibles.

### ❌ Desventajas:
- Requiere más código.
- Con `bcrypt`, deja trazas en el IAT.
- Con Tiny-AES, se deben manejar detalles como padding y detección por firmas.

---

## 🧮 Complejidad Temporal

| Algoritmo | Complejidad Temporal |
|-----------|-----------------------|
| XOR       | O(n)                 |
| RC4       | O(n)                 |
| AES       | O(n)                 |

> **Nota:** `n` = tamaño del payload en bytes. Todos procesan byte a byte o bloque a bloque.

---

## 🧠 Conclusión y Recomendaciones

| Objetivo                                | Mejor opción                          |
|-----------------------------------------|---------------------------------------|
| Ofuscar una string o dato pequeño       | XOR con clave variable                |
| Cifrar shellcode sin depender de WinAPI | Tiny-AES con padding                  |
| Cifrar payloads con APIs nativas        | AES con bcrypt (WinAPI)               |
| Evadir detección de funciones criptográficas | RC4 con SystemFunction033         |
| Código muy compacto y rápido            | RC4 o XOR                             |

---

## Códigos
- En maldev academy