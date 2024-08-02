# Procesamiento y Preparación de la Base de Datos

En esta sección, nos familiarizaremos con las tablas y variables disponibles para entender con qué datos contamos.

## Descripción de las Tablas y Variables

A continuación se detalla la información contenida en cada tabla:

### Tabla: `user_info`

| Variable            | Descripción                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| `user_id`           | Número de identificación del cliente (único para cada cliente).              |
| `age`               | Edad del cliente.                                                             |
| `sex`               | Sexo del cliente.                                                             |
| `last_month_salary` | Último salario mensual que el cliente reportó al banco.                       |
| `number_dependents` | Número de dependientes del cliente.                                           |

### Tabla: `loans_outstanding`

| Variable            | Descripción                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| `loan_id`           | Número de identificación del préstamo (único para cada préstamo).            |
| `user_id`           | Número de identificación del cliente.                                        |
| `loan_type`         | Tipo de préstamo (real estate = inmobiliario, others = otro).                 |

### Tabla: `loans_detail`

| Variable                                    | Descripción                                                                                                         |
|---------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| `user_id`                                   | Número de identificación del cliente.                                                                              |
| `more_90_days_overdue`                      | Número de veces que el cliente estuvo más de 90 días vencido.                                                        |
| `using_lines_not_secured_personal_assets`   | Cuánto está utilizando el cliente en relación con su límite de crédito, en líneas no garantizadas con bienes personales, como inmuebles y automóviles. |
| `number_times_delayed_payment_loan_30_59_days` | Número de veces que el cliente se retrasó en el pago de un préstamo (entre 30 y 59 días).                          |
| `debt_ratio`                                | Relación entre las deudas y el patrimonio del prestatario. Ratio de deuda = Deudas / Patrimonio.                      |
| `number_times_delayed_payment_loan_60_89_days` | Número de veces que el cliente retrasó el pago de un préstamo (entre 60 y 89 días).                                |

### Tabla: `default`

| Variable         | Descripción                                                                 |
|------------------|-----------------------------------------------------------------------------|
| `user_id`        | Número de identificación del cliente.                                      |
| `default_flag`   | Clasifica

### Integración de la Tabla `default` con `user_info`

La tabla `default` contiene únicamente dos variables: `user_id` y `default_flag`. Para consolidar esta información con la tabla `user_info`, utilizaremos el `user_id` como clave de unión. A continuación se muestra cómo se realizará esta integración:

```sql
SELECT
  user.user_id,
  user.age,
  user.last_month_salary,
  user.number_dependents,
  df.default_flag
FROM `riesgo-relativo-1.dataset.user_info` AS user
FULL JOIN `riesgo-relativo-1.dataset.default` AS df
ON user.user_id = df.user_id
```
En esta consulta SQL, realizamos una unión completa (`FULL JOIN`) entre las tablas `user_info` y `default`. Este tipo de unión garantiza que todos los registros de ambas tablas se incluyan en el resultado final, independientemente de si hay coincidencias entre ellas. 

## Identificar y manejar los valores nulos
Este procedimiento se realizara para las tres tablas del dataset. 

Para este paso utilizaremos las siguientes consultas:

```sql
 -------- CONSULTA DE NULOS PARA LA TABLA user_info_default -----
SELECT
  COUNTIF(user_id IS NULL) AS nulos_user_id,
  COUNTIF(age IS NULL) AS nulos_age,
  COUNTIF(last_month_salary IS NULL) AS nulos_last_month_salary,
  COUNTIF(number_dependents IS NULL) AS nulos_number_dependents,
  COUNTIF(default_flag IS NULL) AS nulos_default_flag,
FROM
  `riesgo-relativo-1.dataset.user_info_default`;

-------- CONSULTA DE NULOS PARA LA TABLA loans_detail -----
SELECT
  COUNTIF(user_id IS NULL) AS nulos_user_id,
  COUNTIF(more_90_days_overdue IS NULL) AS nulos_more_90_days_overdue,
  COUNTIF(using_lines_not_secured_personal_assets IS NULL) AS nulos_using_lines_not_secured_personal_assets,
  COUNTIF(number_times_delayed_payment_loan_30_59_days IS NULL) AS nulos_number_times_delayed_payment_loan_30_59_days,
  COUNTIF(debt_ratio IS NULL) AS nulos_debt_ratio,
  COUNTIF(number_times_delayed_payment_loan_60_89_days IS NULL) AS nulos_number_times_delayed_payment_loan_60_89_days,
FROM
  `riesgo-relativo-1.dataset.loans_detail`;

-------- CONSULTA DE NULOS PARA LA TABLA loans_outstanding -----
SELECT
  COUNTIF(user_id IS NULL) AS nulos_user_id,
  COUNTIF(loan_id IS NULL) AS nulos_loan_id,
  COUNTIF(loan_type IS NULL) AS nulos_loan_type,
FROM
  `riesgo-relativo-1.dataset.loans_outstanding`;
```
Obtuvimos que solo hay valores nulos en la tabla `user_info_default` en las variables `number_dependents` y `last_month_salary`

