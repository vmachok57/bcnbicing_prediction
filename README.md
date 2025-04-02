****Proyecto Final** - Posgrado en Data Science**

**Tabla de Contenidos
1. Introducción
2. Objetivo del Proyecto
3. Dataset
4. Preprocesamiento
5. Análisis Exploratorio
6. Modelado
7. Evaluación del Modelo
8. Limitaciones y Mejoras Futuras
9. Conclusiones
10. Autores

**Introducción**
Este proyecto forma parte del trabajo final del posgrado en Data Science e IA y tiene como objetivo abordar un problema real relacionado con el uso del sistema de bicicletas compartidas en Barcelona.
El foco principal es predecir la disponibilidad de espacios libres para aparcar una bicicleta en una estación determinada, en un momento concreto. Para ello, se parte de un conjunto de datos reales que recogen la disponibilidad en tiempo real, y se desarrollan diferentes estrategias de análisis, transformación de datos y modelado predictivo.

**Objetivo del Proyecto**
El principal objetivo es construir un modelo que permita predecir de forma fiable cuántos espacios estarán disponibles para aparcar en cada estación en función del histórico de uso y otras variables externas.
Además del modelo predictivo, se pretende:
- Analizar el comportamiento de los usuarios a lo largo del tiempo.
- Estudiar cómo afectan factores externos (día de la semana, festivos, altitud, etc.) al uso del sistema.
- Evaluar la relación entre estaciones cercanas y su influencia en la ocupación.

**Dataset**
Se utilizaron dos datasets principales:
- InfoStations - Contiene información estática sobre cada estación de bicicletas:
  ID, nombre y ubicación geográfica,
  Latitud y longitud,
  Capacidad máxima de anclajes,
  Código postal, altitud, is_charging_station, etc.


**Histórico de disponibilidad**
Dataset masivo con un histórico de más de 4 años y medio (datos cada 5 minutos), hemos cogido 2022-2024 para evitar que el periodo de COVID impacte el modelo:
- Número de bicicletas disponibles,
- Número de anclajes libres,
- Fecha y hora del reporte (ReportedDT),
- ID de la estación correspondiente.


**Preprocesamiento**
Dado el tamaño y la granularidad del dataset (más de 4 años con registros cada 5 minutos por estación), el preprocesamiento fue un paso clave.

1. Agregación temporal.
   Para reducir el tamaño del dataset y hacerlo más manejable, se agregaron los registros a nivel de hora, calculando la mediana horaria de disponibilidad. Esto facilitó los cálculos posteriores y redujo
   la complejidad del modelo.
2. Unión con InfoStations.
   Se hizo un join entre el dataset de disponibilidad y el de estaciones para incorporar datos como la capacidad máxima de cada estación. Esta información fue clave para validar si los datos de
   disponibilidad eran coherentes.
3. Validación y corrección de capacidad.
   En algunos casos, se detectaron incoherencias: la suma de bicicletas disponibles y espacios libres superaba la capacidad máxima registrada en InfoStations. Para corregirlo:
   Se priorizó el valor de num_bikes_available como más fiable.
   Se calculó num_docks_available_mod = capacidad - num_bikes_available, asegurando así que no se superara el máximo (capacidad).
4. Estimación de capacidad (cuando faltaba).
   Algunas estaciones no tenían capacidad registrada. En esos casos, se estimó como: capacidad = bikes_available + docks_available.
   Esta aproximación se consideró aceptable dadas las limitaciones del dataset.
5. Cálculo del porcentaje de disponibilidad.
   Se añadió una nueva variable:
   porcentaje_disponibilidad = docks_available / capacidad
   Este porcentaje estandariza la disponibilidad y facilita la comparación entre estaciones con diferentes capacidades.
6. Formato para modelo y submission.
   Para cumplir con el formato de entrega requerido:
   Cada fila representa una combinación de StationID, fecha y hora.
   Se incluyen variables con la disponibilidad en las 4 horas anteriores (t-1, t-2, t-3, t-4).
   Esto genera un dataset supervisado, donde cada fila contiene información histórica para predecir el valor actual.
7. Control de calidad y valores nulos.
   Se eliminaron todas las filas sin ReportedDT, ya que no se podía ubicar temporalmente.
   El resto de valores nulos, que eran pocos, se imputaron con la moda de cada columna correspondiente.
8. Eliminación de duplicados y filas redundantes.
   Para evitar solapamientos, se seleccionó solo la quinta fila de cada grupo temporal por estación.
   Esto asegura que cada fila tiene datos completos de las 4 horas anteriores y evita repetir la misma información en filas sucesivas.
