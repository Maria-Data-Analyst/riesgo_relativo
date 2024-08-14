# ANÁLISIS EXPLORATORIO 

Comenzaremos visualizando histogramas para cada variable de la tabla para entender mejor la distribución de los datos. Estos histogramas nos permitirán identificar patrones, tendencias y posibles anomalías en cada variable, facilitando así una exploración más profunda de los datos

![image](https://github.com/user-attachments/assets/851a79bd-37a0-4efe-842c-8e721f532fc6)

El histograma de la edad muestra una distribución que, en general, no presenta un sesgo evidente. Sin embargo, he observado fluctuaciones en la cantidad de usuarios en distintos rangos de edad. Al analizar otros histogramas, he notado que varias variables presentan un sesgo positivo hacia la derecha, indicando la presencia de valores extremadamente altos que afectan la distribución de los datos. En el análisis previo, los boxplots demostraron cómo estos valores elevados influían en la visualización (ver el apartado de procesamiento y limpieza de datos). Dado que no fue posible eliminarlos debido a su importancia en la base de datos, he decidido segmentar las variables. Esta estrategia permitirá un análisis exploratorio más detallado y ayudará a gestionar mejor los valores elevados.

Dado lo anterior, inicié la segmentación utilizando cuartiles. No obstante, para lograr una visualización y segmentación más precisa en algunas variables opté por una segmentación manual. Esto me permitió ajustar las categorías de manera más efectiva y obtener una visión más clara. 

Para observar el comportamiento de la segmentación, calcule las **medidas de tendencia central** 


![Captura de pantalla 2024-08-11 162234](https://github.com/user-attachments/assets/bd79a1ed-9b69-4cc6-b23c-57d46c7e46ec)


![Captura de pantalla 2024-08-11 161949](https://github.com/user-attachments/assets/d4cc2330-dc0b-4493-bdd5-6394459e868e)



Estas medidas nos permiten evaluar la eficacia de la segmentación. Para las variables age, total_loans y last_month_salary, la mediana y el promedio son bastante cercanos, lo que sugiere una distribución relativamente uniforme. Sin embargo, en debt_ratio y using_lines_not_secured_personal_assets, el último cuartil presenta un sesgo positivo hacia la derecha, ya que el promedio es significativamente mayor que la mediana. Esto indica que estos segmentos están afectados por valores extremos elevados, que mantienen el sesgo positivo.

A pesar de este sesgo, decidí no modificar el último cuartil en estas variables debido a que debt_ratio y using_lines_not_secured_personal_assets son indicadores clave del nivel de endeudamiento. Valores superiores a 0.8 en estas variables ya representan una preocupación significativa en sí mismos. Por lo tanto, mantener el segmento tal como está nos ayuda a identificar y gestionar adecuadamente los casos con mayor riesgo de endeudamiento. 

La mayor desviación estándar en el último cuartil de todas las variables indica que los valores en este segmento son mucho más variados. Esto podría ser el resultado de la presencia de valores atípicos o extremos que están influyendo significativamente en la variabilidad de los datos.

Para **explorar las variables categóricas**, decidí plantear preguntas específicas y analizar cómo responden los datos a estas interrogantes. Además, voy a graficar las hipótesis planteadas en el proyecto para obtener una primera visión de su validez

### 1.¿Cuál es el rango de edad predominante entre los usuarios, y cuáles son los grupos etarios que solicitan más préstamos?

![image](https://github.com/user-attachments/assets/2672bdb7-6ade-4ea4-b34a-9b300ab81c90)


El gráfico está filtrado para mostrar únicamente a los clientes catalogados como morosos. En él, el rango etario con la mayor cantidad de usuarios es el de 47 a 63 años, y también es el grupo con el mayor número de préstamos. Este gráfico sugiere, como primer indicio, que la hipótesis de que los más jóvenes tienen un mayor riesgo de impago podría ser incorrecta, ya que el número de usuarios en el rango de edad más joven es menor. Sin embargo, esta observación no proporciona una base sólida, ya que es necesario comparar la cantidad de clientes en cada segmento. Este análisis más detallado se realizará mediante el cálculo del riesgo relativo en etapas posteriores.




### 2. ¿En qué rango de salario se encuentran los usuarios con mayor cantidad de préstamos?

![image](https://github.com/user-attachments/assets/dd333599-8560-4b4e-8f89-2298b55a712a)

Con el fitro de default_flag=1, la mayoría de los usuarios se encuentran en el rango de salario de 0 a 3.947 dólares, que es el segundo rango con la mayor cantidad de préstamos, solo superado por el rango de 3.498 a 5.797 dólares.



### 3. ¿Qué rango de salario corresponde a los usuarios que han caído en mora con mayor frecuencia?

![image](https://github.com/user-attachments/assets/7e8cd86f-b0b4-4877-9ee5-6bc25079bfd4)


Con el fitro de default_flag=1 , obervamos que el rango salarial que ha acumulado más veces en mora  es el de 0 a 3.947

### 4. ¿ Las personas con más cantidad de préstamos activos tienen mayor riesgo de ser malos pagadores ?

![image](https://github.com/user-attachments/assets/0c544384-91ab-494f-b4c7-b5c706518a45)

Vemos que los clientes morosos tienen los porcentajes más altos en los rangos de 1 a 8 préstamos. Esto sugiere, a primera vista, que la hipótesis podría ser incorrecta, ya que los usuarios con un mayor número de préstamos no son los más propensos a ser clasificados como morosos.

### 5 ¿Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores?

![image](https://github.com/user-attachments/assets/3edc292a-078b-410f-b1ab-646ccf18658a)

Aquí podemos observar que el 72% de los usuarios catalogados como morosos ha tenido entre 1 y 4 atrasos de más de 90 días. Este resultado apoya la hipótesis propuesta

Estas preguntas nos proporcionan una idea más clara sobre el perfil y el comportamiento de los usuarios. Sin embargo, esta exploración por sí sola no es suficiente para validar o refutar las hipótesis. Por lo tanto, continuaremos utilizando técnicas de análisis más avanzadas para obtener fundamentos sólidos en la validación de las hipótesis planteadas y para desarrollar un proceso automatizado de análisis de clientes que permita identificar eficazmente a los buenos y malos pagadores.


[Técnica de análisis](https://github.com/Maria-Data-Analyst/riesgo_relativo/blob/Consultas-Query/Ficha_tecnica/tecnica_analisis.md)
