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
10. Autores**

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
  ID, nombre y ubicación geográfica
  Latitud y longitud
  Capacidad máxima de anclajes
  Código postal, altitud, is_charging_station, etc


**Histórico de disponibilidad**
Dataset masivo con un histórico de más de 4 años y medio (datos cada 5 minutos):
- Número de bicicletas disponibles
- Número de anclajes libres
- Fecha y hora del reporte (ReportedDT)
- ID de la estación correspondiente


**Preprocesamiento**
Dado el tamaño y la granularidad del dataset (más de 4 años con registros cada 5 minutos por estación), el preprocesamiento fue un paso clave.

1. Agregación temporal
   Para reducir el tamaño del dataset y hacerlo más manejable, se agregaron los registros a nivel de hora, calculando la mediana horaria de disponibilidad. Esto facilitó los cálculos posteriores y redujo
   la complejidad del modelo.
2. Unión con InfoStations
   Se hizo un join entre el dataset de disponibilidad y el de estaciones para incorporar datos como la capacidad máxima de cada estación. Esta información fue clave para validar si los datos de
   disponibilidad eran coherentes.
3. Validación y corrección de capacidad
   En algunos casos, se detectaron incoherencias: la suma de bicicletas disponibles y espacios libres superaba la capacidad máxima registrada en InfoStations. Para corregirlo:
   Se priorizó el valor de num_bikes_available como más fiable.
   Se calculó num_docks_available_mod = capacidad - num_bikes_available, asegurando así que no se superara el máximo (capacidad).
4. Estimación de capacidad (cuando faltaba)
   Algunas estaciones no tenían capacidad registrada. En esos casos, se estimó como: capacidad = bikes_available + docks_available.
   Esta aproximación se consideró aceptable dadas las limitaciones del dataset.
5. Cálculo del porcentaje de disponibilidad
   Se añadió una nueva variable:
   porcentaje_disponibilidad = docks_available / capacidad
   Este porcentaje estandariza la disponibilidad y facilita la comparación entre estaciones con diferentes capacidades.
6. Formato para modelo y submission
   Para cumplir con el formato de entrega requerido:
   Cada fila representa una combinación de StationID, fecha y hora.
   Se incluyen variables con la disponibilidad en las 4 horas anteriores (t-1, t-2, t-3, t-4).
   Esto genera un dataset supervisado, donde cada fila contiene información histórica para predecir el valor actual.
7. Control de calidad y valores nulos
   Se eliminaron todas las filas sin ReportedDT, ya que no se podía ubicar temporalmente.
   El resto de valores nulos, que eran pocos, se imputaron con la moda de cada columna correspondiente.
8. Eliminación de duplicados y filas redundantes
   Para evitar solapamientos, se seleccionó solo la quinta fila de cada grupo temporal por estación.
   Esto asegura que cada fila tiene datos completos de las 4 horas anteriores y evita repetir la misma información en filas sucesivas.
9. Detectamos que una estación queda lejos del resto de las estaciones y decidimos capar el valor de la variable nueva "nearest_station_distance" a 99no percentil.
   
**Análisis Exploratorio**
Durante el análisis exploratorio se identificaron varios patrones relevantes en el uso del sistema de bicicletas:
1. Días laborables vs. fines de semana
   Se observa mayor uso de bicicletas en días laborables.
   Picos principales:
   Por la mañana temprano (entrada al trabajo)
   Por la tarde (salida del trabajo), donde se alcanza el máximo de uso diario.
   En fines de semana el patrón es más relajado, sin picos tan marcados.
![image](https://github.com/user-attachments/assets/d32d52c6-e461-4b7a-b71f-efa6ffadf2a3)

3. Festivos
   El patrón de uso en días festivos es similar al de los fines de semana: uso más bajo y horarios menos definidos.
   Se confirma que los días no festivos tienen mayor intensidad de uso.
4. Eventos especiales
   Se buscó una posible relación entre eventos de la ciudad (festivales, partidos, etc.) y la disponibilidad, pero no se identificó un patrón claro o significativo.
5. Temporalidad anual
   Se detecta un uso más bajo en invierno y un pico de uso en verano.
   Excepción: agosto, donde el uso baja bruscamente, probablemente por vacaciones y menor actividad laboral.
6. Altitud
   Se observó una correlación positiva entre altitud y espacios disponibles.
   Interpretación: los usuarios toman bicicletas en zonas altas (montaña) y las dejan en zonas bajas (mar). No hacen el trayecto inverso con la misma frecuencia.
7. Proximidad a estaciones de metro
   No se encontró correlación significativa entre la distancia al metro y la disponibilidad de anclajes.
8. Densidad de estaciones cercanas
   No se encontró relación clara con la distancia a la estación más cercana.
   Sin embargo, sí se observó que a mayor número de estaciones dentro de un radio de 300 metros, menor es la disponibilidad, probablemente por competencia por espacio.

***Modelado***
**Modelo 1: Regresión Lineal (baseline)**
Se entrenó una regresión lineal simple utilizando solo las variables históricas (t-1 a t-4) para establecer un modelo de referencia.

Split: 80% train / 20% test
Resultados:

[Aquí insertar captura con MSE, RMSE, R²]

**Enriquecimiento del modelo**
Se añadieron múltiples variables nuevas:
Día de la semana
Indicador de festivo
Densidad de estaciones cercanas (100, 300, 500 m)
Altitud
Código postal
Cercanía al metro
Disponibilidad en la estación más cercana (últimas 4 horas)
Se intentó incorporar datos de eventos de ciudad, pero fueron descartados por falta de estructura.

**Nuevos modelos probados**
Regresión Lineal con variables extendidas
Ridge Regression: con regularización L2 para evitar sobreajuste
XGBoost Regressor: modelo de gradient boosting más potente, capaz de capturar relaciones no lineales
Se analizó la importancia de las variables, lo que permitió detectar algunas sin impacto real.
[Esta parte la completará tu compañera con las variables descartadas]

**Reducción de dimensionalidad**
Se aplicó PCA (Análisis de Componentes Principales) para reducir la dimensionalidad del dataset y mejorar la eficiencia del modelo, eliminando redundancias sin perder capacidad explicativa.

**Evaluación del Modelo**
[Aquí puedes añadir gráficos comparativos de rendimiento, métricas de validación cruzada, curva de errores o interpretación de variables importantes en XGBoost.]

**Limitaciones y Mejoras Futuras**

Algunas variables añadidas no aportaron valor y fueron eliminadas.
[Espacio para detallar cuáles y por qué]
Los eventos especiales no pudieron integrarse por falta de una fuente de datos fiable.
Futuras mejoras podrían incluir:
Datos meteorológicos
Series temporales con redes LSTM
Mayor personalización por zona geográfica
Optimización del sistema con simulaciones en tiempo real


**Conclusiones**
[Espacio para añadir conclusiones personales, utilidad del modelo y aprendizaje obtenido.]

**Autores**

