# SCORE DE RIESGO
Para desarrollar el score de riesgo, utilizaremos las variables de bandera del riesgo relativo, que nos proporcionan información sobre los rangos más susceptibles a ser clasificados como malos pagadores. Asignaremos un peso a cada una de estas variables y estableceremos un umbral para diferenciar entre buenos y malos pagadores. Este enfoque nos permitirá identificar con mayor precisión el riesgo asociado a cada cliente.

El score de riesgo tendrá un máximo de 13 puntos, distribuidos de la siguiente manera:

* ### Edad (age)
La edad es un factor crucial en el análisis crediticio porque está vinculada a la estabilidad financiera y la experiencia en la gestión de crédito. Según el riesgo relativo, los grupos de edad más jóvenes muestran una mayor propensión a ser clasificados como malos pagadores. Por esta razón, la variable edad se valora con 2 puntos en el score.


* ### Salario (last_month_salary)
El rango de salario con un riesgo relativo mayor a 1.05 es de 0 a 3947, lo que indica una alta exposición a la morosidad. Así, asignaremos a esta variable 3 puntos en el score para reflejar su alto impacto en la capacidad de pago.

* ### Total de Préstamos (total_loans)
El análisis del riesgo relativo muestra que el riesgo no aumenta con un mayor número de préstamos; en cambio, se concentra en los dos primeros cuartiles, que tienen menos préstamos. Por lo tanto, asignaremos un peso bajo de 2 puntos a esta variable en el score.

* ### Retrasos de Pago por Más de 90 Días (more90_days)
Las personas que han tenido retrasos en sus pagos por más de 90 días, aunque sea una sola vez, tienen un riesgo relativo extremadamente alto (192.000). Esto demuestra una fuerte relación entre los retrasos prolongados y la morosidad. Por esta razón, asignaremos el máximo de 3 puntos a esta variable en el score.

* ### Uso de líneas de crédito no garantizadas con activos personales (using_lines_not_secured_personal_assets)
Un valor de 0.7 en esta variable indica un uso elevado del crédito no garantizado, lo cual refleja una situación de riesgo significativo. Un cliente que utiliza una proporción considerable de su crédito no garantizado se encuentra en una posición de alto riesgo. Por lo tanto, esta variable será asignada con 3 puntos en nuestro sistema de evaluación

## Umbral
El umbral para clasificar a un cliente como mal pagador se fija en 3 puntos. Esto implica que un cliente podría ser considerado para un préstamo si solo tiene un riesgo alto en la variable de edad o en total_loans, siempre que las demás variables muestren resultados favorables. Sin embargo, si un cliente presenta un alto riesgo en más de una variable, o si su único riesgo alto proviene de las variables de using_lines_not_secured_personal_assets, salario o días de retraso, alcanzará el umbral de 3 puntos. En estos casos, el cliente será clasificado como de alto riesgo y se le negará el préstamo.

Para implementar esta clasificación, se creará una variable llamada puntos y una variable de bandera denominada mal_pagador. La variable mal_pagador tomará el valor de 1 cuando el puntaje sea igual o superior a 3, indicando que el cliente es clasificado como mal pagador. Para la creación de estas dos variables, se agregarán las siguientes líneas de código a la tabla consolidada con la que hemos estado trabajando:

