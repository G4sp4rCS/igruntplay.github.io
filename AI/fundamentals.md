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