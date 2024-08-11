# ANÁLISIS EXPLORATORIO
En esta sección, realizaremos un análisis exploratorio de los datos utilizando herramientas como Google Colab, BigQuery y Looker Studio. Nuestro objetivo es obtener una visión detallada y comprensible de la información contenida en nuestra tabla tabla_consolidada.

Comenzaremos visualizando histogramas para cada variable de la tabla para entender mejor la distribución de los datos. Estos histogramas nos permitirán identificar patrones, tendencias y posibles anomalías en cada variable, facilitando así una exploración más profunda de los datos
```python
df_consolidado.hist(figsize=(15,8),bins =30, edgecolor="yellow")
``` 
![image](https://github.com/user-attachments/assets/d86611c3-5540-49e9-993c-da27b1157e91)

![image](https://github.com/user-attachments/assets/5315f946-194f-4ae7-8399-1d23db8b426f)

Hemos observado que varias variables presentan un sesgo positivo hacia la derecha, indicando la presencia de valores extremadamente altos que están afectando la distribución de los datos. En el análisis previo, los boxplots revelaron cómo estos valores altos influían en la visualización, pero no se pudieron eliminar debido a que constituyen una parte significativa de la base de datos. Para manejar estos valores de manera más efectiva, hemos decidido segmentar las variables . Esta estrategia permitirá un análisis exploratorio más adecuado y facilitará el control de estos valores elevados

## Segmentación  
En el paso anterior, obtuvimos una tabla con todas las variables limpias y preparadas para su exploración. Para facilitar un análisis más detallado, segmentaremos algunas de estas variables directamente en la vista denominada tabla_consolidada. Esta segmentación  nos permitirá examinar las variables en grupos específicos, ayudándonos a identificar patrones y tendencias de manera más efectiva.

``` sql
SELECT
    user_default.user_id,
    user_default.age,
    user_default.last_month_salary,
    user_default.number_dependents,
    user_default.default_flag,
    loans_outstanding.real_estate_loans,
    loans_outstanding.total_loans,
    loans_detail.using_lines_not_secured_personal_assets,
    loans_detail.more_90_days_overdue,
    loans_detail.debt_ratio,

    -- Rango de edad
    CASE
      WHEN user_default.age >= 21 AND user_default.age <= 35 THEN '21 - 35'
      WHEN user_default.age >= 36 AND user_default.age <= 46 THEN '36 - 46'
      WHEN user_default.age >= 47 AND user_default.age <= 63 THEN '47 - 63'
      WHEN user_default.age >= 64 AND user_default.age <= 95 THEN '64 - 95'
      ELSE 'Fuera de rango'
    END AS age_range,

    -- Rango de salario
    CASE
      WHEN user_default.last_month_salary >= 0 AND user_default.last_month_salary <= 3947 THEN '0 - 3947'
      WHEN user_default.last_month_salary >= 3948 AND user_default.last_month_salary <= 5797 THEN '3948 - 5797'
      WHEN user_default.last_month_salary >= 5798 AND user_default.last_month_salary <= 7495 THEN '5798 - 7495'
      WHEN user_default.last_month_salary >= 7496 AND user_default.last_month_salary <= 150000 THEN '7496 - 150000'
      ELSE 'Fuera de rango'
    END AS salary_range,

    -- Rango de total de préstamos
    CASE
      WHEN loans_outstanding.total_loans >= 1 AND loans_outstanding.total_loans <= 4 THEN '1 - 4'
      WHEN loans_outstanding.total_loans >= 5 AND loans_outstanding.total_loans <= 8 THEN '5 - 8'
      WHEN loans_outstanding.total_loans >= 9 AND loans_outstanding.total_loans <= 11 THEN '9 - 11'
      WHEN loans_outstanding.total_loans >= 12 AND loans_outstanding.total_loans <= 57 THEN '12 - 57'
      ELSE 'Fuera de rango'
    END AS total_loans_range,

    -- Rango de mora de más de 90 días
    CASE
      WHEN loans_detail.more_90_days_overdue = 0 THEN '0'
      WHEN loans_detail.more_90_days_overdue >= 1 AND loans_detail.more_90_days_overdue <= 3   THEN '1-3'
      WHEN loans_detail.more_90_days_overdue >= 4 AND loans_detail.more_90_days_overdue <= 6   THEN '4-6'
      WHEN loans_detail.more_90_days_overdue >=  7 THEN '7+'
      ELSE 'fuera de rango'
    END AS more90_range,

    -- Rango de deuda
    CASE 
      WHEN loans_detail.debt_ratio >= 0 AND loans_detail.debt_ratio < 0.19 THEN '0 - 0.18'
      WHEN loans_detail.debt_ratio >= 0.19 AND loans_detail.debt_ratio < 0.38 THEN '0.19 - 0.37'
      WHEN loans_detail.debt_ratio >= 0.38 AND loans_detail.debt_ratio < 0.89 THEN '0.38 - 0.88'
      WHEN loans_detail.debt_ratio >= 0.89 AND loans_detail.debt_ratio < 36706 THEN '0.89 - 36705'
      ELSE 'Fuera de rango'
    END AS debt_ratio_range,

    -- Rango de uso de líneas de crédito no aseguradas
    CASE 
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 0 AND loans_detail.using_lines_not_secured_personal_assets < 0.15 THEN '0 - 0.14'
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.15 AND loans_detail.using_lines_not_secured_personal_assets < 0.7 THEN '0.15 - 0.6'
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.7 AND loans_detail.using_lines_not_secured_personal_assets < 1 THEN '0.7 - 0.9'
      WHEN loans_detail.using_lines_not_secured_personal_assets >= 1 AND loans_detail.using_lines_not_secured_personal_assets <= 8710 THEN '1 - 8710'
      ELSE 'Fuera de rango'
    END AS using_lines_range,


FROM 
    `riesgo-relativo-1.dataset.loans_detail_clean` AS loans_detail
INNER JOIN
    `riesgo-relativo-1.dataset.loans_outstanding_clean` AS loans_outstanding
ON
    loans_detail.user_id = loans_outstanding.user_id
INNER JOIN
    `riesgo-relativo-1.dataset.user_info_default` AS user_default
ON
    user_default.user_id = loans_outstanding.user_id;
```
## Medidas de tendencia central y desviación estándar
Vamos a analizar las medias y desviaciones estándar de las variables para evaluar la segmentación de datos realizada. Este análisis nos ayudará a comprender mejor la distribución de los datos y a validar la precisión de nuestra segmentación.

![Captura de pantalla 2024-08-11 162234](https://github.com/user-attachments/assets/81e440c3-cfb7-4916-83f1-23aa6e145bb1)


![Captura de pantalla 2024-08-11 161949](https://github.com/user-attachments/assets/01437b58-0552-4346-a56f-71a59ddb4179)



Observamos que la segmentación de las variables more_90_days, last_month_salary, age y total_loans muestra valores de mediana y promedio cercanos entre sí, lo que indica que la segmentación ha sido adecuada. Sin embargo, para las variables , debt_ratio, y using_lines_not_secured_personal_assets, el último cuartil aún presenta un sesgo positivo hacia la derecha, ya que el promedio es mayor que la mediana. Esto sugiere que estos datos están influidos por valores extremos elevados en estas variables. Tambien podemos observar que la mayoria de los datos de la variable more_90_days  estan en el segmento de 0 veces, ese segmento constitute el 95% de la base de datos.
La mayor desviación estándar en el último cuartil indica que los valores en este segmento son mucho más variados. Esto podría ser el resultado de la presencia de valores atípicos o extremos que están influyendo significativamente en la variabilidad de los datos

## Asociación y visualización de variables categóricas  
Para este paso, conectaremos la tabla_consolidada en Looker Studio, lo que nos permitirá crear tablas y gráficos necesarios para el análisis. Además, formulamos preguntas específicas para orientar nuestra exploración de datos, de modo que podamos responderlas con la información disponible.

Para responder a las siguientes preguntas, hemos aplicado un filtro a la variable default_flag. Este filtro nos permite observar con mayor claridad el comportamiento de los usuarios en relación con los incumplimientos de pago.

### 1.¿Cuál es el rango de edad predominante entre los usuarios, y cuáles son los grupos etarios que solicitan más préstamos?


![image](https://github.com/user-attachments/assets/2672bdb7-6ade-4ea4-b34a-9b300ab81c90)


El gráfico está filtrado para mostrar únicamente a los clientes catalogados como morosos. En él, el rango etario con la mayor cantidad de usuarios es el de 47 a 63 años, y también es el grupo con el mayor número de préstamos. Este gráfico sugiere, como primer indicio, que la hipótesis de que los más jóvenes tienen un mayor riesgo de impago podría ser incorrecta, ya que el número de usuarios en el rango de edad más joven es menor. Sin embargo, esta observación no proporciona una base sólida, ya que es necesario comparar la cantidad de clientes en cada segmento. Este análisis más detallado se realizará mediante el cálculo del riesgo relativo en etapas posteriores.

### 2. ¿En qué rango de salario se encuentran los usuarios con mayor cantidad de préstamos?

![image](https://github.com/user-attachments/assets/dd333599-8560-4b4e-8f89-2298b55a712a)

Con el fitro de default_flag=1, la mayoría de los usuarios se encuentran en el rango de salario de 0 a 3.947 dólares, que es el segundo rango con la mayor cantidad de préstamos, solo superado por el rango de 3.498 a 5.797 dólares.



### 3. ¿Qué rango de salario corresponde a los usuarios que han caído en mora con mayor frecuencia?

![image](https://github.com/user-attachments/assets/7e8cd86f-b0b4-4877-9ee5-6bc25079bfd4)


Con el fitro de default_flag=1 , obervamos que el rango salarial que ha acumulado más veces en mora  es el de 0 a 3.947


## BOXPLOT
### age
![image](https://github.com/user-attachments/assets/949b7701-f9a8-42e9-88a1-57b9cec93d80)

El análisis del boxplot para la variable age muestra que la mediana en el cuartil de 43 a 62 años está desplazada hacia un extremo, lo que indica una posible asimetría en la distribución de edades dentro de ese rango. En comparación con otros cuartiles donde la mediana está centrada, esta desviación sugiere que la distribución de edades en el cuartil de 43 a 62 años podría estar sesgada, con una mayor concentración de datos en un extremo del rango. Esto puede reflejar una variabilidad diferente o una concentración de datos en ese segmento de edad.

### total_loans

![image](https://github.com/user-attachments/assets/4d8e7f95-80b9-4257-83f2-fbdecd9f89aa)


Cuartil 1 a 4: En este rango, la mediana está centrada dentro de la caja, indicando una distribución relativamente simétrica. Los bigotes muestran una línea corta desde el mínimo hasta el primer cuartil, mientras que no se observa línea alguna desde el tercer cuartil hasta el máximo, lo que sugiere que los datos en el extremo superior están agrupados cerca del tercer cuartil, con poca variabilidad en ese rango.

Cuartil 12 a 57: La mediana está centrada en la caja, similar al cuartil anterior, pero los bigotes muestran una línea del tercer cuartil al máximo mucho más larga en comparación con la línea del mínimo al primer cuartil. Esto indica una mayor dispersión en los valores altos dentro de este cuartil, con una cola más extendida hacia el máximo.

Cuartil 9 a 11: En este rango, sólo se observa la caja sin bigotes. Esto sugiere que hay poca o ninguna variabilidad fuera del rango intercuartílico, indicando que los datos están concentrados en una gama estrecha de valores sin extremos significativos.

Cuartil 58: Los bigotes están simétricos, pero la caja es muy pequeña y la línea de la mediana no es claramente visible. Esto indica que hay una distribución de datos muy compacta en torno a la mediana, con una pequeña variabilidad en los valores del cuartil.

### last_month_salary

![image](https://github.com/user-attachments/assets/bd5dd987-f827-4eee-8905-bb6734cd7676)

Como se observó en las medidas de tendencia central, el cuartil de 7,496 a 150,000 presenta un sesgo considerable hacia la derecha, lo que dificulta la visualización de los demás boxplots. Por lo tanto, vamos a ajustar el gráfico para mejorar la visualización de los otros boxplots.

![Captura de pantalla 2024-08-04 120126](https://github.com/user-attachments/assets/9e1e932f-b0b1-46d2-b4e9-33adbd31a99d)

En el rango de salarios de 3498 a 5797, se observa que la mediana y el máximo coinciden. Esto podría indicar que los datos en este rango están muy concentrados cerca del valor máximo, o que la variabilidad dentro de este segmento es extremadamente baja.

De 5798 a 7495: La línea de la mediana está centrada en la caja, y los bigotes están aproximadamente del mismo tamaño. Esto sugiere que la distribución de los salarios en este rango es más simétrica y homogénea, con una variabilidad más equilibrada tanto en el extremo inferior como en el superior

De 0 a 3497: En este rango, la línea de la mediana está centrada dentro de la caja, lo que indica una distribución relativamente equilibrada en torno a la mediana. Sin embargo, el bigote inferior es notablemente más largo que el bigote superior. Esto sugiere que hay una mayor dispersión en los salarios bajos en comparación con los salarios más altos dentro de este rango

# Correlación
Para finalizar el análisis exploratorio, procederemos a generar una matriz de correlación en Python para examinar las relaciones entre las variables seleccionadas que tenemos en la tabla_consolidado

![image](https://github.com/user-attachments/assets/874948ce-44fc-4062-9152-68ef3973fc5c)

Vemos un 0.58 que indica una correlación positiva moderada. Esto significa que hay una tendencia general a que los clientes que tienen un alto número de veces vencidas más de 90 días (valor alto en more_90_days_overdue) también tienden a ser clasificados como morosos (valor de 1 en default_flag)


[Riesgo Relativo](https://github.com/Maria-Data-Analyst/riesgo_relativo/tree/Consultas-Query/Riesgo_relativo)




