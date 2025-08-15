# Closures en Rust

## 1. ¿Qué es una closure?
Una **closure** es una función anónima que puede **capturar variables** de su entorno.  
Sintaxis básica:

```rust
|parametros| -> TipoRetorno { cuerpo }
```

### Ejemplos

```rust
|n| n * 2                  // inferencia de tipo
|x: f32| -> f32 { x + 1.0 } // con tipos explícitos
|| println!("sin args")     // sin parámetros
```

## 2. Captura de variables
Las closures pueden capturar variables del entorno donde se definen.

Las closures pueden capturar de tres formas:

| Modo de captura | Trait | Cuándo ocurre |
|---|---|---|
| Referencia inmutable | Fn | Solo lectura |
| Referencia mutable | FnMut | Modifica variables capturadas |
| Por valor (mueve datos) | FnOnce | Consume variables capturadas (con move) |


### Por ejemplo

```rust
// Fn: solo lectura
let max_value = 5;
let clamp = |v| v.min(max_value); // captura max_value por referencia inmutable

// FnMut: muta estado
let mut sum = 0;
let mut add_to_sum = |x| sum += x;

// FnOnce: consume
let data = vec![1, 2, 3];
let consume_data = move || {
    println!("{:?}", data);
}; // Lo que ocurre acá es que se mueve el vector `data` dentro de la closure haciendo que no se pueda usar fuera de ella.
```


## 3. utilización de move

Forza la captura por valor, moviendo la propiedad de las variables a la closure.

Útil para:
- Threads `(thread::spawn)`
- Evitar problemas de lifetime
- Asegurar que la closure tenga su propia copia de datos
- Evitar referencias a variables externas
- Entre mil cositas más.

### Ejemplo: mover propiedad a un hilo

`move` transfiere la propiedad de las variables capturadas a la closure, lo cual es necesario al usar `thread::spawn` porque el hilo puede vivir más que el stack local.

Puntos clave
- `move` captura por valor; los datos deben ser `Send` (y a menudo `'static`).
- Tras el `move` la variable original ya no es usable en el hilo llamador.
- Para compartir estado entre hilos usa `Arc<Mutex<T>>`.
- Llama a `join()` en el `JoinHandle` para esperar y manejar errores.

Ejemplo:

```rust
use std::thread;

let payload = vec![0x90, 0x90, 0xCC]; // NOP NOP INT3
thread::spawn(move || {
    // aquí payload es propiedad de la closure
    println!("Payload size: {}", payload.len());
}).join().unwrap(); // join y unwrap hacen espera y manejo de errores
```

---

## 4. Jerarquia de traits
- Los traits son interfaces que definen comportamientos.
- Las closures implementan los traits `Fn`, `FnMut` y `FnOnce`
- Fn es el más flexible: se puede usar donde pidan cualquiera de los tres.
- FnMut se puede usar donde pidan FnMut o FnOnce.
- FnOnce solo donde pidan FnOnce.

### Ejemplo de jerarquía de traits

```rust
fn call_fn<F: Fn()>(f: F) {
    f();
}

fn call_fn_mut<F: FnMut()>(mut f: F) {
    f();
}

fn call_fn_once<F: FnOnce()>(f: F) {
    f();
}

// La verdad es que esto no lo entiendo del todo bien.

``` 

---

## 5 Punteros a función `fn`
Si una closure no captura nada, puede convertirse en un puntero de función

```rust
let add1 = |x: i32| x + 1; 
let f: fn(i32) -> i32 = add1;

// Ahora `f` es un puntero a función que puede ser llamado como `f(5)`
//Entonces podemos imprimir
println!("Resultado: {}", f(5)); // Resultado: 6
``` 

### Otro ejemplo de puntero a función más complejo

Demostración más útil: usar `apply_fn` con una closure que captura entorno, con una función normal (puntero) y un ejemplo de hilos.

Aplica una función o clausura que toma un i32 y devuelve un i32 al valor `x`.

Esta es una función de orden **superior genérica**: recibe un parámetro de tipo `F`
que implementa el trait `Fn(i32) -> i32` (es decir, cualquier función o clausura
que acepte un `i32` y produzca un `i32`). La función simplemente llama a `f(x)`
y devuelve el resultado.

