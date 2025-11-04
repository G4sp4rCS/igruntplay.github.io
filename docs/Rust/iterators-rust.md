# 🦀 Iteradores en Rust

## Introducción

Un **iterador** en Rust es un objeto que produce una secuencia de valores uno por uno.  
Su comportamiento está definido por el *trait* `Iterator`, y se usa ampliamente para recorrer, transformar y procesar colecciones.

Los iteradores son **perezosos (lazy)**: no realizan trabajo hasta que se los consume con métodos como `.collect()`, `.sum()`, `.count()`, etc.

---

## Trait `Iterator`

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

- **`Item`**: tipo de elemento que el iterador produce.  
- **`next()`**: devuelve `Some(valor)` mientras haya elementos, y `None` al terminar.

Ejemplo:

```rust
let v = vec![1, 2, 3];
let mut it = v.iter();

assert_eq!(it.next(), Some(&1));
assert_eq!(it.next(), Some(&2));
assert_eq!(it.next(), Some(&3));
assert_eq!(it.next(), None);
```

---

## Adaptadores de iteradores

Los métodos que **devuelven nuevos iteradores** se llaman *adaptadores*.  
No ejecutan nada hasta que se llega a un consumidor.

### Ejemplos comunes:

| Método | Descripción |
|--------|--------------|
| `.map(f)` | Aplica una función a cada elemento |
| `.filter(pred)` | Retiene solo los elementos que cumplan una condición |
| `.take(n)` | Toma los primeros `n` elementos |
| `.skip(n)` | Salta los primeros `n` elementos |
| `.cycle()` | Repite infinitamente la secuencia |
| `.zip(iter2)` | Combina dos iteradores en pares `(a, b)` |
| `.enumerate()` | Agrega índices `(i, valor)` |

Ejemplo:

```rust
let nums = vec![1, 2, 3, 4];
let result: Vec<_> = nums.iter()
    .filter(|x| **x % 2 == 0)
    .map(|x| x * x)
    .collect();

assert_eq!(result, vec![4, 16]);
```

---

## Consumidores de iteradores

Son métodos que **consumen el iterador** y devuelven un valor o colección.

| Método | Descripción |
|--------|--------------|
| `.collect()` | Convierte los resultados en una colección (`Vec`, `HashMap`, etc.) |
| `.sum()`, `.product()` | Calculan suma o producto |
| `.count()` | Cuenta los elementos |
| `.any()`, `.all()` | Evalúan condiciones |
| `.find()`, `.position()` | Buscan elementos |
| `.for_each(f)` | Ejecutan una función por elemento (sin devolver nada) |

Ejemplo:

```rust
let primes = vec![2, 3, 5, 7];
let squares: Vec<_> = primes.into_iter().map(|p| p * p).collect();
println!("{squares:?}"); // [4, 9, 25, 49]
```

---

## Trait `IntoIterator`

`IntoIterator` define **cómo obtener un iterador a partir de un tipo**.  
Es el trait que hace funcionar los `for` loops en Rust.

```rust
trait IntoIterator {
    type Item;
    type IntoIter: Iterator<Item = Self::Item>;
    fn into_iter(self) -> Self::IntoIter;
}
```

Ejemplo:

```rust
struct Grid {
    x_coords: Vec<u32>,
    y_coords: Vec<u32>,
}

struct GridIter {
    grid: Grid,
    i: usize,
    j: usize,
}

impl Iterator for GridIter {
    type Item = (u32, u32);
    fn next(&mut self) -> Option<Self::Item> {
        if self.i >= self.grid.x_coords.len() {
            self.i = 0;
            self.j += 1;
            if self.j >= self.grid.y_coords.len() {
                return None;
            }
        }
        let res = (self.grid.x_coords[self.i], self.grid.y_coords[self.j]);
        self.i += 1;
        Some(res)
    }
}

impl IntoIterator for Grid {
    type Item = (u32, u32);
    type IntoIter = GridIter;
    fn into_iter(self) -> GridIter {
        GridIter { grid: self, i: 0, j: 0 }
    }
}

fn main() {
    let grid = Grid { x_coords: vec![1, 2], y_coords: vec![10, 20] };
    for (x, y) in grid {
        println!("({x}, {y})");
    }
}
```

📍 Nota: `for` en Rust traduce internamente a:
```rust
let mut iter = grid.into_iter();
while let Some(item) = iter.next() { ... }
```

---

## 🔁 Iteradores infinitos y combinadores

Ejemplo de uso con `cycle()` y `skip()`:

```rust
fn offset_differences(offset: usize, values: Vec<i32>) -> Vec<i32> {
    let a = values.iter();
    let b = values.iter().cycle().skip(offset);
    a.zip(b).map(|(a, b)| *b - *a).collect()
}

fn main() {
    let result = offset_differences(1, vec![1, 2, 3, 4]);
    println!("{result:?}"); // [1, 1, 1, -3]
}
```

- `cycle()` repite infinitamente los elementos.
- `skip(n)` descarta los primeros `n`.
- `zip()` combina ambos iteradores.
- `map()` aplica una función sobre los pares.
- `collect()` materializa el resultado.

---

## ⚡ Ventajas sobre los ciclos tradicionales

| Ventaja | Descripción |
|----------|-------------|
| **Composición funcional** | Encadenás operaciones como un pipeline (`map → filter → collect`) |
| **Sin mutabilidad** | Evita variables temporales o contadores |
| **Seguridad** | Nunca hay riesgo de “índice fuera de rango” |
| **Lazy evaluation** | Las operaciones no se ejecutan hasta que se consumen |
| **Reutilización** | Podés combinar, extender y adaptar iteradores fácilmente |

---

## 📚 Referencias

- [Comprehensive Rust — Iterators](https://google.github.io/comprehensive-rust/iterators.html)
- [Rust Docs: std::iter](https://doc.rust-lang.org/std/iter/)
- [Rust by Example — Iterators](https://doc.rust-lang.org/rust-by-example/trait/iter.html)
