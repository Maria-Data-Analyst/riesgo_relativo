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

[Análisis Exploratorio]()