![image](https://github.com/user-attachments/assets/610b97b4-b6f5-4e35-acbd-cb6bf220e35e)

## Imputación de Valores Faltantes
En este paso imputaremos los valores nulos de las dos variables de la tabla `user_info_default` 

###  `last_month_salary`
Dado que los 7,199 registros nulos representan más del 20% de los datos, realizaremos una imputación para manejar estos valores faltantes. La imputación se llevará a cabo utilizando el promedio de los datos correspondientes a `default_flag=1` (malos pagadores) y `default_flag=0` (buenos pagadores). Este enfoque es fundamental para el proyecto, cuyo objetivo es generar reglas precisas para identificar a los malos pagadores. Aseguraremos que los valores imputados sean coherentes con la clasificación de los usuarios.

Para visualizar mejor los datos antes de realizar la imputación, cargaremos la tabla `user_info_default` en Google Colab. A continuación se muestra el código en Python para crear un boxplot de la variable `last_month_salary` para los registros con `default_flag=1`.

```python
import plotly.express as px

# Filtrar los datos para default_flag = 1
df_default_1 = df[df['default_flag'] == 1]

# Filtrar los datos para default_flag = 0
df_default_0 = df[df['default_flag'] == 0]

# Crear un boxplot interactivo para last_month_salary con default_flag = 1
fig_1 = px.box(df_default_1, y='last_month_salary', points="all", title="Boxplot de Last Month Salary (default_flag = 1)")

# Mostrar el gráfico para default_flag = 1
fig_1.show()

# Crear un boxplot interactivo para last_month_salary con default_flag = 0
fig_0 = px.box(df_default_0, y='last_month_salary', points="all", title="Boxplot de Last Month Salary (default_flag = 0)")

# Mostrar el gráfico para default_flag = 0
fig_0.show()
```

![Captura de pantalla 2024-08-01 152127](https://github.com/user-attachments/assets/0086406e-44a5-4a15-af5a-7a82d8ba5cd0)

![Captura de pantalla 2024-08-01 153110](https://github.com/user-attachments/assets/c3edb960-4ee9-4a90-804e-9646740feba0)

A partir de las gráficas generadas, hemos identificado la presencia de datos atípicos (outliers). Estos valores extremos pueden influir negativamente en la precisión del promedio calculado. Por ello, procederemos a excluir estos outliers para obtener un promedio más representativo y realizar una imputación más adecuada.

Para los registros clasificados como buenos pagadores (default_flag = 0), excluiremos los valores iguales o superiores a 15,731. Por otro lado, para los registros clasificados como malos pagadores (default_flag = 1), excluiremos los valores iguales o superiores a 10,895.
La consulta usaremos para calcular los promedios es la siguiente

``` sql
 SELECT
    ROUND(AVG(CASE 
          WHEN default_flag = 0 AND last_month_salary IS NOT NULL AND last_month_salary < 15731 THEN last_month_salary 
          ELSE NULL 
        END)) AS avg_salary_default_0,
    ROUND(AVG(CASE 
          WHEN default_flag = 1 AND last_month_salary IS NOT NULL AND last_month_salary < 10895 THEN last_month_salary 
          ELSE NULL 
        END)) AS avg_salary_default_1
  FROM `riesgo-relativo-1.dataset.default` df
  FULL JOIN `riesgo-relativo-1.dataset.user_info` user
  ON df.user_id = user.user_id
```
Esta consulta nos arroja los siguientes resultados 

![image](https://github.com/user-attachments/assets/d9f6e596-e739-43e4-907d-57f608193f40)


Antes de modificar la tabla de 'user_info_default' con la imputación miraremos los nulos de la variable `number_dependents` 

###  `number_dependents`
La variable number_dependents contiene 943 valores nulos. Para abordar esta situación, utilizaremos la moda como método de imputación. Dado que ya hemos cargado la tabla con los datos en Google Colab, crearemos un código para calcular y visualizar la moda de number_dependents para los casos en que default_flag sea 0 y 1

``` python
# Primero, revisamos los valores nulos en la columna 'number_dependents'
print(f"Valores nulos en 'number_dependents': {df['number_dependents'].isnull().sum()}")

# Filtramos los datos por 'default_flag'
df_default_0 = df[df['default_flag'] == 0]
df_default_1 = df[df['default_flag'] == 1]

# Calculamos la moda para 'number_dependents' cuando 'default_flag' es 0
moda_default_0 = df_default_0['number_dependents'].mode().iloc[0] if not df_default_0['number_dependents'].mode().empty else None
print(f"Moda de 'number_dependents' para default_flag=0: {moda_default_0}")

# Calculamos la moda para 'number_dependents' cuando 'default_flag' es 1
moda_default_1 = df_default_1['number_dependents'].mode().iloc[0] if not df_default_1['number_dependents'].mode().empty else None
print(f"Moda de 'number_dependents' para default_flag=1: {moda_default_1}")

# Imputamos los valores nulos con la moda correspondiente
df['number_dependents'] = df.apply(
    lambda row: moda_default_0 if pd.isnull(row['number_dependents']) and row['default_flag'] == 0 else 
                (moda_default_1 if pd.isnull(row['number_dependents']) and row['default_flag'] == 1 else 
                 row['number_dependents']),axis=1
)

# Verificamos que no queden valores nulos
print(f"Valores nulos después de imputar: {df['number_dependents'].isnull().sum()}")
```
![image](https://github.com/user-attachments/assets/814b3159-44f9-4e35-b9d5-cc622520fa66)

Sabiendo los valores con los que debemos imputar, modificaremos la tabla user_info_default para hacer este procedimiento con la siguiente consulta 

``` sql
---- 1. Valores con los que se va a imputar -------
WITH valores_imputacion AS (
  SELECT
    ROUND(AVG(CASE 
          WHEN df.default_flag = 0 AND user.last_month_salary IS NOT NULL AND user.last_month_salary < 15731 THEN user.last_month_salary 
          ELSE NULL 
        END)) AS avg_salary_default_0,
    ROUND(AVG(CASE 
          WHEN df.default_flag = 1 AND user.last_month_salary IS NOT NULL AND user.last_month_salary < 10895 THEN user.last_month_salary 
          ELSE NULL 
        END)) AS avg_salary_default_1,
    (SELECT number_dependents
     FROM (
       SELECT
         number_dependents,
         COUNT(*) AS frequency
       FROM `riesgo-relativo-1.dataset.user_info`
       GROUP BY number_dependents
       ORDER BY frequency DESC
       LIMIT 1
     )) AS mode_number_dependents
  FROM `riesgo-relativo-1.dataset.user_info` user
  LEFT JOIN `riesgo-relativo-1.dataset.default` df
  ON user.user_id = df.user_id
),
------ 2. Imputar ------------
imputacion AS (
  SELECT
    user.user_id,
    user.age,
    COALESCE(
      CASE 
        WHEN df.default_flag = 0 AND user.last_month_salary IS NULL THEN (SELECT avg_salary_default_0 FROM valores_imputacion)
        WHEN df.default_flag = 1 AND user.last_month_salary IS NULL THEN (SELECT avg_salary_default_1 FROM valores_imputacion)
        ELSE user.last_month_salary
      END, 
      user.last_month_salary
    ) AS imputed_last_month_salary,
    COALESCE(
      CASE 
        WHEN user.number_dependents IS NULL THEN (SELECT mode_number_dependents FROM valores_imputacion)
        ELSE user.number_dependents
      END, 
      user.number_dependents
    ) AS imputed_number_dependents,
    df.default_flag
  FROM `riesgo-relativo-1.dataset.user_info` user
  LEFT JOIN `riesgo-relativo-1.dataset.default` df
  ON user.user_id = df.user_id
)
--- 3. Seleccionar las variables------
SELECT
  user_id,
  age,
  imputed_last_month_salary AS last_month_salary,
  imputed_number_dependents AS number_dependents,
  default_flag
FROM imputacion;

```
Con esta consulta ya tenemos los valores imputados, vamos a ver los resultados

**1. Imputación de valores nulos  (Antes de imputar)**:
![Otra vista de la imputación](https://github.com/user-attachments/assets/c711a696-c774-4004-9ead-d68e73fd1927)

**2. Imputación de valores nulos  (Después de Imputar)**:
![Imagen después de imputar](https://github.com/user-attachments/assets/3f0a357c-299b-4586-992a-3da8643b9732)

**3. Comprobación consulta de valores nulos**:
![image](https://github.com/user-attachments/assets/61f9eccb-19ce-47b0-bd59-a96fa7e10bfe)


## Identificar y manejar valores duplicados
vrificamos los duplicados de todas las tablas con las siguientes Queries :

``` sql
---Consulta para verificar duplicados en la tabla user_info_default
SELECT
  user_id,
  COUNT (*) AS cantidad
FROM
  `riesgo-relativo-1.dataset.user_info_default`
GROUP BY
  user_id
HAVING
  COUNT (*)>1;

---Consulta para verificar duplicados en la tabla loans_detail
SELECT
  user_id,
  
  COUNT (*) AS cantidad
FROM
  `riesgo-relativo-1.dataset.loans_detail`
GROUP BY
  user_id 
HAVING
  COUNT (*)>1;  

---Consulta para verificar duplicados en la tabla loans_outstanding
SELECT
  user_id, 
  COUNT (*) AS cantidad
FROM
  `riesgo-relativo-1.dataset.loans_outstanding`
GROUP BY
  user_id
HAVING
  COUNT (*)>1;
```
Hemos identificado los siguientes duplicados en la tabla `loans_outstanding`:

![Duplicados en la tabla `loans_outstanding`](https://github.com/user-attachments/assets/1e7d723c-883b-47c6-8ba9-c3b49fd5e0dc)

Para entender la causa de estos duplicados, hemos revisado la tabla correspondiente y analizado los datos:

![Revisión de datos en la tabla](https://github.com/user-attachments/assets/7f5b4760-14fc-4f7f-bdf1-33cf25d703d4)


Hemos identificado que el user_id se repite para cada préstamo asociado a un usuario, lo que resulta en múltiples entradas para el mismo user_id en la tabla y genera duplicados aparentes.

Además, hemos detectado inconsistencias en la forma en que se registran los tipos de préstamos, con variaciones en su escritura. Esta falta de uniformidad puede causar problemas futuros. Para abordar esto, estandarizaremos los nombres de los préstamos convirtiéndolos a minúsculas. Esta estandarización nos permitirá contabilizar los préstamos de manera más precisa.

Crearemos tres nuevas variables:

* other_loans: Para reflejar la cantidad de préstamos que no son de bienes raíces.
* real_estate_loans: Para mostrar la cantidad de préstamos relacionados con bienes raíces.
* total_loans: Para mostrar la cantidad total de préstamos por usuario.

Estos cambios reemplazarán la variable loan_type original y eliminarán la columna loan_id, ya que esta última no es necesaria para los procedimientos futuros. Implementaremos estas modificaciones mediante una consulta sobre la tabla original `loans_outstanding` la cual guardaremos como una `vista` llamada `loans_outstanding_clean`

```sql
SELECT
  CAST(user_id AS STRING) AS user_id,
   -- Contar otros tipos de préstamos
  SUM(CASE
      WHEN LOWER(loan_type) = 'others' OR LOWER(loan_type) = 'other' THEN 1
      ELSE 0
  END
    ) AS other_loans,
 -- Contar préstamos inmobiliarios
  SUM(CASE
      WHEN LOWER(loan_type) = 'real estate' THEN 1
      ELSE 0
  END
-- Contar el total de préstamos
    ) AS real_estate_loans,
  COUNT(*) AS total_loans
FROM
  `riesgo-relativo-1.dataset.loans_outstanding`
GROUP BY
  user_id
ORDER BY
  user_id;
```
Además, hemos modificado el tipo de dato de user_id. Originalmente estaba definido como integer, pero en nuestro contexto es más adecuado tratarlo como texto.
Con esta modificaciones tenemos estos resultados: 

![image](https://github.com/user-attachments/assets/3c78de04-8a65-48ed-b625-12a36d2da2df)

Ahora, volveremos a ejecutar la consulta para los valores duplicados, ajustando la cláusula FROM para que apunte a la nueva tabla. Esto nos permitirá verificar los duplicados y asegurarnos de que los datos están correctamente actualizados.

![image](https://github.com/user-attachments/assets/06e3cf84-3b3c-4365-a624-9b624bf5fe7d)


# Selección de Variables

Una alta correlación entre dos variables puede señalar multicolinealidad, lo que indica que las variables están demasiado relacionadas. Esto puede afectar la precisión del modelo de regresión y complicar su interpretación, ya que dificulta la evaluación del impacto individual de cada variable.

Por esta razón, en esta etapa del procesamiento de datos, analizaremos la correlación entre las variables en cada tabla. Cargaremos las tablas en Google Colab y crearemos una matriz de correlación. Para ver el código de Python, visita el siguiente enlace:
[Ver código de Python ](https://github.com/Maria-Data-Analyst/riesgo_relativo/tree/Consultas-Query/python)

### Tabla: `loans_outstanding`

![image](https://github.com/user-attachments/assets/f5a880e0-150d-409f-b338-a8bee7edc97e)

En la matriz de correlación, identificamos una alta correlación entre other_loans y total_loans. Para evitar problemas de multicolinealidad, no incluiremos la variable other_loans al crear nuestra tabla general.

### Tabla: `user_info`

![image](https://github.com/user-attachments/assets/fdad123e-dff4-46c4-8aee-3b0ccf4eb764)

En esta tabla, no hemos identificado valores relevantes que requieran exclusión. Por lo tanto, seleccionaremos todas las variables para la tabla general. Sin embargo, es necesario realizar un CAST en la variable user_id, ya que debemos manejarla como tipo texto.

### Tabla: `loans_detail`

![image](https://github.com/user-attachments/assets/12d3238c-94a5-4ab2-9d1b-8addf4501a62)

En esta tabla, hemos observado una alta correlación entre las variables que representan la cantidad de retrasos. Para evitar la multicolinealidad, seleccionaremos una de estas variables basándonos en su desviación estándar. Optaremos por la variable con la mayor desviación estándar, ya que proporcionará una mayor variabilidad en los datos.

![image](https://github.com/user-attachments/assets/78871587-fef9-41b1-bbb2-1a283cdd331e)

Dado que la variable `number_times_delayed_payment_loan_30_59_days` presenta la mayor desviación estándar, la hemos seleccionado para incluirla en la tabla general, junto con las demás variables que no presentan alta correlación 

# Identificar y manejar datos discrepantes en variables numéricas
En este procedimiento, identificaremos los valores atípicos (outliers) en las variables numéricas utilizando gráficos de boxplot en Google Colab (Python). Crearemos un boxplot para cada variable numérica de las tres tablas que venimos manejando 

### Tabla: `loans_outstanding`
#### Last_month_salary
![Captura de pantalla 2024-08-01 200914](https://github.com/user-attachments/assets/f2bdaa94-9db6-4eb2-9beb-dfd24b219bac)

Al observar el gráfico, notamos que los valores iguales o mayores a 428,000 están significativamente alejados del resto de los datos. Para investigar más a fondo, realizaremos una consulta en BigQuery para identificar y contar a estos usuarios.

![image](https://github.com/user-attachments/assets/56cca741-a841-4400-b461-9157b3de1770)

Dado que identificamos únicamente 5 usuarios con estos valores extremos, decidimos eliminarlos de nuestra base de datos para mantener la calidad y consistencia de los datos.
Debido a que obtuvimos solo 5 usuarios podemos quitarlos de nuestra base de datos 
