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
Obtuvimos que solo hay valores nulos en la tabla `user_info_default`

![image](https://github.com/user-attachments/assets/610b97b4-b6f5-4e35-acbd-cb6bf220e35e)


