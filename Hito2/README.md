# SCORE DE RIESGO


``` sql
-- Cálculo de puntos
    (CASE 
     WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.7 AND loans_detail.using_lines_not_secured_personal_assets < 1 THEN 1
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 1 AND loans_detail.using_lines_not_secured_personal_assets <= 8710 THEN 1
      ELSE 0
    END +
    CASE
      WHEN user_default.last_month_salary >= 0 AND user_default.last_month_salary <= 3947 THEN 1
      ELSE 0
    END +
    CASE
      WHEN loans_detail.more_90_days_overdue = 0 THEN 0
      ELSE 1
    END * 1) AS puntos,

    -- Determinación de malos pagadores
    CASE
      WHEN (
            CASE 
              WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.7 AND loans_detail.using_lines_not_secured_personal_assets < 1 THEN 1
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 1 AND loans_detail.using_lines_not_secured_personal_assets <= 8710 THEN 1
              ELSE 0
            END  +
            CASE
              WHEN user_default.last_month_salary >= 0 AND user_default.last_month_salary <= 3947 THEN 1
              ELSE 0
            END  +
            CASE
              WHEN loans_detail.more_90_days_overdue = 0 THEN 0
              ELSE 1
            END ) >= 1 THEN 1
      ELSE 0
    END AS malos_pagadores

```


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


[HITO 3](https://github.com/Maria-Data-Analyst/riesgo_relativo/blob/Consultas-Query/Hito3/README.md)
