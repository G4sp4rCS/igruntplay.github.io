# Cuestiones básicas sobre Inteligencia Artificial

## [Para una mejor lectura click aquí](https://apuntes.grunt.ar/4JhtaGBwRFCNQjxFZCBCMQ)

## Regresión Lineal
- La **regresión lineal** es fundamental para los algoritmos de aprendizaje supervisado, que predicen un valor continuo en función de una variable independiente.
- El algoritmo modela esta relación lineal entre la variable independiente y la variable dependiente.
- La regresión lineal se puede utilizar para predecir el precio de una casa en función de su tamaño, la calificación crediticia de un cliente en función de sus ingresos, etc.

### ¿Qué es la "Regresión"?
- La **regresión** es un método estadístico que se utiliza para modelar la relación entre una variable dependiente y una o más variables independientes.
- Se puede pensar como un estimativo de la relación entre las variables.

### Regresión Lineal Simple
- Relaciona un solo predictor (variable independiente) con una variable de respuesta (variable dependiente).
- Siendo su ecuación una función lineal:
    - $y = mx + b$
    - donde:
        - $y$ es la **variable dependiente**
        - $x$ es la **variable independiente**
        - $m$ es la **pendiente de la línea**
        - $b$ es la **intersección de la línea**

### Regresión Lineal Múltiple
- Relaciona dos o más predictores (variables independientes) con una variable de respuesta (variable dependiente).
- La ecuación de la regresión lineal múltiple es:
    - $y = b_0 + b_1 \times x_1 + b_2 \times x_2 + \dots + b_n \times x_n$
    - donde:
        - $y$ es la **variable dependiente**
        - $x_1, x_2, \dots, x_n$ son las **variables independientes**
        - $b_0$ es la **intersección de la línea**
        - $b_1, b_2, \dots, b_n$ son los **coeficientes de regresión**

## Mínimos Cuadrados Ordinarios (OLS)
- Es un método para estimar los coeficientes de regresión en un modelo de regresión lineal.
- El objetivo es minimizar la suma de los cuadrados de las diferencias entre los valores observados y los valores predichos por el modelo.
- La regresión lineal se ajusta a los datos minimizando la suma de los cuadrados de las diferencias entre los valores observados y los valores predichos por el modelo.
- **Proceso de OLS**:
    1. **Calcular restos**: Para cada punto de datos, calcular la diferencia entre el valor observado y el valor predicho.
    2. **Elevar al cuadrado los residuos**: Para cada punto de datos, elevar al cuadrado la diferencia entre el valor observado y el valor predicho.
    3. **Sumar los cuadrados de los residuos**: Sumar los cuadrados de las diferencias entre los valores observados y los valores predichos.
    4. **Minimizar la suma de los cuadrados de los residuos**: Encontrar los coeficientes de regresión que minimizan la suma de los cuadrados de los residuos.

## Asunciones de la Regresión Lineal
- Las asunciones de la regresión lineal son condiciones que deben cumplirse para que los coeficientes de regresión sean insesgados, consistentes y eficientes.
- Las asunciones de la regresión lineal son:
    1. **Linealidad**: La relación entre las variables independientes y la variable dependiente es lineal.
    2. **Independencia**: Los errores de la regresión no están correlacionados.
    3. **Homocedasticidad**: La varianza de los errores de la regresión es constante.
    4. **Normalidad**: Los errores de la regresión siguen una distribución normal.
    5. **Multicolinealidad**: Las variables independientes no están altamente correlacionadas.

## Regresión Logística
- La **regresión logística** es un algoritmo de clasificación que se utiliza para predecir la probabilidad de una variable dependiente categórica.
- La regresión logística es un algoritmo de clasificación binaria, que predice si una observación pertenece a una clase o a otra.
- Es parte del aprendizaje supervisado y se utiliza para problemas de clasificación binaria.
- Clasificación en machine learning es un proceso de asignar una etiqueta a una observación. Como la regresión que predice un valor continuo, la clasificación predice una etiqueta.
- Ejemplos:
    - Clasificar si un correo electrónico es spam o no.
    - Clasificar si un tumor es benigno o maligno.
    - Clasificar si un cliente comprará o no un producto.

### Cómo funciona?
- La regresión logistica utiliza la función logística para modelar la probabilidad de una variable dependiente categórica.
- La función logística es una función sigmoidea que toma cualquier valor real y lo mapea en un valor entre 0 y 1.
- La ecuación de la regresión logística es:
    - $p = \frac{1}{1 + e^{-(b_0 + b_1 \times x_1 + b_2 \times x_2 + \dots + b_n \times x_n)}}$
    - donde:
        - $p$ es la **probabilidad de la variable dependiente**
        - $x_1, x_2, \dots, x_n$ son las **variables independientes**
        - $b_0$ es la **intersección de la línea**
        - $b_1, b_2, \dots, b_n$ son los **coeficientes de regresión**
- Esta función toma cualquier valor real y lo mapea en un valor entre 0 y 1.
- Se utiliza mucho para modelar la probabilidad de una observación que pertenece a una clase o a otra.

#### Ejemplo: Detección de spam
- Digamos que estamos construyendo un filtro de spam utilizando la regresión logística.
- El algoritmo analizaria varios correos electrónicos y aprendería a clasificarlos como spam o no spam.
    - Tendria que analizar las palabras clave, la frecuencia de las palabras, la dirección de correo electrónico, etc.
    - El email tendría que tener un puntaje de probabilidad de spam por debajo de un cierto umbral para ser clasificado como no spam.
- Un aspecto importante de la regresión lineal es la **función de decisión**.
    - La función de decisión es una función que toma un puntaje de probabilidad y lo convierte en una etiqueta.
    - Si el puntaje de probabilidad es mayor que un cierto umbral, la observación se clasifica en una clase.
    - Si el puntaje de probabilidad es menor que un cierto umbral, la observación se clasifica en la otra clase.
    - Hay una recta que divide las dos clases.

##### Umbral de decisión
- El umbral de decisión es el valor que se utiliza para clasificar una observación en una clase o en otra.
- Si el puntaje de probabilidad es mayor que el umbral de decisión, la observación se clasifica en una clase.
- Si dado cierto punto de dato predicado la probabilidad es mayor que el umbral, se clasifica en una clase..
- Si la probabilidad cae por debajo del umbral, se clasifica en la otra clase.
- La ecuación para el umbral de probabilidad es:
    - $p = \frac{1}{1 + e^{-z}}$
    - donde:
        - $p$ es la **probabilidad**
        - $z$ es el **valor de la función lineal** (es decir, $z = b_0 + b_1 \times x_1 + b_2 \times x_2 + \dots + b_n \times x_n$)
- Si $p$ es mayor que el umbral de decisión, la observación se clasifica en una clase.
- Si $p$ es menor que el umbral de decisión, la observación se clasifica en la otra clase.


- [Arboles de decisiones](./DecisionTrees.md)