# RIESGO RELATIVO
Para calcular el riesgo relativo de buenos y malos pagadores en función de la variable default_flag, utilizaremos el siguiente código en Python. Este código nos permitirá contar y visualizar los usuarios en cada rango de edad según el valor de default_flag (1 y 0) y calcular el riesgo relativo para cada segmento.

``` python 
import pandas as pd
from IPython.display import display, HTML

def calculate_riesgo_relativo(df, group_column):
    # Crear una tabla de resumen para contar usuarios con default_flag = 0 y = 1 por grupo
    summary = df.groupby([group_column, 'default_flag']).size().unstack(fill_value=0)

    # Renombrar columnas para claridad
    summary.columns = ['default_0', 'default_1']

    # Calcular la tasa de incidencia (proporción de default_flag = 1) por grupo
    summary['Total'] = summary['default_0'] + summary['default_1']
    summary['Tasa_incidencia'] = summary['default_1'] / summary['Total']

    # Calcular la tasa de incidencia global
    total_default_1 = df['default_flag'].sum()
    total_users = len(df)
    overall_incidence_rate = total_default_1 / total_users

    # Calcular la tasa de incidencia en los demás grupos no expuestos
    summary['Total_no_expuesto'] = total_users - summary['Total']
    summary['Total_default_1_no_expuesto'] = total_default_1 - summary['default_1']
    summary['Tasa_incidencia_no_expuesto'] = summary['Total_default_1_no_expuesto'] / summary['Total_no_expuesto']

    # Calcular el riesgo relativo para cada grupo
    summary['Riesgo_Relativo'] = summary['Tasa_incidencia'] / summary['Tasa_incidencia_no_expuesto']

    return summary

# Calcular riesgo relativo para cada variable
result_age_range = calculate_riesgo_relativo(df_consolidado, 'age_range')
result_total_loans_range = calculate_riesgo_relativo(df_consolidado, 'total_loans_range')
result_salary_range = calculate_riesgo_relativo(df_consolidado, 'salary_range')
result_more90_range = calculate_riesgo_relativo(df_consolidado, 'more90_range')
result_debt_ratio_range = calculate_riesgo_relativo(df_consolidado, 'debt_ratio_range')
result_using_lines_range = calculate_riesgo_relativo(df_consolidado, 'using_lines_range')

# Configuración de visualización en Google Colab
pd.set_option('display.max_columns', None)  # Muestra todas las columnas
pd.set_option('display.max_rows', None)     # Muestra todas las filas
pd.set_option('display.width', 1000)         # Ancho máximo para mostrar datos

# Mostrar resultados
def display_full_df(df, title):
    print(f"\n{title}:")
    display(HTML(df.reset_index().to_html()))

display_full_df(result_age_range, "Riesgo Relativo por Rango de Edad")
display_full_df(result_total_loans_range, "Riesgo Relativo por Rango de Total de Préstamos")
display_full_df(result_salary_range, "Riesgo Relativo por Rango de Salario")
display_full_df(result_more90_range, "Riesgo Relativo por Rango de More90")
display_full_df(result_debt_ratio_range, "Riesgo Relativo por Rango de Debt Ratio")
display_full_df(result_using_lines_range, "Riesgo Relativo por Rango de Using Lines")
```