9. Hemos intentado tratar los outliers, pero el modelo no ha mejorado significativamente, aunque Curtosis haya disminuido un poco, después de quitar el outlier. Por ejemplo hemos capado la variable “nearest_station_distance” al percentil 99 porque tenía un valor que destacaba mucho - outlier.
   
**Análisis Exploratorio**
Durante el análisis exploratorio se identificaron varios patrones relevantes en el uso del sistema de bicicletas:
1. Días laborables vs. fines de semana
   Se observa mayor uso de bicicletas en días laborables.
   Picos principales:
   Por la mañana temprano (entrada al trabajo)
   Por la tarde (salida del trabajo), donde se alcanza el máximo de uso diario.
   En fines de semana el patrón es más relajado, sin picos tan marcados.
![image](https://github.com/user-attachments/assets/d32d52c6-e461-4b7a-b71f-efa6ffadf2a3)

2. Festivos
   El patrón de uso en días festivos es similar al de los fines de semana: uso más bajo y horarios menos definidos.
   Se confirma que los días no festivos tienen mayor intensidad de uso.
![image](https://github.com/user-attachments/assets/b50408d1-6485-4243-84f8-2b6486e781f0)

3. Eventos especiales
   Se buscó una posible relación entre eventos de la ciudad (festivales, partidos, etc.) y la disponibilidad, pero no se identificó un patrón claro o significativo.

![image](https://github.com/user-attachments/assets/591e9d48-2448-4f14-a315-be8adabdbea6)

4. Temporalidad anual
   Se detecta un uso más bajo en invierno y un pico de uso en verano.
   Excepción: agosto, donde el uso baja bruscamente, probablemente por vacaciones y menor actividad laboral.
![image](https://github.com/user-attachments/assets/c4055d11-b500-45f7-81ec-c430e416e2bd)

6. Altitud
   Se observó una correlación positiva entre altitud y espacios disponibles.
   Posible interpretación: los usuarios toman bicicletas en zonas altas (montaña) y las dejan en zonas bajas (mar). No hacen el trayecto inverso con la misma frecuencia.
![image](https://github.com/user-attachments/assets/77675ca3-3931-4259-bf63-ab4fc2ef4790)
![image](https://github.com/user-attachments/assets/d082f3a6-129b-4b2d-87ad-ae44cf8910a2)
Green = More Docks Available
Red = Less Docks Available
7. Proximidad a estaciones de metro
   No se encontró correlación significativa entre la distancia al metro y la disponibilidad de anclajes.
![image](https://github.com/user-attachments/assets/a6d34f45-bb2e-4734-9e44-8753f516534d)
9. Densidad de estaciones cercanas
   Se observó que a mayor número de estaciones dentro de un radio de 300 metros, menor es la disponibilidad, probablemente por competencia por espacio.
![image](https://github.com/user-attachments/assets/81ab80f9-1cd8-4e99-8e52-8849f4436b3b)
10. Capacidad de la estación
    Se ha detectado una relación positiva tambien con la capacidad de la estación.
![image](https://github.com/user-attachments/assets/e63e9098-dc03-4976-aabd-3ba876898049)


***Modelado***
**Modelo 1: Regresión Lineal (baseline)**
Se entrenó una regresión lineal simple utilizando solo las variables históricas (t-1 a t-4) para establecer un modelo de referencia.
Split: 80% train / 20% test
Resultados:
![image](https://github.com/user-attachments/assets/db3f7a51-3ca1-465d-ba4f-eb6341ee43fb)


**Enriquecimiento del modelo**
Se añadieron múltiples variables nuevas:
Día de la semana
Indicador de festivo
Densidad de estaciones cercanas (100, 300, 500 m)
Altitud
Código postal
Cercanía al metro
Events de la ciudad
Cross Street (calle intersectante)
Cross Barrio (Barrio de la estación)

**Analisis de las Variables**
Hacer un OLS (Ordinary Least Squares), o regresión lineal ordinaria, al principio de un análisis de datos es importante por varias razones clave. Aunque XGBoost u otros modelos complejos pueden ser muy potentes, OLS proporciona una base sólida para comprender la relación entre las variables y ayuda a identificar posibles problemas con los datos o el modelo.
Al realizar OLS, obtenemos pruebas estadísticas (como el valor p) para evaluar si las variables son significativas para predecir el resultado. Esto nos permite identificar cuáles variables aportan valor al modelo.
Si algunas variables no son significativas (por ejemplo, valor p > 0.05), podríamos considerar eliminar esas variables o realizar transformaciones.

![image](https://github.com/user-attachments/assets/1d4f88ff-5d89-49ef-bc9e-60a2c9409bda)



1. Estadísticas generales del modelo:
R-squared (0.796): Indica que aproximadamente el 79.6% de la variabilidad en el porcentaje de muelles disponibles puede ser explicado por las variables independientes del modelo.
Adj. R-squared (0.796): Similar al R-squared, pero ajustado por el número de predictores. Un valor cercano al R-squared indica que la inclusión de más variables no ha causado una sobreajuste del modelo.
F-statistic (5.056e+05) y Prob (F-statistic) (0.00): La prueba F evalúa si al menos una de las variables independientes tiene una relación significativa con la variable dependiente.
Un valor p (Prob) de 0.00 indica que el modelo es altamente significativo.
Log-Likelihood (1.4140e+06): Mide la bondad de ajuste del modelo, con valores más altos indicando un mejor ajuste.
AIC (-2.828e+06) y BIC (-2.828e+06): Son criterios para la selección de modelos. Valores más bajos indican un buen ajuste.


2. Coeficientes de las variables independientes:
Cada coeficiente representa la relación de esa variable independiente con la variable dependiente (percentage_docks_available). Los coeficientes significativos son aquellos con un valor p (P>|t|) menor a 0.05. A continuación, se describen algunos de los resultados más relevantes:
- month (0.0009): Cada aumento en el mes incrementa el porcentaje de muelles disponibles en 0.0009 unidades. Este coeficiente es muy pequeño pero significativo (valor p < 0.001).
- day (1.463e-06 ): El día tiene un efecto positivo muy pequeño, pero no es significativo (P>|t| = 0.881), lo que sugiere que no hay una relación clara entre el día y la disponibilidad de muelles.
- hour (-4.054e-05): La hora del día tiene una relación negativa con la disponibilidad de muelles. Cada aumento en la hora disminuye ligeramente el porcentaje de muelles disponibles.
- capacity (0.0002): La capacidad tiene un efecto positivo, lo que sugiere que a mayor capacidad, hay más muelles disponibles.
- altitude (0.0004): El altitud tiene un coeficiente positivo y es altamente significativo, lo que implica que el aumento de la altitud está asociado con un aumento en la disponibilidad de muelles.
- special_event (-0.0011): Los eventos especiales no parecen tener un impacto significativo en la disponibilidad de muelles.
- is_weekend (-0.0008): Los fines de semana están asociados con una pequeña disminución en la disponibilidad de muelles.
- is_holiday (-0.0031): Similar al fin de semana, los días festivos también tienen una relación negativa con la disponibilidad de muelles.
- cross_street : Dependiendo de la calle donde se encuentre y con qué calle se cruce se observa una relación con la variable target.
- postal_code : Dependiendo del barrio la disponibilidad de los sitios para bicicletas puede depender. Se observa que es bastante interesante, ya que su p-value < 0.05.
- disponibilidad_porcentage_1h_antes (0.9658): La disponibilidad de muelles una hora antes tiene un impacto muy fuerte en la disponibilidad actual, con un coeficiente cercano a 1, lo que indica una relación muy fuerte y positiva.
- disponibilidad_porcentage_2h_antes (-0.0988): A mayor disponibilidad de muelles dos horas antes, menor es la disponibilidad de muelles en el momento actual, lo que sugiere un patrón cíclico.
- nearest_station_distance (-1.878e-05): A medida que aumenta la distancia hasta la estación más cercana, disminuye ligeramente la disponibilidad de muelles.
- stations_within_100m (-0.0011): La proximidad a otras estaciones dentro de los 100 metros tiene un efecto negativo en la disponibilidad de muelles, pero este efecto no es significativo.
- stations_within_300m (-0.0014): A más estaciones cercanas a 300 metros también muestran una relación negativa significativa con la disponibilidad de muelles.
Como también sugiere la grafica  del análisis exploratorio, la variable 300 metros parece tener una relación lineal negativa con la disponibilidad de muelles.
![image](https://github.com/user-attachments/assets/ec0b0102-f6f8-44de-9c25-18c8cd5da0da)
- stations_within_500m (0.0009): Sin embargo, las estaciones dentro de los 500 metros están positivamente relacionadas con la disponibilidad de muelles. Es posible debido al efecto de autocorrelación entre estas variables, para contrarrestar el efecto de la variable anterior. Por eso se toma la decisión de utilizar solo la variable stations_within_300m en el modelo final.

Se procede al análisis de factores de inflación de la varianza (FIV) para evaluar la multicolinealidad entre las variables independientes en el modelo. Observamos dos o más variables independientes que están altamente correlacionadas, lo que puede hacer que el modelo sea menos confiable.
![image](https://github.com/user-attachments/assets/cb8acf07-379a-4263-b99d-b7509215b537)
Tambien  observamos en las matrices de correlación fuerte correlación entre estas variables:
![image](https://github.com/user-attachments/assets/d29ceb5a-efcf-48b9-a0e0-9b708bff73ad)


3. Pruebas adicionales:
- Omnibus (310756.820): Es una prueba de normalidad de los residuos. Un valor p de 0.00 indica que los residuos no siguen una distribución normal.
- Durbin-Watson (1.988): Este estadístico evalúa la autocorrelación de los residuos. Un valor cercano a 2 indica que no hay autocorrelación significativa.
- Jarque-Bera (JB) y Prob (JB) (0.00): Indican que los residuos no siguen una distribución normal (esto se confirma con el valor p muy bajo).
- Skew (-0.145) y Kurtosis (10.540): La asimetría es baja, pero la curtosis indica que los residuos tienen colas más gruesas que una distribución normal. Esto indica que los residuos pueden tener más valores extremos (outliers) que una distribución normal, lo que podría implicar que el modelo no está capturando algunas variaciones importantes de los datos.

En resumen, el modelo OLS muestra que varias variables, como la capacidad, la altitud, y la disponibilidad  1 hora pasada de muelles tienen una relación significativa con el porcentaje de muelles disponibles, mientras que algunas otras variables como los eventos especiales y el día de la semana no tienen un impacto significativo.

Otra alternativa que hemos utilizado para mitigar las correlaciones entre variables independientes es  utilizar **PCA (Análisis de Componentes Principales)** para reducir la multicolinealidad en nuestro modelo y luego usar las componentes principales como entradas en un modelo de XGBoost para mejorar la eficiencia y la estabilidad del modelo, especialmente cuando las variables de entrada están altamente correlacionadas, como es nuestro caso.

**Explained_variance_ratio_** nos da el porcentaje de varianza explicada por cada componente principal. Si sumas los valores de explained_variance_ratio_, obtenemos la cantidad total de varianza explicada por todas las componentes principales.
Ahora obtenemos las componentes principales (que no están correlacionadas) los usamos en el modelo de XGBoost.
**Nos ha salido el R2 muy parecido que en el modelo de regression, mejorandolo a 0,81**
![image](https://github.com/user-attachments/assets/d6f9c9b0-5828-412d-babe-8d7ccd9c4b95)

Así mismo, se ha procedido a evaluar con el modelo **Ridge** para minimizar el error, pero con una penalización que busca reducir su magnitud (esto se hace para controlar el sobreajuste o overfitting).
El código ha ordenado las características según el valor absoluto de los coeficientes. La idea aquí es identificar cuáles características tienen el mayor impacto en el modelo. 
Los coeficientes con valores absolutos más grandes son más importantes, independientemente de si son positivos o negativos.
**Obtenemos un 0,79 con Regression de Ridge**

![image](https://github.com/user-attachments/assets/2b39ac09-b21c-4481-96c2-9430c6f64086)

Hemos tomado la decisión de utilizar las variables con p value < 5% y el resto de las variables que no aportan mucha variabilidad, eliminar del modelo.

![image](https://github.com/user-attachments/assets/9a061934-f970-4321-90fd-94ef03d48041)

Finalmente, entrenamos el modelo con las variables seleccionadas con el algoritmo **XGBoost y obtenemos el mejor R2 - 0,826**

Alternativamente hemos probado LSTM con solo 4 variables - la secuencia de la disponibilidad de las últimas 4 horas. A pesar de ser simple con la cantidad de caracteristicas, nos ha sorprendido con un resultado decente:
![image](https://github.com/user-attachments/assets/73f057c8-80a2-4808-871d-055a87f717b0)

Y despues aplicamos LSTM con el dataset mas amplio incluyendo todas las variables que consideramos para XGBoost. El resultado ha obtenido un lígera mejora en comparación con el modelo simple, lo que nos sugiere que las variables mas importantes para la predicción (y confirmado con todos los modelos) son la información de la disponibilidad de las últimas 2 horas.

![image](https://github.com/user-attachments/assets/6f01240d-3713-432e-b214-6ca8da10c0d0)



**Conclusiones**
Hemos probado distintos modelos y evaluamos distintas variables para predecir la disponibilidad en la estacion de bicing: desde una simple Regresion Lineal hasta una RNN LTSM. Sin embargo, nos quedamos con XGBoost, eligiendo el balance entre interpretabilidad y el resultado de las estimaciones.

**Autores**
Natalia Drevila,
Vitaliy Machok
