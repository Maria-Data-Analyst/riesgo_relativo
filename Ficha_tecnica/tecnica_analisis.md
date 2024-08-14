# TÉCNICA DE ANÁLISIS
## RIESGO RELATIVO

El riesgo relativo en segmentos de variables específicas me permitió comparar la probabilidad de que un cliente fuera un mal pagador en cada segmento. Esto me ayudó a identificar qué segmentos tenían mayor riesgo en comparación con otros.

Calcule el riesgo relativo para todas las variables que segmenté en el análisis exploratorio. Esto me permitió ver qué segmentos estaban más expuestos a ser catalogados como malos pagadores. Además, estos resultados me proporcionaron la base para validar o refutar las hipótesis planteadas y para definir el perfil de los clientes con mayor riesgo de morosidad.

![Captura de pantalla 2024-08-11 162848](https://github.com/user-attachments/assets/02fb37b8-6f21-4252-b494-721a54451dd7)



![Captura de pantalla 2024-08-11 162731](https://github.com/user-attachments/assets/7f463a5b-46d8-4fca-aee7-20029dc9242b)

**Validación de Hipótesis**

**1. ¿Los más jóvenes tienen un mayor riesgo de impago?**
Los resultados muestran que los rangos etarios más jóvenes (21 - 35 y 36 - 46) tienen un riesgo relativo más alto de impago en comparación con los demás rangos. Por lo tanto se valida la hipotesis 

**2. ¿Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores?**
Los resultados muestran que las personas con entre 1 y 4 préstamos activos tienen un riesgo  más alto de impago en comparación con el resto de segmentos.Estos resultados no apoyan la hipótesis de que una mayor cantidad de préstamos activos está asociada con un mayor riesgo de impago. De hecho, el riesgo parece ser menor en los grupos con más préstamos activos.

**3. ¿Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores?**

Los resultados muestran que las personas que han retrasado sus pagos por más de 90 días tienen un riesgo significativamente mayor de impago  en comparación con aquellas que no han tenido retrasos en sus pagos. Los valores altos de riesgo relativo en los rangos de retraso indican que cuanto mayor es el retraso en los pagos, mayor es el riesgo de ser un mal pagador.
Esto apoya la hipótesis de que las personas con retrasos prolongados en sus pagos tienen un mayor riesgo de ser malos pagadores.

## HITO 2 : Score Crediticio 

Al optimizar la evaluación del riesgo crediticio, decidí enfocarme en tres variables clave: último salario, retraso de más de 90 días, y uso de crédito en líneas no aseguradas. Estas variables fueron seleccionadas por las siguientes razones:

**1. Último Salario:**

**Relevancia Directa:** El último salario se consideró un indicador fundamental de la capacidad de pago del cliente. Un salario más alto generalmente sugiere una mayor capacidad para cumplir con las obligaciones financieras.

**Estabilidad Financiera:** Esta variable ofreció una visión clara de la estabilidad financiera del cliente, ayudando a evaluar su capacidad para manejar nuevas obligaciones crediticias.

**2. Retraso de Más de 90 Días:**

**Historial de Mora:** El retraso en los pagos por más de 90 días fue un fuerte indicador de problemas financieros serios y un alto riesgo de impago. Esta variable proporcionó una señal clara de la probabilidad de que un cliente pudiera enfrentar dificultades en el futuro.

**Impacto Significativo:** Los retrasos prolongados tuvieron un impacto significativo en la evaluación del riesgo, ya que reflejaron patrones de comportamiento que indicaban problemas de solvencia.

**3. Uso de Crédito en Líneas No Aseguradas:**

**Nivel de Endeudamiento:** El uso de crédito en líneas no aseguradas indicaba el nivel de endeudamiento del cliente sin respaldo de activos. Un alto uso de estas líneas pudo ser un indicio de estrés financiero y un mayor riesgo de impago.

**Exposición al Riesgo:** Esta variable ayudó a identificar clientes que podrían estar sobrecargados financieramente, afectando su capacidad para cumplir con nuevos compromisos de crédito.

**Variables Excluidas:**

**Edad:** Aunque la edad podría ofrecer contexto, su impacto en el riesgo crediticio no resultó ser tan directo como el de las variables seleccionadas. La capacidad de pago y el historial financiero se consideraron indicadores más precisos del riesgo de impago.

**Número de Préstamos:** El número total de préstamos no reflejó adecuadamente la capacidad de un cliente para manejar nuevos créditos. Un cliente con varios préstamos pudo tener un historial sólido y una buena capacidad de pago, por lo que esta variable se consideró menos relevante para la evaluación del riesgo en comparación con las variables seleccionadas.

En el código de BigQuery, se crearon variables bandera para facilitar la visualización de los usuarios en los segmentos de riesgo relativo. Estas variables, denominadas salary_rr_flag, more90_rr_flag, y using_lines_rr_flag, indican si un cliente pertenece a segmentos de riesgo en las tres categorías consideradas.

Un cliente será clasificado como buen pagador si todas estas variables bandera están en estado "apagado", lo que indica que no está en riesgo en ninguna de las categorías. Sin embargo, si al menos una de estas variables bandera está activada, el cliente será marcado como mal pagador.

A pesar de que la activación de una sola bandera ya indique un riesgo, también crearemos una variable adicional llamada puntos que sumará los valores de las tres variables bandera. Esto nos permitirá evaluar la magnitud del riesgo del cliente, identificando si está en riesgo en una, dos o en las tres categorías. Esta métrica adicional proporcionará una visión más detallada del perfil de riesgo del cliente.


![image](https://github.com/user-attachments/assets/9cafe854-c62f-4d71-a9f2-ebb976c467c5)


**Matriz de confusión**
Con el puntaje asignado y la identificación de malos pagadores, podemos evaluar nuestro modelo de detección de malos pagadores creando una matriz de confusión en Google Colab. Esta matriz nos permitirá comparar nuestra clasificación con la existente en la base de datos, denominada default_flag, para validar la efectividad de nuestro modelo.

![Captura de pantalla 2024-08-11 182942](https://github.com/user-attachments/assets/bbb8aeef-1f50-46a8-9092-2e09df172f6c)

El modelo que he implementado para identificar malos pagadores se basa en tres condiciones específicas:

* Retrasos en los pagos superiores a 90 días.
* Uso de líneas de crédito no aseguradas superior al 70%.
* Ingresos mensuales inferiores a $3,947.
  
**Resultados del Modelo:**

El modelo ha alcanzado un recall de 1 para todos los clientes que actualmente están clasificados como malos pagadores según el default_flag existente. Además, ha identificado a 13,035 clientes adicionales que el default_flag considera como buenos pagadores, pero que cumplen con al menos una de las condiciones de riesgo establecidas en el nuevo modelo

**Implicaciones:**

**Discrepancias con el default_flag Actual:** La discrepancia entre el nuevo modelo y el default_flag sugiere que el actual default_flag podría estar subestimando el riesgo. El modelo propuesto ha identificado un 37% más de malos pagadores en comparación con el default_flag, que solo identifica el 1.75% de los clientes como malos pagadores.

**Necesidad de Revisión:** Es crucial revisar los criterios utilizados para definir el default_flag. Es posible que el default_flag actual esté basado en información clave que no está presente en el conjunto de datos actual, lo que podría estar contribuyendo a la discrepancia observada en las clasificaciones. Una revisión detallada permitirá identificar y corregir posibles deficiencias en los datos y mejorar la precisión de la clasificación de malos pagadores.

## HITO 3: Regresión Logística

El modelo de regresión logística es una técnica estadística que estima la probabilidad de que ocurra un evento, en este caso, que un cliente sea clasificado como "mal pagador". Este modelo utiliza una fórmula matemática para relacionar una o más variables independientes con la probabilidad de que se produzca el evento.

He decidido usar variables segmentadas en lugar de continuas para el modelo de regresión logística, ya que permiten capturar efectos no lineales, facilitan la interpretación al dividir los datos en intervalos manejables y ayudan a manejar outliers al agrupar valores extremos. Dado que la regresión logística requiere entradas numéricas, estas variables deben ser convertidas en dummy, creando columnas binarias que el modelo puede procesar de manera efectiva. Además, he optado por incluir la variable number_dependents, que no se había considerado previamente, para evaluar su impacto en la regresión.

Data columns (total 26 columns):

| #  | Column                             | Non-Null Count | Dtype |
|----|------------------------------------|----------------|-------|
| 0  | number_dependents                  | 35546 non-null | Int64 |
| 1  | default_flag                       | 35546 non-null | Int64 |
| 2  | age_range_21 - 35                  | 35546 non-null | bool  |
| 3  | age_range_36 - 46                  | 35546 non-null | bool  |
| 4  | age_range_47 - 63                  | 35546 non-null | bool  |
| 5  | age_range_64 - 95                  | 35546 non-null | bool  |
| 6  | salary_range_0 - 3947              | 35546 non-null | bool  |
| 7  | salary_range_3948 - 5797           | 35546 non-null | bool  |
| 8  | salary_range_5798 - 7495           | 35546 non-null | bool  |
| 9  | salary_range_7496 - 150000         | 35546 non-null | bool  |
| 10 | total_loans_range_1 - 4            | 35546 non-null | bool  |
| 11 | total_loans_range_12 - 57          | 35546 non-null | bool  |
| 12 | total_loans_range_5 - 8            | 35546 non-null | bool  |
| 13 | total_loans_range_9 - 11           | 35546 non-null | bool  |
| 14 | more90_range_0                     | 35546 non-null | bool  |
| 15 | more90_range_1-3                   | 35546 non-null | bool  |
| 16 | more90_range_4-6                   | 35546 non-null | bool  |
| 17 | more90_range_7+                    | 35546 non-null | bool  |
| 18 | debt_ratio_range_0 - 0.18          | 35546 non-null | bool  |
| 19 | debt_ratio_range_0.19 - 0.37       | 35546 non-null | bool  |
| 20 | debt_ratio_range_0.38 - 0.88       | 35546 non-null | bool  |
| 21 | debt_ratio_range_0.89 - 36705      | 35546 non-null | bool  |
| 22 | using_lines_range_0 - 0.14         | 35546 non-null | bool  |
| 23 | using_lines_range_0.15 - 0.6       | 35546 non-null | bool  |
| 24 | using_lines_range_0.7 - 0.9        | 35546 non-null | bool  |
| 25 | using_lines_range_1 - 8710         | 35546 non-null | bool  |

dtypes: Int64(2), bool(24)
Nota: Se utilizaron todas las variables, pero en el modelo se excluyó default_flag de las variables independientes para asegurar que la predicción tuviera sentido

## Modelo de Regresión Logística para Predecir default_flag

Con los datos preparados, construyo el primer modelo de regresión logística para predecir la variable default_flag. Luego, genero la matriz de confusión, que facilita la interpretación de los resultados del modelo al mostrar el número de predicciones correctas e incorrectas en cada categoría, proporcionando una visión clara de la precisión y efectividad del modelo en clasificar a los clientes.


![Captura de pantalla 2024-08-12 183218](https://github.com/user-attachments/assets/29b8c4c0-1e1f-4c3c-91e4-8c4edc0d6cf0)


El análisis muestra que el modelo de regresión logística tiene un desempeño desigual entre las dos clases del objetivo (default_flag). Mientras que el modelo alcanza una alta precisión del 99% en identificar a clientes que no han incumplido, su precisión en identificar a los clientes que sí han incumplido es mucho menor, con un 61%. Esto resulta en un F1-Score bajo para la clase de incumplimiento, lo que indica un desafío significativo en la identificación de estos clientes.

Este desequilibrio en el desempeño del modelo puede explicarse por la distribución desigual en los datos: hay muchos más clientes que no han incumplido, lo que hace más fácil predecir esta clase. El modelo recupera solo el 40% de los clientes incumplidores, lo que es un indicativo de que está fallando en detectar adecuadamente a los clientes en riesgo.

Además, visualizamos los coeficientes del modelo mediante un grafico de barras horizontales que muestra la magnitud de los coeficientes de cada característica. Estos coeficientes representan el impacto de cada característica en la probabilidad de que ocurra el evento de interés, en este caso, el incumplimiento de pago


![Captura de pantalla 2024-08-12 192551](https://github.com/user-attachments/assets/2e92bb5d-a66d-4b4b-beba-4a66d167b4e8)


En el análisis de riesgo relativo, identificamos que el grupo con mayores probabilidades de ser calificado como mal pagador era aquel con ingresos del último mes entre 0 y 3947. Sin embargo, al revisar los coeficientes del modelo de regresión para default_flag, encontramos que este segmento de ingresos no tiene un peso relevante, a pesar de su alto riesgo relativo. En contraste, el segmento de ingresos entre 5798 y 7495 muestra un coeficiente positivo considerable, lo que indica que este rango tiene una mayor influencia en la probabilidad de incumplimiento cuando se evalúan todas las variables en conjunto

