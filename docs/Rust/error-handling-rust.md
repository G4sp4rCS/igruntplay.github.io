# Error Handling en Rust

## 1. Panics

En Rust, los *panics* representan errores irrecuperables. Cuando ocurre un `panic!`, el programa finaliza abruptamente.  
Se utilizan en situaciones donde continuar la ejecución no tendría sentido, por ejemplo:

```rust
panic!("Algo salió mal");
```

Es recomendable evitar `panic!` en código de producción o librerías. En su lugar, se prefiere devolver un `Result` y manejar el error explícitamente.

---

## 2. Result

`Result<T, E>` es un *enum* estándar que representa éxito (`Ok`) o error (`Err)`:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Ejemplo:

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("División por cero".into())
    } else {
        Ok(a / b)
    }
}
```

Manejar un `Result` puede hacerse con `match`, `if let` o el operador `?`.

---

## 3. Operador `?` y Try

El operador `?` simplifica la propagación de errores. Expande a un `match` que devuelve automáticamente en caso de error.

```rust
fn read_file(path: &str) -> Result<String, std::io::Error> {
    let mut s = String::new();
    std::fs::File::open(path)?.read_to_string(&mut s)?;
    Ok(s)
}
```

Expansión implícita:

```rust
match std::fs::File::open(path) {
    Ok(file) => file,
    Err(e) => return Err(From::from(e)),
}
```

El `From::from` convierte el tipo de error automáticamente si la función devuelve un `Result` con otro tipo compatible.

---

## 4. Try Conversions (`From` y `map_err`)

Cuando los errores tienen tipos distintos, se usa el trait `From` para definir conversiones entre ellos:

```rust
impl From<io::Error> for MyError {
    fn from(err: io::Error) -> MyError {
        MyError::Io(err)
    }
}
```

Esto permite usar `?` sin necesidad de convertir manualmente.  
Alternativamente, puede usarse `map_err` para una conversión puntual:

```rust
fs::File::open(path).map_err(MyError::Io)?;
```

---

## 5. Trait `Error`

`std::error::Error` es un trait que define el comportamiento común de todos los errores:

```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
struct MyError;

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Ocurrió un error")
    }
}

impl Error for MyError {}
```

Implementar `Display` y `Error` permite describir e integrar los errores con el sistema estándar.

---

## 6. Tipos de error dinámicos (`Box<dyn Error>`)

Cuando una función puede devolver múltiples tipos de error, puede usarse `Box<dyn Error>`:

```rust
fn read_count(path: &str) -> Result<i32, Box<dyn std::error::Error>> {
    let mut s = String::new();
    std::fs::File::open(path)?.read_to_string(&mut s)?;
    let n: i32 = s.parse()?;
    Ok(n)
}
```

Esto permite devolver cualquier tipo de error que implemente `Error`, sin crear un `enum` específico.  
Es más flexible pero menos preciso (pierde el tipo estático del error).

---

## 7. Crate `thiserror`

El crate `thiserror` automatiza la implementación de traits como `From`, `Display` y `Error`, reduciendo el *boilerplate*.

Ejemplo:

```rust
use std::io;
use thiserror::Error;

#[derive(Debug, Error)]
enum ReadUsernameError {
    #[error("I/O error: {0}")]
    Io(#[from] io::Error),
    #[error("Found no username in {0}")]
    EmptyUsername(String),
}

fn read_username(path: &str) -> Result<String, ReadUsernameError> {
    let mut username = String::new();
    std::fs::File::open(path)?.read_to_string(&mut username)?;
    if username.is_empty() {
        return Err(ReadUsernameError::EmptyUsername(path.into()));
    }
    Ok(username)
}
```

Ventajas:
- Genera automáticamente las implementaciones de `Display`, `Error` y `From`.
- Mejora la legibilidad y mantiene el código limpio.
- Ideal para definir tipos de error en aplicaciones medianas o grandes.

---

## 8. Crate `anyhow`

`anyhow` está diseñado para programas (no librerías) y simplifica la gestión de errores al permitir devolver *cualquier* tipo de error con contexto adicional.

```rust
use anyhow::{Result, Context};
use std::fs;

fn read_config() -> Result<String> {
    let data = fs::read_to_string("config.toml")
        .context("Error al leer el archivo de configuración")?;
    Ok(data)
}
```

Características:
- Usa `anyhow::Result<T>` como alias de `Result<T, anyhow::Error>`.
- Permite agregar contexto con `.context("...")`.
- Adecuado para CLI o binarios donde solo se necesita mostrar un mensaje descriptivo.

---

## Conclusión

Rust ofrece múltiples niveles de manejo de errores:
- **`panic!`**: errores fatales.
- **`Result<T, E>`**: control total sobre errores recuperables.
- **`From` / `?`**: simplificación en la propagación.
- **`thiserror`**: automatiza el manejo estructurado de errores.
- **`anyhow`**: simplifica el manejo dinámico en aplicaciones.

Cada enfoque tiene su lugar dependiendo del grado de control, precisión o conveniencia que se necesite.