Propósito práctico:
- Abstraer la operación de "aplicar" una transformación sobre un entero sin fijar
  qué transformación es.
- Permitir pasar tanto funciones libres como clausuras que puedan capturar el
  entorno (siempre que sean compatibles con `Fn`).
- Si necesitas que la clausura pueda mutar su entorno, considera usar `FnMut`;
  si la clausura se consume al invocarse, usa `FnOnce`.

Parámetros:
- `f`: la función o clausura a aplicar sobre `x`.
- `x`: el entero que se pasará a `f`.

Retorna:
- El resultado de `f(x)` como `i32`.

- Ejemplo:

```rust

fn apply_fn<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(x)
}

fn add_one(n: i32) -> i32 {
    n + 1
}

fn main() {
    // Closure que captura por referencia inmutable
    let factor = 3;
    let multiply = |n: i32| n * factor;

    // Closure que muta estado externo (FnMut)
    let mut total = 0;
    {
        let mut add_to_total = |n: i32| total += n;
        add_to_total(5);
        add_to_total(10);
    }

    println!("apply_fn con closure multiply: {}", apply_fn(multiply, 4)); // 12
    println!("apply_fn con función add_one: {}", apply_fn(add_one, 4)); // 5

    // Ejemplo con threads: mover propiedad a la closure
    let nums = vec![1, 2, 3];
    let handles: Vec<_> = nums.into_iter().map(|n| {
        std::thread::spawn(move || n * 2)
    }).collect();

    for h in handles {
        println!("Resultado del hilo: {}", h.join().unwrap());
    }

    println!("total acumulado = {}", total);
}
``` 

### Analogía de abstracción porque si no no se entiende del todo bien

#### Analogía: Un ayudante de cocina robotico
Imaginemos que construimos un robot de cocina llamado `apply_fn`. Su unica tarea es tomar un ingrediente `x`y aplicarle una acción de cocina `f` que le pasamos.

```rust
// Nuestro robot de cocina
fn apply_fn<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    // f: la acción de cocina (ej: cortar, freír, mezclar)
    // x: el ingrediente (ej: una papa, representada por un número)
    f(x) // El robot aplica la acción al ingrediente
}
``` 

Ahora tenemos diferentes "recetas" (funciones o closures) que pueden ser pasadas al robot:
- receta `add_one` (función normal): "Pelar papa", O sea sumarle 1 al número.
- receta `multiply` (closure): "Cortar papa en trozos de tamaño `factor`", O sea multiplicar el número por un factor.

Nosotros como jefe de cocina podemos decirle al robot:
- "Agarrá esta papa `4` y usá la receta `add_one` para prepararla".
    - El robot hace `apply_fn(add_one, 4)` y nos devuelve `5`.
- "Robot, agarrá esta papa `4` y usá la receta `multiply` para prepararla".
    - El robot hace `apply_fn(multiply, 4)` y nos devuelve `12`.

Ahora imaginemos una función `preparar_gran_banquete` que:
- Lava 100 papas.
- Lllama al robot `apply_fn` para que haga algo con cada papa.
- Hornea todas las papas.

Ahora podemos utilizar `preparar_gran_banquete` con diferentes recetas:
- papas fritas: `preparar_gran_banquete(cortar_en_bastones)`
- puré de papas: `preparar_gran_banquete(pisar_papas)`

Entonces no hace falta reescribir los pasos de lavar y hornear, solo cambiamos la receta que le pasamos al robot.

### El Poder en el Mundo Real: Más Allá de `apply_fn`
Este patrón es la base de muchos metodos que se usan constantemente en Rust, especialmente con iteradores. Si pensamos en el metodo `.map()` es exactamente como `apply_fn`, pero aplicado a cada elemento de una colección.

```rust
let numeros = vec![1, 2, 3];

// .map() es una función de orden superior como apply_fn.
// Le pasas una "receta" (la closure |n| n * 2) y la aplica a cada elemento.
let dobles: Vec<_> = numeros.iter().map(|n| n * 2).collect(); // [2, 4, 6]
``` 

Acá `.map()` está abstrayendo el bucle y la logica de aplicar una transformación. Nosotros solo nos preocupamos por definir que transformación queremos aplicar a cada elemento.