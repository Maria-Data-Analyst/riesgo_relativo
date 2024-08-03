# ANÁLISIS EXPLORATORIO
En esta sección, realizaremos un análisis exploratorio de los datos utilizando herramientas como Google Colab, BigQuery y Looker Studio. Nuestro objetivo es obtener una visión detallada y comprensible de la información contenida en nuestra tabla tabla_consolidada.

Comenzaremos visualizando histogramas para cada variable de la tabla para entender mejor la distribución de los datos. Estos histogramas nos permitirán identificar patrones, tendencias y posibles anomalías en cada variable, facilitando así una exploración más profunda de los datos
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

    CASE
      WHEN user_default.age >= 21 AND user_default.age <= 42 THEN '21 - 42'
      WHEN user_default.age >= 43 AND user_default.age <= 52 THEN '43 - 52'
      WHEN user_default.age >= 53 AND user_default.age <= 63 THEN '53 - 63'
      WHEN user_default.age >= 64 AND user_default.age <= 95 THEN '64 - 95'
      ELSE 'Fuera de rango'
    END AS age_range,

    CASE
      WHEN user_default.last_month_salary >= 0 AND user_default.last_month_salary <= 3947 THEN '0 - 3947'
      WHEN user_default.last_month_salary >= 3948 AND user_default.last_month_salary <= 5797 THEN '3948 - 5797'
      WHEN user_default.last_month_salary >= 5798 AND user_default.last_month_salary <= 7495 THEN '5798 - 7495'
      WHEN user_default.last_month_salary >= 7496 AND user_default.last_month_salary <= 150000 THEN '7496 - 150000'
      ELSE 'Fuera de rango'
    END AS salary_range,

    CASE
    WHEN loans_outstanding.total_loans >= 1 AND loans_outstanding.total_loans <= 4 THEN '1 - 4'
    WHEN loans_outstanding.total_loans >= 5 AND loans_outstanding.total_loans <= 8 THEN '5 - 8'
    WHEN loans_outstanding.total_loans >= 9 AND loans_outstanding.total_loans <= 11 THEN '9 - 11'
    WHEN loans_outstanding.total_loans >= 12 AND loans_outstanding.total_loans <= 57 THEN '12 - 57'
    ELSE 'Fuera de rango'
    END AS total_loans_range,

    CASE
    WHEN loans_detail.more_90_days_overdue = 0 THEN '0'
    ELSE '1'
    END AS more90_range,

    CASE 
     WHEN loans_detail.debt_ratio >= 0 AND loans_detail.debt_ratio <= 0.18 THEN '0 - 0.18'
     WHEN loans_detail.debt_ratio >= 0.19 AND loans_detail.debt_ratio <= 0.37 THEN '0.19 - 0.37'
     WHEN loans_detail.debt_ratio >= 0.38 AND loans_detail.debt_ratio <= 0.88 THEN '0.38 - 0.88'
     WHEN loans_detail.debt_ratio >= 0.89 AND loans_detail.debt_ratio <= 36705 THEN '0.89 - 36705'
    ELSE 'FUERA DE RANGO'
    END AS debt_ratio_range,

     CASE 
     WHEN loans_detail.using_lines_not_secured_personal_assets >= 0 AND loans_detail.using_lines_not_secured_personal_assets < 0.04 THEN '0 - 0.03'
     WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.04 AND loans_detail.using_lines_not_secured_personal_assets < 0.15 THEN '0.04 - 0.14'
     WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.15 AND loans_detail.using_lines_not_secured_personal_assets < 0.54 THEN '0.15 - 0.53'
     WHEN loans_detail.using_lines_not_secured_personal_assets >= 0.54 AND loans_detail.using_lines_not_secured_personal_assets <= 8710 THEN ' 0.54 - 8710'
    ELSE 'FUERA DE RANGO'
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
    user_default.user_id = loans_outstanding.user_id
```
Con esta consulta, obtendremos las variables seleccionadas previamente y generaremos una variable categórica de rangos para cada una de las variables que vamos a analizar

![image](https://github.com/user-attachments/assets/c408a9c9-badb-4991-93d6-06bd90ef7509)

## Asociación y visualización de variables categóricas  
Para este paso, conectaremos la tabla_consolidada en Looker Studio, lo que nos permitirá crear tablas y gráficos necesarios para el análisis. Además, formulamos preguntas específicas para orientar nuestra exploración de datos, de modo que podamos responderlas con la información disponible.

Para responder a las siguientes preguntas, hemos aplicado un filtro a la variable default_flag. Este filtro nos permite observar con mayor claridad el comportamiento de los usuarios en relación con los incumplimientos de pago.

### 1.¿Cuál es el rango de edad predominante entre los usuarios, y cuáles son los grupos etarios que solicitan más préstamos?

![image](https://github.com/user-attachments/assets/b58f54df-26e0-42e4-a123-8aa28b1615de)


Podemos observar que el rango etario con la mayor cantidad de usuarios es el de 21 a 42 años, mientras que el rango con la mayor cantidad de préstamos es el de 53 a 63 años. Sin embargo, al aplicar el filtro para visualizar los usuarios catalogados como morosos, encontramos lo siguiente:

![image](https://github.com/user-attachments/assets/7c7a0910-2ebb-4449-b5f1-722ba03dcdc9)


El rango etario con la mayor cantidad de clientes catalogados como morosos es también el de 21 a 42 años. Este es el mismo rango que tiene el mayor número de préstamos.


### 2. ¿En qué rango de salario se encuentran los usuarios con mayor cantidad de préstamos?

![image](https://github.com/user-attachments/assets/dd333599-8560-4b4e-8f89-2298b55a712a)

Con el fitro de default_flag=1, la mayoría de los usuarios se encuentran en el rango de salario de 0 a 3.947 dólares, que es el segundo rango con la mayor cantidad de préstamos, solo superado por el rango de 3.498 a 5.797 dólares.



### 3. ¿Qué rango de salario corresponde a los usuarios que han caído en mora con mayor frecuencia?

![image](https://github.com/user-attachments/assets/644f6e43-6fc6-4a38-8768-57a468d19de1)

Con el fitro de default_flag=1 , obervamos que el rango salarial que ha acumulado más veces en mora  es el de 0 a 3.947

