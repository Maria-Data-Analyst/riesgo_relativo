# Procesamiento y Preparación de la Base de Datos

El primer paso que realicé fue unir la tabla default con la tabla user_info para analizar los datos de manera más integral. Durante la unión, **convertí** el tipo de dato de user_id a **string**, ya que es un dato de identificación y no se realizan operaciones matemáticas con él. Además, excluí la variable sex, ya que no aporta al cálculo del riesgo relativo y podría introducir sesgo de género

A continuación, realicé una consulta para **identificar valores nulos** en todas las variables. Encontramos 7,199 valores nulos en la variable last_month_salary y 943 en la variable number_dependents. Dado que la cantidad de nulos en last_month_salary supera el 20% de la base de datos, decidí imputar estos valores utilizando la media. Para esto, separé a los buenos de los malos pagadores y elaboré un boxplot para cada grupo a fin de identificar posibles valores atípicos que pudieran afectar el cálculo de la media. Una vez identificados, excluí momentáneamente los valores atípicos para calcular la media de los datos restantes, y utilicé este valor para imputar los datos nulos. Posteriormente, reintegré todos los datos en el análisis.

En cuanto a la variable number_dependents, observé que la moda era 0 hijos tanto para los buenos como para los malos pagadores. Por lo tanto, decidí imputar los valores nulos con la moda, que en este caso es 0.

Luego, realicé una consulta para **validar los datos duplicados** y descubrí que el user_id en la tabla loans_outstanding se repite para cada préstamo asociado a un usuario. Esto resulta en múltiples entradas para el mismo user_id y genera duplicados aparentes.

Para abordar este problema, decidí separar los tipos de préstamos y visualizarlos de manera totalizada para cada usuario. Además, **creé una variable llamada total_loans** para sumar la cantidad de préstamos por cada tipo, que en este caso son other_loans y real_estate_loans. 

Para obtener variables separadas para cada tipo de préstamo, primero **estandaricé** la forma en que se escribían los tipos de préstamos, ya que la falta de uniformidad en la escritura impedía un conteo preciso. Esta armonización permitió realizar un conteo exacto de cada variable. Por lo tanto, creé tres nuevas variables:

*  **other_loans:** Para reflejar la cantidad de préstamos que no son de bienes raíces.
*  **real_estate_loans:** Para mostrar la cantidad de préstamos relacionados con bienes raíces.
*  **total_loans:** Para mostrar la cantidad total de préstamos por usuario.
  
Estos cambios reemplazaron la variable loan_type original y eliminaron la columna loan_id, que no es necesaria para los procedimientos futuros. Con estas modificaciones, logramos que los user_id no se repitieran, obteniendo una tabla sin duplicados.

Para la **selección de variables**, visualicé la correlación de las variables mediante una matriz para cada tabla. En la **matriz de correlación** de la tabla loans_outstanding, identifiqué una alta correlación entre other_loans y total_loans. Para evitar problemas de multicolinealidad, decidí no incluir la variable other_loans al crear la tabla general.

Además, observé altas correlaciones entre las variables de retraso en la tabla loans_detail. Para determinar cuál de estas variables tenía mayor variación, analicé la desviación estándar de cada una, obteniendo los siguientes valores:

