# RIESGO RELATIVO
Para calcular el riesgo relativo de buenos y malos pagadores en función de la variable default_flag, utilizaremos el siguiente código en Python. Este código nos permitirá contar y visualizar los usuarios en cada rango de edad según el valor de default_flag (1 y 0) y calcular el riesgo relativo para cada segmento.

``` python 
def calculate_riesgo_relativo(df, group_column):
    # Crear una tabla de resumen para contar usuarios con default_flag = 0 y = 1 por grupo
    summary = df.groupby([group_column, 'default_flag']).size().unstack(fill_value=0)

    # Renombrar columnas para claridad
    summary.columns = ['Count_default_0', 'Count_default_1']

    # Calcular la tasa de incidencia (proporción de default_flag = 1) por grupo
    summary['Total'] = summary['Count_default_0'] + summary['Count_default_1']
    summary['Tasa_incidencia'] = summary['Count_default_1'] / summary['Total']

    # Calcular la tasa de incidencia global
    total_default_1 = df['default_flag'].sum()
    total_users = len(df)
    overall_incidence_rate = total_default_1 / total_users

    # Calcular la tasa de incidencia en los demás grupos no expuestos
    summary['Total_no_expuesto'] = total_users - summary['Total']
    summary['Total_default_1_no_expuesto'] = total_default_1 - summary['Count_default_1']
    summary['Tasa_incidencia_no_expuesto'] = summary['Total_default_1_no_expuesto'] / summary['Total_no_expuesto']

    # Calcular el riesgo relativo para cada grupo
    summary['Riesgo_Relativo'] = summary['Tasa_incidencia'] / summary['Tasa_incidencia_no_expuesto']

    # Ordenar por la columna del grupo en orden ascendente
    summary = summary.sort_index()

    return summary

# Calcular riesgo relativo para cada variable
result_age_range = calculate_riesgo_relativo(df_consolidado, 'age_range')
result_total_loans_range = calculate_riesgo_relativo(df_consolidado, 'total_loans_range')
result_salary_range = calculate_riesgo_relativo(df_consolidado, 'salary_range')
result_more90_range = calculate_riesgo_relativo(df_consolidado, 'more90_range')

# Mostrar resultados
print("Riesgo Relativo por Rango de Edad:")
print(result_age_range.reset_index())
print("\nRiesgo Relativo por Rango de Total de Préstamos:")
print(result_total_loans_range.reset_index())
print("\nRiesgo Relativo por Rango de Salario:")
print(result_salary_range.reset_index())
print("\nRiesgo Relativo por Rango de More90:")
print(result_more90_range.reset_index())
```
![image](https://github.com/user-attachments/assets/9db0c7fa-637b-4eb7-82b5-07e81c4ec0bf)

Con base en los resultados del riesgo relativo para cada variable, crearemos variables tipo bandera en la vista `tabla_consolidado`. Estas variables se marcarán con un valor de 1 para los rangos que muestren un riesgo relativo superior a 1.05, indicando una mayor probabilidad de ser clasificados como morosos según default_flag. Esto nos permitirá identificar de manera más precisa los segmentos con mayor exposición al riesgo de incumplimiento.

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
```

![image](https://github.com/user-attachments/assets/2bf5fb20-bc33-4576-a55d-ff899a153b10)

## Validar Hipótesis
En los grupos encontrados, validar la hipótesis de cuáles tienen un riesgo relativo distinto.

### 1. Los más jóvenes tienen un mayor riesgo de impago.
Los cálculos del riesgo relativo indican que los grupos con mayor exposición a impago se encuentran entre los 21 y 52 años, que corresponden a los dos primeros cuartiles de edad. Por lo tanto, esta hipótesis parece ser correcta, ya que los datos sugieren que los jóvenes tienen un mayor riesgo de ser clasificados como morosos.

### 2. Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores.
Esta hipótesis se rechaza, ya que el análisis del riesgo relativo muestra que los grupos con mayor exposición al riesgo de ser calificados como morosos se encuentran en el primer y segundo cuartil, los cuales tienen una cantidad de préstamos menor en comparación con los demás cuartiles. Esto sugiere que, contrariamente a lo esperado, una mayor cantidad de préstamos activos no se correlaciona con un mayor riesgo de impago.

### 3. Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores
Observamos que el riesgo relativo para las personas que han retrasado sus pagos por más de 90 días, aunque solo haya sido una vez, es significativamente mayor en comparación con aquellos clientes que no han tenido retrasos de más de 90 días. Esto indica que incluso un único retraso prolongado en los pagos está asociado con un riesgo considerablemente mayor de ser clasificado como mal pagador.


[Hito 2](https://github.com/Maria-Data-Analyst/riesgo_relativo/tree/Consultas-Query/Hito2)
