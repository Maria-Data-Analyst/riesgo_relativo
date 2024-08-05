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
![image](https://github.com/user-attachments/assets/932c811c-836c-4177-bb20-e049f9fbb124)


![image](https://github.com/user-attachments/assets/9bdd737f-7a05-4162-a263-b23c589c641f)





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
Los cálculos del riesgo relativo indican que los grupos con mayor exposición a impago se encuentran entre los 21 y 52 años, que corresponden a los dos primeros cuartiles de edad. Por lo tanto, esta hipótesis parece ser correcta, ya que los datos sugieren que los jóvenes tienen un mayor riesgo de ser clasificados como morosos.

### 2. Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores.
Esta hipótesis se rechaza, ya que el análisis del riesgo relativo muestra que los grupos con mayor exposición al riesgo de ser calificados como morosos se encuentran en el primer y segundo cuartil, los cuales tienen una cantidad de préstamos menor en comparación con los demás cuartiles. Esto sugiere que, contrariamente a lo esperado, una mayor cantidad de préstamos activos no se correlaciona con un mayor riesgo de impago.

### 3. Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores
Observamos que el riesgo relativo para las personas que han retrasado sus pagos por más de 90 días, aunque solo haya sido una vez, es significativamente mayor en comparación con aquellos clientes que no han tenido retrasos de más de 90 días. Esto indica que incluso un único retraso prolongado en los pagos está asociado con un riesgo considerablemente mayor de ser clasificado como mal pagador.


[Hito 2](https://github.com/Maria-Data-Analyst/riesgo_relativo/tree/Consultas-Query/Hito2)
