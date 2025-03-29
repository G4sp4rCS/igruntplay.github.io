[Mejor legibilidad](https://apuntes.grunt.ar/s/iTRb4dtfZ)

# Bayesiano ingenuo

![](https://academy.hackthebox.com/storage/modules/290/bayes_classification.png)
- Es un algoritmo probabilistico utilizado para problemas de clasificación basado en el teorema de Bayes.

## Teorema de Bayes
- El teorema de Bayes es una ecuación que describe la relación de probabilidades condicionales de cantidades estadísticas.
- El teorema de Bayes es:
    - $P(A|B) = \frac{P(B|A) \times P(A)}{P(B)}$
    - donde:
        - $P(A|B)$ es la probabilidad de A dado B.
        - $P(B|A)$ es la probabilidad de B dado A.
        - $P(A)$ es la probabilidad de A.
        - $P(B)$ es la probabilidad de B.

## Como el bayesiano ingenuo funciona?
- El clasificador Bayesiano ingenuo utiliza el teorema de Bayes para predecir la probabilidad de que un punto de datos pertenezca a una clase particular dadas sus características. Para lograr esto, hace la suposición "ingenua" de independencia condicional entre las características. Esto significa que asume que la presencia o ausencia de una característica no afecta la presencia o ausencia de otra característica, dado que conocemos la etiqueta de la clase.
- Aunque esta es una suposición simplificada, el clasificador bayesiano ingenuo ha demostrado ser efectivo en la práctica.

----


### Calcular Probabilidades Previas
El algoritmo primero calcula la probabilidad previa de cada clase. Esta es la probabilidad de que un punto de datos pertenezca a una clase particular antes de considerar sus características. Por ejemplo, en un escenario de detección de spam, la probabilidad de que un correo electrónico sea spam podría ser 0.2 (20%), mientras que la probabilidad de que no sea spam es 0.8 (80%).

### Calcular Verosimilitudes
A continuación, el algoritmo calcula la verosimilitud de observar cada característica dada cada clase. Esto implica determinar la probabilidad de observar un valor particular de una característica dado que el punto de datos pertenece a una clase específica. Por ejemplo, ¿cuál es la probabilidad de ver la palabra "gratis" en un correo electrónico dado que es spam? ¿Cuál es la probabilidad de ver la palabra "reunión" dado que no es spam?

### Aplicar el Teorema de Bayes
Para un nuevo punto de datos, el algoritmo combina las probabilidades previas y las verosimilitudes utilizando el teorema de Bayes para calcular la probabilidad posterior de que el punto de datos pertenezca a cada clase. La probabilidad posterior es la probabilidad actualizada de un evento (en este caso, que el punto de datos pertenezca a una cierta clase) después de considerar nueva información (las características observadas). Esto representa la creencia revisada sobre la etiqueta de clase después de considerar las características observadas.

## Tipos de Clasificadores Bayesianos Ingenuos

### Predecir la Clase
Finalmente, el algoritmo asigna el punto de datos a la clase con la probabilidad posterior más alta.
## Tipos de Clasificadores Bayesianos Ingenuos

La implementación específica del clasificador Bayesiano ingenuo depende del tipo de características y su distribución asumida:

### Bayesiano Ingenuo Gaussiano
Se utiliza cuando las características son continuas y se asume que siguen una distribución gaussiana (una curva en forma de campana). Por ejemplo, si se predice si un cliente comprará un producto en función de su edad e ingresos, se podría usar el Bayesiano ingenuo gaussiano, suponiendo que la edad y los ingresos están distribuidos normalmente.

### Bayesiano Ingenuo Multinomial
Es adecuado para características discretas y se utiliza a menudo en la clasificación de texto. Por ejemplo, en la detección de spam, la frecuencia de palabras como "gratis" o "dinero" podría ser una característica, y el Bayesiano ingenuo multinomial modelaría la probabilidad de que estas palabras aparezcan en correos electrónicos de spam y no spam.

### Bayesiano Ingenuo Bernoulli
Este tipo se emplea para características binarias, donde la característica está presente o ausente. En la clasificación de documentos, una característica podría ser si una palabra específica está presente en el documento. El Bayesiano ingenuo bernoulli modelaría la probabilidad de esta presencia o ausencia para cada clase.

## Suposiciones de los Datos

Aunque el Bayesiano ingenuo es relativamente robusto, es útil tener en cuenta algunas suposiciones sobre los datos:

- **Independencia de las Características**: Como se mencionó, la suposición principal es que las características son condicionalmente independientes dado la clase.
- **Distribución de los Datos**: La elección del clasificador Bayesiano ingenuo (Gaussiano, Multinomial, Bernoulli) depende de la distribución asumida de las características.
- **Datos de Entrenamiento Suficientes**: Aunque el Bayesiano ingenuo puede funcionar con datos limitados, es importante contar con suficientes datos para estimar las probabilidades con precisión.
