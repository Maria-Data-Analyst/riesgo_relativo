# riesgo_relativo
TABLA USER_INFO_ DEFAULT

NULOS LAST_MONTH_SALARY

  DEFAULT_FLAG = 1
  
      * para determinar datos atipicos en last month salary de ususarios malos pagadores tomamos este grafico y decidimos que son los datos mayores a 10.000 los outliers
      * Asi que sacamos el promedio solo con los datos menores a 10.000 y ese es el dato que usamos en la imputacion para los null de last_month_slary con default_flag 1
      * valor de inputacion : 4.169
      
![image](https://github.com/user-attachments/assets/26ca519c-6404-484a-810c-81717751d982)

   DEFAULT_FLAG = 0
   
      *para determinar datos atipicos en last month salary de ususarios buenos pagadores tomamos este grafico y decidimos que son los datos mayores a 50.000 los outliers 
      * Asi que sacamos el promedio solo con los datos menores a 50.000 y ese es el dato que usamos en la imputacion para los null de last_month_slary con default_flag 0
      * valor de inputacion 6.261
      
![image](https://github.com/user-attachments/assets/fd3589c6-757d-46df-95ec-dedd09a79478)






unimos las tablas y nos damos cuenta que hay 425 nulos en la tabla prestamos vigentes que corresponden a los id de los clientes por lo tanto los vamos a quitar poque no podremos rastrearlos ni asociarlos a id de prestamos, ni con la demas informacion, para esto al unir las tablas usamos inner join y nos quedan en total 35.575 usuarios, con 7.032 nulos en last_month_salary y 910 en number dependent 
 
Se debe quitar la variable de genero porque agranda la brecha de genero asi que es prohibido 

Voy a filtrar solo los default flag en 1 y nos quedamos con 98 nulos en last_month, 524 no null
el grafico de dispersion nos da el siguiente resultado para last month salary donde vamor a borrar los outliers tomando desde el dato de 12.000, por lo tanto se borran 12 usuarios 
![image](https://github.com/user-attachments/assets/4f0f5135-13a3-4826-86f1-379e65df3ceb)
