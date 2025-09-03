# Smart pointers en Rust

## `Box<T>`
Box es un puntero inteligente que asigna datos en el heap en lugar del stack. Es útil para almacenar datos grandes o estructuras recursivas.

```rust
fn main() {
    let five = Box::new(5);
    println!("five: {}", *five);
}
```

- `Box<T>` implementa el trait `Deref`, lo que permite acceder al valor apuntado usando el operador `*`.
- Los tipos de datos recursivos o tipos de datos con tamaños dinámicos no pueden almacenarse en línea sin una indirección de puntero. Box logra esa indirección:

---

```rust
#[derive(Debug)]
enum List<T> {
    /// A non-empty list: first element and the rest of the list.
    Element(T, Box<List<T>>),
    /// An empty list.
    Nil,
}

fn main() {
    let list: List<i32> =
        List::Element(1, Box::new(List::Element(2, Box::new(List::Nil))));
    println!("{list:?}");
}
```

## Características de `Box<T>`

Box es similar a `std::unique_ptr` en C++, pero garantiza que nunca será null.

Un Box es útil cuando:

- Tienes un tipo cuyo tamaño no se puede conocer en tiempo de compilación, pero el compilador de Rust necesita conocer un tamaño exacto.
- Quieres transferir la propiedad de una gran cantidad de datos. Para evitar copiar grandes cantidades de datos en el stack, almacena los datos en el heap usando Box, así solo se mueve el puntero.

Si no usáramos Box e intentáramos incrustar una Lista directamente en la Lista, el compilador no podría calcular un tamaño fijo para la estructura en memoria (la Lista sería de tamaño infinito).

Box resuelve este problema ya que tiene el mismo tamaño que un puntero regular y simplemente apunta al siguiente elemento de la Lista en el heap.

**Ejemplo de error del compilador:**
Si eliminas Box de la definición de List, obtendrás el mensaje "recursive without indirection", porque para recursión de datos, debemos usar indirección (Box o algún tipo de referencia) en lugar de almacenar el valor directamente.

Aunque Box se parece a `std::unique_ptr` en C++, no puede estar vacío/null. Esto hace que Box sea uno de los tipos que permiten al compilador optimizar el almacenamiento de algunos enums (la "optimización de nicho").


## `Rc<T>`
Rc es un puntero compartido con conteo de referencias. Úsalo cuando necesites referirte a los mismos datos desde múltiples lugares. (Rc standfs for Reference Counted).

```rust
use std::rc::Rc;

fn main() {
    let a = Rc::new(10);
    let b = Rc::clone(&a);

    dbg!(a);
    dbg!(b);
}
```

---
## Características de `Rc<T>`

- El conteo de referencias de `Rc` asegura que el valor contenido sea válido mientras existan referencias.
- `Rc` en Rust es similar a `std::shared_ptr` en C++.
- `Rc::clone` es económico: crea un puntero a la misma asignación e incrementa el contador de referencias. No hace una clonación profunda.
- `make_mut` clona el valor interno si es necesario ("clone-on-write") y retorna una referencia mutable.
- Usa `Rc::strong_count` para verificar el contador de referencias.
- `Rc::downgrade` proporciona un objeto con referencia débil para crear ciclos que se eliminen correctamente (probablemente en combinación con `RefCell`).
