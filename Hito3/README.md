# REGRESIÓN LOGÍSTICA

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


En el análisis de riesgo relativo, identificamos que el grupo con mayores probabilidades de ser calificado como mal pagador eran aquellos con ingresos del último mes entre 0 y 3947. Este segmento de salario tuvo un peso significativo en la clasificación de malos pagadores.

No obstante, al revisar los coeficientes del modelo de regresión para default_flag, encontramos que este segmento de ingresos no tiene un peso relevante, a pesar de su alto riesgo relativo. En contraste, el segmento de ingresos entre 5798 y 7495 muestra un coeficiente positivo considerable, indicando que este rango tiene una mayor influencia en la probabilidad de incumplimiento según el modelo.

Esta discrepancia puede explicar en parte por qué el modelo de regresión no presenta buenas métricas en la identificación de malos pagadores. La falta de consideración de variables importantes, como los ingresos más bajos que resultaron críticos en el análisis de riesgo relativo, podría estar limitando la capacidad del modelo para predecir con precisión el incumplimiento de pagos


## Modelo de Regresión Logística para Predecir malos_pagadores

La variable de clasificación creada a partir del score crediticio se denomina malos_pagadores. Para construir el modelo de regresión logística, aplicaremos el mismo proceso que utilizamos para la variable default_flag. Esto incluye seleccionar las variables independientes relevantes y establecer malos_pagadores como la variable dependiente. De esta manera, podremos evaluar el desempeño del modelo en la predicción de clientes clasificados como malos pagadores mediante la matriz de confusión

![Captura de pantalla 2024-08-12 194540](https://github.com/user-attachments/assets/8b59e55c-d58c-43bb-abb0-6ecce49d6ddf)

La matriz de confusión muestra que el modelo ha clasificado todas las instancias correctamente, sin errores de clasificación en ninguno de los grupos. Esto indica que el modelo ha logrado una clasificación precisa en el conjunto de datos de prueba. Sin embargo, para asegurar que el modelo no esté simplemente memorizando los datos de entrenamiento y que pueda generalizar a nuevos datos, es crucial evaluarlo en un conjunto de datos completamente independiente que no se haya utilizado en el entrenamiento ni en la validación. Esta evaluación es esencial para confirmar la capacidad de generalización del modelo y su efectividad en condiciones reales

![Captura de pantalla 2024-08-12 195517](https://github.com/user-attachments/assets/2d915f0c-fbc9-42c3-824f-7d7ebdaf4d41)

Al examinar los coeficientes de la regresión lineal, se observa que reflejan adecuadamente la importancia de las características en la construcción del score de riesgo. Los segmentos con los coeficientes más altos corresponden a las características que tienen una mayor influencia en la clasificación de un mal pagador. En particular, los segmentos de ingresos bajos (salary_range_0 - 3947), la cantidad de retrasos de más de 90 días y ciertos rangos de uso de líneas de crédito (using_lines_range_0.7 - 0.9, using_lines_range_1 - 8710) tienen los coeficientes más altos, lo que indica su fuerte impacto en el riesgo de incumplimiento. 
