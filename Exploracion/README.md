# ANÁLISIS EXPLORATORIO
En esta sección, llevaremos a cabo un análisis exploratorio de los datos mediante consultas, gráficos y tablas. Utilizaremos tanto BigQuery como Looker Studio para obtener una visión detallada de los datos. Empezaremos con una segmentación por cuartiles, lo que nos permitirá clasificar y analizar las variables clave en diferentes grupos de datos. Esta segmentación nos ayudará a identificar patrones y tendencias significativas dentro de los datos.

## Segmentación por Cuartiles 
En el paso anterior, obtuvimos una tabla con todas las variables limpias y preparadas para su exploración. Para facilitar un análisis más detallado, segmentaremos algunas de estas variables utilizando la vista denominada tabla_consolidada. Esta segmentación por cuartiles nos permitirá examinar las variables en grupos específicos, ayudándonos a identificar patrones y tendencias de manera más efectiva.

``` sql

WITH cuartiles AS (
  SELECT
    user_default.user_id,
    user_default.age,
    user_default.last_month_salary,
    user_default.number_dependents,
    user_default.default_flag,
    loans_outstanding.real_estate_loans,
    loans_outstanding.total_loans,
    loans_detail.using_lines_not_secured_personal_assets,
    loans_detail.number_times_delayed_payment_loan_30_59_days,
    loans_detail.debt_ratio,

    NTILE(4) OVER (ORDER BY user_default.age) AS age_quartile,
    NTILE(4) OVER (ORDER BY user_default.last_month_salary) AS salary_quartile,
    NTILE(4) OVER (ORDER BY loans_outstanding.real_estate_loans) AS real_estate_loans_quartile,
    NTILE(4) OVER (ORDER BY loans_outstanding.total_loans) AS total_loans_quartile,
    NTILE(4) OVER (ORDER BY loans_detail.using_lines_not_secured_personal_assets) AS using_lines_quartile,
    NTILE(4) OVER (ORDER BY loans_detail.debt_ratio) AS debt_ratio_quartile

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
)
SELECT
  user_id,
  age,
  last_month_salary,
  number_dependents,
  default_flag,
  real_estate_loans,
  total_loans,
  using_lines_not_secured_personal_assets,
  number_times_delayed_payment_loan_30_59_days,
  debt_ratio,

  CASE age_quartile
    WHEN 1 THEN 'Q1'
    WHEN 2 THEN 'Q2'
    WHEN 3 THEN 'Q3'
    ELSE 'Q4'
  END AS age_quartile_label,

  CASE salary_quartile
    WHEN 1 THEN 'Q1'
    WHEN 2 THEN 'Q2'
    WHEN 3 THEN 'Q3'
    ELSE 'Q4'
  END AS salary_quartile_label,

  CASE real_estate_loans_quartile
    WHEN 1 THEN 'Q1'
    WHEN 2 THEN 'Q2'
    WHEN 3 THEN 'Q3'
    ELSE 'Q4'
  END AS real_estate_loans_quartile_label,

  CASE total_loans_quartile
  WHEN 1 THEN 'Q1'
    WHEN 2 THEN 'Q2'
    WHEN 3 THEN 'Q3'
    ELSE 'Q4'
  END AS total_loans_quartile_label,

  CASE using_lines_quartile
    WHEN 1 THEN 'Q1'
    WHEN 2 THEN 'Q2'
    WHEN 3 THEN 'Q3'
    ELSE 'Q4'
  END AS using_lines_quartile_label,

  CASE debt_ratio_quartile
   WHEN 1 THEN 'Q1'
    WHEN 2 THEN 'Q2'
    WHEN 3 THEN 'Q3'
    ELSE 'Q4'
  END AS debt_ratio_quartile_label

FROM 
  cuartiles


```