![Captura de pantalla 2024-08-11 162848](https://github.com/user-attachments/assets/ad6d7f5c-5cf8-48ed-9702-52aacbbb53bd)




![Captura de pantalla 2024-08-11 162731](https://github.com/user-attachments/assets/8eabf62d-59fc-4daa-bcf1-e6358c24e06c)


Con base en los resultados del análisis de riesgo relativo para cada variable, procederemos a crear variables de tipo bandera en la vista tabla_consolidado. Estas variables se marcarán con un valor de 1 para aquellos rangos que presenten un riesgo relativo superior a 1.05, lo que indica una mayor probabilidad de ser clasificados como morosos según la variable default_flag. Esta estrategia nos permitirá identificar con mayor precisión los segmentos que presentan una mayor exposición al riesgo de incumplimiento.

Sin embargo, excluiremos la variable debt_ratio del análisis, ya que observamos que el único cuartil con riesgo relativo significativo no corresponde al cuartil con los valores más altos, como era de esperar. Por lo tanto, es necesario realizar una investigación más detallada sobre los valores de esta variable para entender por qué los valores menores de debt_ratio están siendo más propensos a la morosidad en comparación con los valores altos de esta métrica.

Sentencias que se le agregaron a la tabla : 

``` sql
CASE
      WHEN user_default.age >= 21 AND user_default.age <= 42 THEN 1
      WHEN user_default.age >= 43 AND user_default.age <= 52 THEN 1
      ELSE 0
    END AS age_rr_flag,
CASE
      WHEN user_default.last_month_salary >= 0 AND user_default.last_month_salary <= 3947 THEN 1
      ELSE 0
    END AS salary_rr_flag,
CASE
    WHEN loans_outstanding.total_loans >= 1 AND loans_outstanding.total_loans <= 4 THEN 1
    WHEN loans_outstanding.total_loans >= 5 AND loans_outstanding.total_loans <= 8 THEN 0
    ELSE 0
    END AS total_loans_rr_flag,
CASE
    WHEN loans_detail.more_90_days_overdue = 0 THEN 0
    ELSE 1
    END AS more90_rr_flag,
CASE 
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.54 AND loans_detail.using_lines_not_secured_personal_assets <= 8710 THEN 1
      ELSE 0
    END AS using_lines_rr_flag,
```

![image](https://github.com/user-attachments/assets/4ca2e6b1-a444-4d1f-9941-fc3205b9ff83)

## Validar Hipótesis
En los grupos encontrados, validar la hipótesis de cuáles tienen un riesgo relativo distinto.

### 1. Los más jóvenes tienen un mayor riesgo de impago.
Los cálculos del riesgo relativo indican que los grupos con mayor exposición a impago se encuentran entre los 21 y 46 años, que corresponden a los dos primeros cuartiles de edad. Por lo tanto, esta hipótesis parece ser correcta, ya que los datos sugieren que los jóvenes tienen un mayor riesgo de ser clasificados como morosos.

### 2. Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores.
Esta hipótesis se rechaza, ya que el análisis del riesgo relativo muestra que los grupos con mayor exposición al riesgo de ser calificados como morosos se encuentran en el primer y segundo cuartil, los cuales tienen una cantidad de préstamos menor en comparación con los demás cuartiles. Esto sugiere que, contrariamente a lo esperado, una mayor cantidad de préstamos activos no se correlaciona con un mayor riesgo de impago.

### 3. Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores
Observamos que el riesgo relativo para las personas que han retrasado sus pagos por más de 90 días, aunque solo haya sido una vez, es significativamente mayor en comparación con aquellos clientes que no han tenido retrasos de más de 90 días. Esto indica que incluso un único retraso prolongado en los pagos está asociado con un riesgo considerablemente mayor de ser clasificado como mal pagador.


## Hito 2 SCORE DE RIESGO


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

## Hito 3 :REGRESIÓN LOGÍSTICA

El modelo de regresión logística es una técnica estadística que estima la probabilidad de que ocurra un evento, en este caso, que un cliente sea clasificado como "mal pagador". Este modelo utiliza una fórmula matemática para relacionar una o más variables independientes con la probabilidad de que se produzca el evento.

Para este análisis, desarrollaré dos modelos de regresión logística:
1. Uno utilizando el `default_flag` original de la base de datos.
2. Otro basado en la clasificación obtenida mediante el score calculado.

Esto permitirá evaluar y comparar la efectividad de las diferentes clasificaciones en la predicción del comportamiento de pago de los clientes.

Utilizaré Google Colab para esta parte del proyecto, donde primero visualizaré las variables de mi `tabla_consolidado`:

Data columns (total 23 columns):

| #  | Column                                   | Non-Null Count | Dtype   |
|----|------------------------------------------|----------------|---------|
| 0  | user_id                                  | 35546 non-null  | object  |
| 1  | age                                      | 35546 non-null  | Int64   |
| 2  | last_month_salary                        | 35546 non-null  | float64 |
| 3  | number_dependents                        | 35546 non-null  | Int64   |
| 4  | default_flag                             | 35546 non-null  | Int64   |
| 5  | real_estate_loans                        | 35546 non-null  | Int64   |
| 6  | total_loans                              | 35546 non-null  | Int64   |
| 7  | using_lines_not_secured_personal_assets  | 35546 non-null  | float64 |
| 8  | more_90_days_overdue                     | 35546 non-null  | Int64   |
| 9  | debt_ratio                               | 35546 non-null  | float64 |
| 10 | age_range                                | 35546 non-null  | object  |
| 11 | salary_range                             | 35546 non-null  | object  |
| 12 | total_loans_range                        | 35546 non-null  | object  |
| 13 | more90_range                             | 35546 non-null  | object  |
| 14 | debt_ratio_range                         | 35546 non-null  | object  |
| 15 | using_lines_range                        | 35546 non-null  | object  |
| 16 | age_rr_flag                              | 35546 non-null  | Int64   |
| 17 | salary_rr_flag                           | 35546 non-null  | Int64   |
| 18 | total_loans_rr_flag                      | 35546 non-null  | Int64   |
| 19 | more90_rr_flag                           | 35546 non-null  | Int64   |
| 20 | using_lines_rr_flag                      | 35546 non-null  | Int64   |
| 21 | puntos                                   | 35546 non-null  | Int64   |
| 22 | malos_pagadores                          | 35546 non-null  | Int64   |

dtypes: Int64(13), float64(3), object(7)


He decidido usar variables segmentadas (aquellas que terminan en range) en lugar de continuas para el modelo de regresión logística, ya que permiten capturar efectos no lineales, facilitan la interpretación al dividir los datos en intervalos manejables y ayudan a manejar outliers al agrupar valores extremos. Dado que la regresión logística requiere entradas numéricas, estas variables deben ser convertidas en dummy, creando columnas binarias que el modelo puede procesar de manera efectiva. Además, he optado por incluir la variable number_dependents, que no se había considerado previamente, para evaluar su impacto en la regresión

* **Eliminar las variables que no serán tomadas para el modelo de regresión** 
```python
df_s.drop(["malos_pagadores", "puntos", "using_lines_rr_flag", "more90_rr_flag",
                  "total_loans_rr_flag", "salary_rr_flag", "age_rr_flag", "debt_ratio",
                  "using_lines_not_secured_personal_assets", "more_90_days_overdue",
                  "total_loans", "real_estate_loans", "age",
                  "last_month_salary","user_id"], axis=1, inplace=True)
```
* **Cambiar tipo de variable**
```python
# Seleccionar las columnas de tipo objeto
columnas_objeto = df_s.select_dtypes(include=['object']).columns
# Convertir las columnas de tipo objeto en dummies
df_s_dummies = pd.get_dummies(df_s, columns=columnas_objeto)
# Verifica las primeras filas del DataFrame con dummies
print(df_s_dummies.info())
```


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

## Modelo de Regresión Logística para Predecir default_flag

Con los datos preparados, construyo el primer modelo de regresión logística para predecir la variable default_flag. Luego, genero la matriz de confusión, que facilita la interpretación de los resultados del modelo al mostrar el número de predicciones correctas e incorrectas en cada categoría, proporcionando una visión clara de la precisión y efectividad del modelo en clasificar a los clientes.

```python
# Importar las librerías necesarias
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import confusion_matrix, classification_report, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

# Separa las variables independientes (X) de la variable dependiente (y) para el modelo de regresión logística.
# X contiene todas las variables explicativas excepto 'default_flag', que es la variable objetivo.
# y es la variable dependiente que queremos predecir (si un cliente es un 'mal pagador').

X = df_s_dummies.drop(["default_flag"], axis=1)
y = df_s_dummies["default_flag"]

# Divide el conjunto de datos en conjuntos de entrenamiento y prueba.
# 'test_size=0.2' indica que el 20% de los datos se usarán para prueba y el 80% para entrenamiento.
# 'stratify=y' asegura que la proporción de las clases en el conjunto de entrenamiento y prueba
# se mantenga igual a la del conjunto original, preservando la distribución de la variable objetivo.
# 'random_state=42' se establece para garantizar la reproducibilidad de la división.
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

# Crear el modelo de regresión logística
model = LogisticRegression()

# Entrenar el modelo con los datos de entrenamiento
model.fit(X_train, y_train)

# Hacer predicciones con el conjunto de prueba
y_pred = model.predict(X_test)

# Calcular la matriz de confusión
conf_matrix = confusion_matrix(y_test, y_pred)
print('Confusion Matrix:')
print(conf_matrix)

# Mostrar la matriz de confusión
conf_matrix_display = ConfusionMatrixDisplay(confusion_matrix=conf_matrix, display_labels=['Buen Pagador', 'Mal Pagador'])
fig, ax = plt.subplots(figsize=(8, 6))  # Tamaño del gráfico
conf_matrix_display.plot(cmap=plt.cm.Blues, ax=ax, values_format='d')  # 'd' para formato decimal
plt.show()

# Reporte de clasificación
class_report = classification_report(y_test, y_pred)
print('Classification Report:')
print(class_report)

```

![Captura de pantalla 2024-08-12 183218](https://github.com/user-attachments/assets/29b8c4c0-1e1f-4c3c-91e4-8c4edc0d6cf0)


El análisis muestra que el modelo de regresión logística tiene un desempeño desigual entre las dos clases del objetivo (default_flag). Mientras que el modelo alcanza una alta precisión del 99% en identificar a clientes que no han incumplido, su precisión en identificar a los clientes que sí han incumplido es mucho menor, con un 61%. Esto resulta en un F1-Score bajo para la clase de incumplimiento, lo que indica un desafío significativo en la identificación de estos clientes.

Este desequilibrio en el desempeño del modelo puede explicarse por la distribución desigual en los datos: hay muchos más clientes que no han incumplido, lo que hace más fácil predecir esta clase. El modelo recupera solo el 40% de los clientes incumplidores, lo que es un indicativo de que está fallando en detectar adecuadamente a los clientes en riesgo.

Además, contamos con el siguiente código que nos permite visualizar los coeficientes del modelo mediante un grafico de barras horizontales que muestra la magnitud de los coeficientes de cada característica. Estos coeficientes representan el impacto de cada característica en la probabilidad de que ocurra el evento de interés, en este caso, el incumplimiento de pago

``` python
# Obtener los coeficientes del modelo
coeficientes = model.coef_[0]

# Crear un DataFrame para visualizar los coeficientes
features = X_train.columns
coef_df = pd.DataFrame({'Feature': features, 'Coefficient': coeficientes})
coef_df = coef_df.sort_values(by='Coefficient', ascending=False)

# Mostrar los coeficientes
print(coef_df)

# Graficar los coeficientes
plt.figure(figsize=(10, 6))
plt.barh(coef_df['Feature'], coef_df['Coefficient'], color='skyblue')
plt.xlabel('Coeficiente')
plt.title('Importancia de las Características en el Modelo')
plt.gca().invert_yaxis()
plt.show()
```

![Captura de pantalla 2024-08-12 192551](https://github.com/user-attachments/assets/2e92bb5d-a66d-4b4b-beba-4a66d167b4e8)


En el análisis de riesgo relativo, identificamos que el grupo con mayores probabilidades de ser calificado como mal pagador era aquel con ingresos del último mes entre 0 y 3947. Sin embargo, al revisar los coeficientes del modelo de regresión para default_flag, encontramos que este segmento de ingresos no tiene un peso relevante, a pesar de su alto riesgo relativo. En contraste, el segmento de ingresos entre 5798 y 7495 muestra un coeficiente positivo considerable, lo que indica que este rango tiene una mayor influencia en la probabilidad de incumplimiento cuando se evalúan todas las variables en conjunto

