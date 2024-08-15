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
| `user_id`        | Número de identificación del cliente.                                       |
| `default_flag`   | 	Clasificación de los clientes morosos (1 para clientes que pagan mal, 0 para clientes que pagan bien)         |
                                                   

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
En esta consulta SQL, realizamos una unión completa (`FULL JOIN`) entre las tablas `user_info` y `default`. Este tipo de unión garantiza que todos los registros de ambas tablas se incluyan en el resultado final, independientemente de si hay coincidencias entre ellas. Hemos decidido excluir la variable sex de la tabla user_info, ya que no contribuye significativamente a nuestro proyecto y puede introducir un sesgo adicional en términos de género.

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

Para visualizar mejor los datos antes de realizar la imputación, cargaremos la tabla `user_info_default` en Google Colab. A continuación se muestra el código en Python para crear un boxplot de la variable `last_month_salary` para los dos posibles valores de `default_flag`.

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
La variable number_dependents contiene 943 valores nulos. Para abordar esta situación, utilizaremos la moda como método de imputación. Dado que ya hemos cargado la tabla con los datos en Google Colab, crearemos un código para calcular y visualizar la moda de number_dependents ademas de realizar el proceso de imputación y ver los resultados, esto para llevarnos una idea clara del procedimeinto que debemos hacer en BigQuery

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

**1. Antes de imputar**:

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

Para abordar este problema, en esta etapa del procesamiento de datos, analizaremos la correlación entre las variables en cada tabla. Cada tabla será cargada en Google Colab como un DataFrame. Utilizaremos el siguiente código, ajustado según el DataFrame de cada tabla, para visualizar las matrices de correlación:

```python
# En este paso ya tenemos todas las librerias necesarias importadas y el DataFrame cargado
# Calcula la matriz de correlación
correlation_matrix = df.corr()
# Crea el mapa de calor
plt.figure(figsize=(8, 6))  # Ajusta el tamaño según lo necesites
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', fmt='.2f', vmin=-1, vmax=1)
plt.title('Mapa de Calor de la Correlación de Pearson (Variables Numéricas)')
plt.show()
```

### Tabla: `loans_outstanding`

