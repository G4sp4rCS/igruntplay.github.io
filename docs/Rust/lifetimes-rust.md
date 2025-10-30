# Lifetimes en Rust

Las variables en rust tienen un *alcance* (scope) que define el tiempo de vida de la variable. Este alcance determina cuándo una variable es válida y cuándo deja de serlo. Los *lifetimes* en Rust son una forma de asegurar que las referencias a datos sean válidas durante el tiempo que se utilizan.

Todo prestamo en rust tiene un lifetime asociado que indica cuánto tiempo la referencia es válida. Rust utiliza lifetimes para evitar problemas comunes en la gestión de memoria, como referencias colgantes o uso después de liberar memoria.

En las funciones a veces es necesario especificar lifetimes explícitamente para indicar cómo se relacionan los lifetimes de los parámetros y el valor de retorno. Esto se hace utilizando anotaciones de lifetime, que son etiquetas que comienzan con una comilla simple (`'`).

1. Cada argumento de referencia en una función puede tener su propio lifetime.
2. Si solo hay un argumento con lifetime, ese lifetime se asigna al valor de retorno.
3. Si hay varios y el primero es &self o &mut self, el lifetime de self se asigna al valor de retorno.

---

```rust

#[derive(Debug)]
struct Point(i32, i32);

fn cab_distance(p1: &Point, p2: &Point) -> i32 {
    (p1.0 - p2.0).abs() + (p1.1 - p2.1).abs()
}

fn find_nearest<'a>(points: &'a [Point], query: &Point) -> Option<&'a Point> {
    let mut nearest = None;
    for p in points {
        if let Some((_, nearest_dist)) = nearest {
            let dist = cab_distance(p, query);
            if dist < nearest_dist {
                nearest = Some((p, dist));
            }
        } else {
            nearest = Some((p, cab_distance(p, query)));
        };
    }
    nearest.map(|(p, _)| p)
}

fn main() {
    let points = &[Point(1, 0), Point(1, 0), Point(-1, 0), Point(0, -1)];
    let nearest = {
        let query = Point(0, 2);
        find_nearest(points, &query)
    };
    println!("{:?}", nearest);
}

``` 


----



```rust
#[derive(Debug)]
struct Point(i32, i32);

fn cab_distance(p1: &Point, p2: &Point) -> i32 {
    (p1.0 - p2.0).abs() + (p1.1 - p2.1).abs()
}
``` 

- Acá no neceista anotaciones, las referencias no salen de la función.

---

```rust
fn find_nearest<'a>(points: &'a [Point], query: &Point) -> Option<&'a Point> {
    let mut nearest = None;
    for p in points {
        if let Some((_, nearest_dist)) = nearest {
            let dist = cab_distance(p, query);
            if dist < nearest_dist {
                nearest = Some((p, dist));
            }
        } else {
            nearest = Some((p, cab_distance(p, query)));
        };
    }
    nearest.map(|(p, _)| p)
}
``` 

Este código lo que hace es buscar el punto más cercano a un punto de consulta (`query`) dentro de una lista de puntos (`points`).
- `[Point]`: Es un slice de puntos, es decir, una vista de una secuencia contigua de elementos `Point`.
- `Option<&'a Point>`: Indica que la función puede devolver un `Point` o `None` si no se encuentra ningún punto cercano.
- Some ((p, dist)): Si se encuentra un punto más cercano, se almacena en una tupla junto con su distancia.


Acá si importa lifetime:
- `points: &'a [Point]`: El slice de puntos vive al menos 'a.
- `query: &Point`: No necesita lifetime explícito, ya que no sale de la función.
- `-> Option<&'a Point>`: El punto retornado vive al menos 'a, igual que los puntos en el slice.