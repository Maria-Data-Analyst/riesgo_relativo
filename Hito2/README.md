# SCORE DE RIESGO
Para desarrollar el score de riesgo, utilizaremos las variables de bandera del riesgo relativo, ya que estas nos indican qué rangos están más expuestos a ser clasificados como malos pagadores. Asignaremos un peso a cada variable y estableceremos un umbral para distinguir entre buenos y malos pagadores. Este enfoque nos permitirá identificar de manera más precisa el riesgo asociado a cada cliente.


### Edad
La edad es un factor crucial en el análisis crediticio porque está vinculada a la estabilidad financiera y la experiencia en la gestión de crédito. Según el riesgo relativo, los grupos de edad más jóvenes muestran una mayor propensión a ser clasificados como malos pagadores. Por esta razón, la variable edad se valora con 2 puntos en el score.

### Salario
El rango de salario con un riesgo relativo mayor a 1.05 es de 0 a 3947, lo que indica una alta exposición a la morosidad. Así, asignaremos a esta variable 3 puntos en el score para reflejar su alto impacto en la capacidad de pago.

### Total de Préstamos
El análisis del riesgo relativo muestra que el riesgo no aumenta con un mayor número de préstamos; en cambio, se concentra en los dos primeros cuartiles, que tienen menos préstamos. Por lo tanto, asignaremos un peso bajo de 2 puntos a esta variable en el score.

### Retrasos de Pago por Más de 90 Días (more90_days)
Las personas que han tenido retrasos en sus pagos por más de 90 días, aunque sea una sola vez, tienen un riesgo relativo extremadamente alto (192.000). Esto demuestra una fuerte relación entre los retrasos prolongados y la morosidad. Por esta razón, asignaremos el máximo de 3 puntos a esta variable en el score.

## Umbral
El umbral para clasificar a un cliente como mal pagador se fija en 3 puntos. Esto implica que un cliente podría ser considerado para un préstamo si solo tiene un riesgo alto en la variable de edad o en total_loans, siempre que las demás variables muestren resultados favorables. Sin embargo, si un cliente presenta un alto riesgo en más de una variable, o si su único riesgo alto proviene de las variables de salario o días de retraso, alcanzará el umbral de 3 puntos. En estos casos, el cliente será clasificado como de alto riesgo y se le negará el préstamo.

Con esta definición, se creará una variable denominada puntos y una variable de bandera llamada mal_pagador. La variable mal_pagador tomará el valor de 1 cuando el puntaje sea igual o superior a 3, indicando que el usuario es clasificado como mal pagador.
