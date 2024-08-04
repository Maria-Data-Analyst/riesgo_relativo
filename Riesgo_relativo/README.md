# RIESGO RELATIVO
Para calcular el riesgo relativo de buenos y malos pagadores en función de la variable default_flag, utilizaremos el siguiente código en Python. Este código nos permitirá contar y visualizar los usuarios en cada rango de edad según el valor de default_flag (1 y 0) y calcular el riesgo relativo para cada segmento.

``` python 
pd.set_option('display.max_columns', None)  # Muestra todas las columnas
pd.set_option('display.expand_frame_repr', False)  # Evita el salto de línea en la visualización

# Crear una tabla de resumen para contar usuarios con default_flag = 0 y = 1 por age_range
summary = df_consolidado.groupby(['age_range', 'default_flag']).size().unstack(fill_value=0)

# Renombrar columnas para claridad
summary.columns = ['Count_default_0', 'Count_default_1']

# Calcular la tasa de incidencia (proporción de default_flag = 1) por grupo
summary['Total'] = summary['Count_default_0'] + summary['Count_default_1']
summary['Tasa_incidencia'] = summary['Count_default_1'] / summary['Total']

# Calcular la tasa de incidencia global para los grupos no expuestos
total_default_1 = df_consolidado['default_flag'].sum()
total_users = len(df_consolidado)
overall_incidence_rate = total_default_1 / total_users

# Calcular el riesgo relativo para cada grupo
summary['Incidencia_no_expuesto'] = (total_default_1 - summary['Count_default_1']) / (total_users - summary['Total'])
summary['Riesgo_Relativo'] = summary['Tasa_incidencia'] / summary['Incidencia_no_expuesto']

print(summary.reset_index())
```
![image](https://github.com/user-attachments/assets/36463a57-a729-49d3-87d0-721f31deebe6)

