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
| `default_flag`   | 	Clasificación de los clientes morosos (1 para clientes que pagan mal, 0 para clientes que pagan bien)   |