![image](https://github.com/user-attachments/assets/f5a880e0-150d-409f-b338-a8bee7edc97e)

En la matriz de correlación, identificamos una alta correlación entre other_loans y total_loans. Para evitar problemas de multicolinealidad, no incluiremos la variable other_loans al crear nuestra tabla general.

### Tabla: `user_info`

![image](https://github.com/user-attachments/assets/fdad123e-dff4-46c4-8aee-3b0ccf4eb764)

En esta tabla, no hemos identificado valores relevantes que requieran exclusión. Por lo tanto, seleccionaremos todas las variables para la tabla general. Sin embargo, es necesario realizar un CAST en la variable user_id, ya que debemos manejarla como tipo texto.

### Tabla: `loans_detail`

![image](https://github.com/user-attachments/assets/12d3238c-94a5-4ab2-9d1b-8addf4501a62)

En esta tabla, hemos observado una alta correlación entre las variables que representan la cantidad de retrasos. Para evitar la multicolinealidad, seleccionaremos una de estas variables pero antes vamos a analizar un poco la desviación estándar entre las tres. 

![image](https://github.com/user-attachments/assets/78871587-fef9-41b1-bbb2-1a283cdd331e)

Dado que las variables tienen desviaciones estándar similares y considerando que en el futuro se validará una hipótesis utilizando la variable more_90_days_overdue, se ha decidido conservar esta variable en la tabla general y en los cálculos posteriores

# Identificación y Manejo de Datos Discrepantes en Variables Numéricas

En este procedimiento, identificamos y manejamos valores atípicos en variables numéricas mediante gráficos de boxplot en Google Colab (Python). Estos gráficos revelan cómo la presencia de datos atípicos puede dificultar la visualización de la caja, destacando principalmente los límites superiores y numerosos registros que parecen ser atípicos. Aunque estos valores atípicos no representan una gran cantidad de registros individualmente, en conjunto constituyen una parte significativa de la base de datos. Por lo tanto, en esta etapa, eliminaremos únicamente los valores que se encuentren extremadamente alejados, con el objetivo de preservar los registros relevantes y mantener la integridad de los datos.

A continuación, se presenta el código para generar los boxplots para las variables de interés. Ajusta el nombre de la variable y del dataframe según sea necesario:

``` python
import plotly.express as px
# Dejamos esta linea de codigo por si queremos ver más a detalle un rango de valores
df_detail_e = df_detail[df_detail['debt_ratio'] >= 0]

# Crear un boxplot interactivo usando Plotly para 'debt_ratio'
fig = px.box(df_detail_e, y='debt_ratio', points="all", title="Boxplot debt_ratio")

# Ajustar el ancho y alto del gráfico
fig.update_layout(
    width=800,   # Ancho en píxeles
    height=600   # Alto en píxeles
)

# Mostrar el gráfico
fig.show()
```

## Tabla: `user_info_default`

### `last_month_salary`

![Boxplot de last_month_salary](https://github.com/user-attachments/assets/f2bdaa94-9db6-4eb2-9beb-dfd24b219bac)

![image](https://github.com/user-attachments/assets/56cca741-a841-4400-b461-9157b3de1770)

Se identificaron valores iguales o mayores a 428,000 que se alejan significativamente del resto. Tras realizar una consulta en BigQuery, encontramos 5 usuarios con estos valores extremos y decidimos eliminarlos para mantener la calidad de los datos.

### `age`

![Boxplot de age](https://github.com/user-attachments/assets/df2007f3-ee9a-46a9-88d4-48c34bb74a61)

![image](https://github.com/user-attachments/assets/fa26c41e-92f3-4ccb-85b8-b128ac79dbda)

Los valores de edad mayores a 96 también se destacan como outliers. Identificamos 10 usuarios con estos valores mediante una consulta en BigQuery y los eliminamos para preservar la consistencia de los datos.

Para eliminar estos datos modificaremos la ultima sentencia de la vista de la tabla `user_info` con la siguiente condición

``` sql
FROM imputacion  WHERE imputed_last_month_salary < 428000 AND age<96
```
## Tabla: `loans_detail`

### `debt_ratio`

![Boxplot de debt_ratio](https://github.com/user-attachments/assets/03ccc86b-7115-4b44-b7bc-e2b301521aaa)

![image](https://github.com/user-attachments/assets/047d2ad4-4251-44b3-a5ad-9de9ade65838)

Se encontraron 3 usuarios con valores extremos mayores  a 49.112 en `debt_ratio`, que fueron eliminados para mantener la integridad de los datos.

### `more_90_days_overdue`

![Boxplot de more_90_days_overdue](https://github.com/user-attachments/assets/7bd2aba8-9e6c-41b8-8465-6612e18a526d)

![image](https://github.com/user-attachments/assets/72d470d1-65c5-4b31-95ec-82a8eb599165)

Identificamos 63 usuarios con valores extremos en `more_90_days_overdue` (igual o mayor a 96), que también fueron eliminados.

### `using_lines_not_secured_personal_assets`

![Boxplot de using_lines_not_secured_personal_assets](https://github.com/user-attachments/assets/7ddcfa12-bda2-49b7-9a36-87a689abd9fc)

![image](https://github.com/user-attachments/assets/ede4a9d6-4eca-4307-b72a-43d01776eff0)

Se identificaron 4 datos extremos mayores a 12.369 en `using_lines_not_secured_personal_assets`, que fueron eliminados para asegurar la calidad de los datos.

Con los valores atípicos identificados, crearemos una vista de la tabla loans_detail que incluirá solo las variables seleccionadas, excluyendo los datos atípicos detectados:

* **user_id:** Utilizaremos esta variable como clave para unir la información con la tabla general, convirtiéndola en formato String.
* **more_90_days_overdue:** Esta variable es crucial para validar una hipótesis planteada en el proyecto.
* **using_lines_not_secured_personal_assets y debt_ratio:** Estas variables proporcionarán información sobre el nivel de endeudamiento de los usuarios.
  
Aplicaremos sentencias para excluir los valores atípicos previamente identificados. La consulta resultante se guardará como una vista llamada loans_detail_clean, que servirá como base para los análisis futuros.

``` sql

SELECT
CAST (user_id AS STRING) AS user_id,
more_90_days_overdue,using_lines_not_secured_personal_assets,number_times_delayed_payment_loan_30_59_days,
debt_ratio

  FROM `riesgo-relativo-1.dataset.loans_detail` WHERE more_90_days_overdue <96 AND number_times_delayed_payment_loan_30_59_days < 96 AND debt_ratio < 49112 AND using_lines_not_secured_personal_assets < 12369
```

# UNIÓN: TABLA CONSOLIDADA
En este paso, crearemos una vista llamada tabla_consolidado que integrará la información  de las tres tablas limpias desarrolladas previamente. Esta vista será nuestra base para la exploración de datos y será el lugar donde realizaremos los ajustes necesarios basados en los hallazgos del análisis.

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
loans_detail.debt_ratio

FROM `riesgo-relativo-1.dataset.loans_detail_clean` AS loans_detail
INNER JOIN
`riesgo-relativo-1.dataset.loans_outstanding_clean` AS loans_outstanding
ON 
loans_detail.user_id = loans_outstanding.user_id
INNER JOIN
`riesgo-relativo-1.dataset.user_info_default` AS user_default
ON 
user_default.user_id = loans_outstanding.user_id
``` 


Esta consulta utiliza ` INNER JOIN ` para asegurar que mantenemos los datos completos y consistentes. Aunque sabemos que los datos están limpios, es posible que haya diferencias en la cantidad de datos entre las tablas. Al emplear INNER JOIN, solo incluimos a los usuarios que están presentes en todas las tablas, garantizando así que nuestra vista consolidada tenga solo los registros completos y sin inconsistencias.

![image](https://github.com/user-attachments/assets/fcaf8bea-496d-497f-93c1-feb0fed922dd)

 [Análisis Exploratorio](https://github.com/Maria-Data-Analyst/riesgo_relativo/blob/Consultas-Query/Bitacora/AED.md)