``` sql
 -- Cálculo de puntos
    (CASE 
     WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.7 AND loans_detail.using_lines_not_secured_personal_assets < 1 THEN 1
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 1 AND loans_detail.using_lines_not_secured_personal_assets <= 8710 THEN 1
      ELSE 0
    END * 3 +
    CASE
      WHEN user_default.age >= 21 AND user_default.age <= 42 THEN 1
      WHEN user_default.age >= 43 AND user_default.age <= 52 THEN 1
      ELSE 0
    END * 2 +
    CASE
      WHEN user_default.last_month_salary >= 0 AND user_default.last_month_salary <= 3947 THEN 1
      ELSE 0
    END * 3 +
    CASE
      WHEN loans_outstanding.total_loans >= 1 AND loans_outstanding.total_loans <= 4 THEN 1
      WHEN loans_outstanding.total_loans >= 5 AND loans_outstanding.total_loans <= 8 THEN 0
      ELSE 0
    END * 2 +
    CASE
      WHEN loans_detail.more_90_days_overdue = 0 THEN 0
      ELSE 1
    END * 3) AS puntos,

    -- Determinación de malos pagadores
    CASE
      WHEN (CASE
              WHEN user_default.age >= 21 AND user_default.age <= 42 THEN 1
              WHEN user_default.age >= 43 AND user_default.age <= 52 THEN 1
              ELSE 0
            END * 2 +
            CASE 
              WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.7 AND loans_detail.using_lines_not_secured_personal_assets < 1 THEN 1
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 1 AND loans_detail.using_lines_not_secured_personal_assets <= 8710 THEN 1
              ELSE 0
            END * 3 +
            CASE
              WHEN user_default.last_month_salary >= 0 AND user_default.last_month_salary <= 3947 THEN 1
              ELSE 0
            END * 3 +
            CASE
              WHEN loans_outstanding.total_loans >= 1 AND loans_outstanding.total_loans <= 4 THEN 1
              WHEN loans_outstanding.total_loans >= 5 AND loans_outstanding.total_loans <= 8 THEN 0
              ELSE 0
            END * 2 +
            CASE
              WHEN loans_detail.more_90_days_overdue = 0 THEN 0
              ELSE 1
            END * 3) >= 3 THEN 1
      ELSE 0
    END AS malos_pagadores
```
![image](https://github.com/user-attachments/assets/dae8a5ce-1a41-4adc-b5a0-42f098c5e704)

## Matriz de Confusión
Con el puntaje asignado y la identificación de malos pagadores, podemos evaluar nuestro modelo de detección de malos pagadores creando una matriz de confusión en Google Colab. Esta matriz nos permitirá comparar nuestra clasificación con la existente en la base de datos, denominada default_flag, para validar la efectividad de nuestro modelo.
``` python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay, classification_report
import matplotlib.pyplot as plt

# Calcular la matriz de confusión
y_true = df_consolidado['default_flag']
y_pred = df_consolidado['malos_pagadores']

cm = confusion_matrix(y_true, y_pred, labels=[0, 1])

# Imprimir la matriz de confusión
print('Matriz de Confusión:')
print(cm)

# Calcular y mostrar el reporte de clasificación
report = classification_report(y_true, y_pred, labels=[0, 1], target_names=['Buen Pagador', 'Mal Pagador'])

# Crear y mostrar la matriz de confusión gráficamente
cm_display = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=['Buen Pagador', 'Mal Pagador'])
cm_display.plot(cmap=plt.cm.Blues)

# Mostrar el gráfico
plt.show()

print('Reporte de Clasificación:')
print(report)
```
![image](https://github.com/user-attachments/assets/8a188599-6ec0-4ad9-8daf-a1670b1db046)



## Análisis de Discrepancias del Modelo

El modelo ha clasificado a 14,501 usuarios como malos pagadores, mientras que `default_flag` los considera buenos pagadores. Esta discrepancia subraya la necesidad de una revisión exhaustiva para optimizar el modelo y cumplir con el objetivo del proyecto: identificar el perfil de clientes que presentan riesgo de impago y desarrollar un motor de reglas de aprobación de crédito eficaz. Aunque el modelo logra identificar casi todos los malos pagadores, solo clasifica a uno como buen pagador. Esto sugiere que el modelo podría estar aplicando algunas reglas que `default_flag` utiliza para determinar malos pagadores, y podría estar utilizando otras variables adicionales que no se han considerado en `default_flag`, o bien, que están siendo subestimadas.

### Posibles Causas de Inconsistencia

1. **Inconsistencias en los Datos**: Es fundamental revisar los criterios y datos empleados para calcular `default_flag`. El modelo ha detectado a 142 usuarios con los puntajes más altos, que se encuentran en los rangos de mayor riesgo según las cinco variables analizadas. Sin embargo, `default_flag` los clasifica como buenos pagadores. Esto indica posibles errores en la información o en la metodología utilizada para definir `default_flag`, lo que requiere una corrección para alinear el modelo con la realidad de los perfiles de riesgo.

2. **Datos Faltantes**: Podría haber información clave que `default_flag` utiliza para su clasificación y que falta en el conjunto de datos actual. La ausencia de estos datos puede estar influyendo en la discrepancia entre el modelo y las clasificaciones actuales. Identificar y añadir esta información es crucial para mejorar la precisión del modelo.

3. **Revisión de Criterios**: Es necesario revisar y, si es necesario, actualizar los criterios utilizados para definir `default_flag`. Alineando estos criterios con los rangos de riesgo identificados por el modelo, se podrán establecer reglas que reflejen de manera más precisa el perfil de riesgo de los clientes.

### Conclusión

Para mejorar la precisión del modelo y asegurar que la clasificación de malos pagadores sea coherente con los datos de `default_flag`, es esencial realizar una investigación exhaustiva sobre las fuentes de datos y los criterios de clasificación empleados en ambas metodologías. Esta revisión permitirá identificar y corregir inconsistencias, garantizando que nuestro modelo de puntuación sea confiable y eficaz en la identificación del riesgo crediticio.