![image](https://github.com/user-attachments/assets/959cc608-6152-4419-9ac7-f8957229a78b)

Dado que las desviaciones estándar son similares y considerando que en el futuro se validará una hipótesis utilizando la variable more_90_days_overdue, decidí conservar esta variable en la tabla general y en los cálculos posteriores.

Un paso crucial es **identificar y manejar los datos atípicos**, ya que estos pueden distorsionar las métricas y llevar a resultados erróneos. Para abordar este problema, utilicé diagramas de caja interactivos para cada variable. Esta visualización me permitió analizar la distribución de los datos y detectar valores atípicos, lo que facilitó la toma de las siguientes decisiones:

* **last_month_salary:** Identifiqué valores atípicos a partir de 428,000 dólares, que correspondían a 5 registros en la base de datos. Estos registros estaban clasificados como buenos pagadores, por lo que su eliminación no afectaría negativamente los análisis.
*  **age:**  Se detectaron valores atípicos en edades superiores a 96 años, con un total de 10 registros. Como estos registros también pertenecían a buenos pagadores, se optó por eliminarlos.
*   **debt_ratio:**  Se encontraron valores extremos iguales o superiores a 49.112 en 3 registros. Estos registros fueron eliminados para preservar la calidad y consistencia de los datos.
*    **more_90_days_overdue:**  Identificamos valores atípicos en el rango de 96 a 98. Se encontraron 63 registros con estos valores extremos, todos clasificados como buenos pagadores. Para mantener la calidad del análisis, estos registros fueron eliminados
  
La exclusión de datos atípicos se realizó mediante sentencias en la vista de cada tabla.

El último paso del procesamiento consistió en unir las vistas de las tablas que habían sido modificadas y contenían datos limpios. La unión se realizó mediante un **INNER JOIN** para asegurar que solo se incluyeran los usuarios con información completa en todas las variables de las distintas tablas. Esta operación resultó en una **tabla consolidada con 35,546 usuarios, de los cuales 621 estaban clasificados como clientes morosos, representando solo el 1.75%**

# ANÁLISIS EXPLORATORIO 

Comenzaremos visualizando histogramas para cada variable de la tabla para entender mejor la distribución de los datos. Estos histogramas nos permitirán identificar patrones, tendencias y posibles anomalías en cada variable, facilitando así una exploración más profunda de los datos

![image](https://github.com/user-attachments/assets/851a79bd-37a0-4efe-842c-8e721f532fc6)

El histograma de la edad muestra una distribución que, en general, no presenta un sesgo evidente. Sin embargo, he observado fluctuaciones en la cantidad de usuarios en distintos rangos de edad. Al analizar otros histogramas, he notado que varias variables presentan un sesgo positivo hacia la derecha, indicando la presencia de valores extremadamente altos que afectan la distribución de los datos. En el análisis previo, los boxplots demostraron cómo estos valores elevados influían en la visualización (ver el apartado de procesamiento y limpieza de datos). Dado que no fue posible eliminarlos debido a su importancia en la base de datos, he decidido segmentar las variables. Esta estrategia permitirá un análisis exploratorio más detallado y ayudará a gestionar mejor los valores elevados.

Dado lo anterior, inicié la segmentación utilizando cuartiles. No obstante, para lograr una visualización y segmentación más precisa en algunas variables opté por una segmentación manual. Esto me permitió ajustar las categorías de manera más efectiva y obtener una visión más clara. Así, hemos definido los siguientes segmentos:

* **age**
  
    * 21 - 42
    * 43 - 52
    * 53 - 63
    * 64 - 95
      
* **last_month_salary**
  
    * 0 - 3947
    * 3948 - 5797
    * 5798 - 7495
    * 7496 - 150000
  
* **total_loans**
  
    * 1 - 4
    * 5 - 8
    * 9 - 11
    * 12 - 57

* **more_90_days_overdue**
  
    * 0
    * 1 - 3
    * 4 - 6
    * 7+

* **debt_ratio**
  
    * 0 - 0.18
    * 0.19 - 0.37
    * 0.38 - 0.88
    * 0.89 - 36705

* **using_lines_not_secured_personal_assets**
  
    * 0 - 0.14
    * 0.15 - 0.6
    * 0.7 - 0.9
    * 1 - 8710

Seguimos con las medidas de tendencia central 

![image](https://github.com/user-attachments/assets/34114893-29b5-4146-b960-7ce19dd3dc7c)

![image](https://github.com/user-attachments/assets/7dd7f4be-5d1b-441a-8786-275880c4edac)

Estas medidas nos permiten evaluar la eficacia de la segmentación. Para las variables age, total_loans y last_month_salary, la mediana y el promedio son bastante cercanos, lo que sugiere una distribución relativamente uniforme. Sin embargo, en debt_ratio y using_lines_not_secured_personal_assets, el último cuartil presenta un sesgo positivo hacia la derecha, ya que el promedio es significativamente mayor que la mediana. Esto indica que estos segmentos están afectados por valores extremos elevados, que mantienen el sesgo positivo.

A pesar de este sesgo, decidí no modificar el último cuartil en estas variables debido a que debt_ratio y using_lines_not_secured_personal_assets son indicadores clave del nivel de endeudamiento. Valores superiores a 1 en estas variables ya representan una preocupación significativa en sí mismos. Por lo tanto, mantener el segmento tal como está nos ayuda a identificar y gestionar adecuadamente los casos con mayor riesgo de endeudamiento. Además, el último segmento en todas las variables muestra la mayor desviación estándar, lo que indica una mayor variación de datos respecto a la media.
 
