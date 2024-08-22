# PRECESAMIENTO Y PREPARACIÓN DE LA BASE DE DATOS 
El dataset citi_bike_trips contiene 50,000 registros con las siguientes variables:

#### DESCRIPCIÓN DE LAS VARIABLES DE LA BASE DE DATOS 

| Variable               | Descripción                                                                                  |
|------------------------|----------------------------------------------------------------------------------------------|
| `tripduration`         | Duración del viaje en segundos.                                                              |
| `stoptime`             | Hora de finalización del viaje.                                                              |
| `start_station_id`     | ID de la estación donde comenzó el viaje.                                                    |
| `start_station_name`   | Nombre de la estación donde comenzó el viaje.                                                |
| `start_station_latitude`  | Latitud de la estación donde comenzó el viaje.                                            |
| `start_station_longitude` | Longitud de la estación donde comenzó el viaje.                                           |
| `end_station_id`       | ID de la estación donde terminó el viaje.                                                    |
| `end_station_name`     | Nombre de la estación donde terminó el viaje.                                                |
| `end_station_latitude`    | Latitud de la estación donde terminó el viaje.                                            |
| `end_station_longitude`   | Longitud de la estación donde terminó el viaje.                                           |
| `bikeid`               | ID de la bicicleta utilizada para el viaje.                                                  |
| `usertype`             | Tipo de usuario: Suscriptor (Subscriber) o Cliente ocasional (Customer).                     |
| `birth_year`           | Año de nacimiento del usuario.                                                               |
| `gender`               | Género del usuario: Desconocido (unknown), Masculino (male) o Femenino (female)                       |
| `customer_plan`        | Plan del cliente utilizado para el servicio de Citi Bike.                                    |

## Identificar y manejar los valores nulos

```sql
--- CONSULTAR VALORES NULOS ---
SELECT 
COUNTIF(tripduration IS NULL) AS nulos_tripduration,
COUNTIF(stoptime IS NULL) AS nulos_stoptime,
COUNTIF(start_station_id IS NULL) AS nulos_start_station_id,
COUNTIF(start_station_name IS NULL) AS nulos_start_station_name,
COUNTIF(start_station_latitude IS NULL) AS nulos_start_station_latitude,
COUNTIF(start_station_longitude IS NULL) AS nulos_start_station_longitude,
COUNTIF(end_station_id IS NULL) AS nulos_end_station_id,
COUNTIF(end_station_name IS NULL) AS nulos_end_station_name,
COUNTIF(end_station_latitude IS NULL) AS nulos_end_station_latitude,
COUNTIF(end_station_longitude IS NULL) AS nulos_end_station_longitude,
COUNTIF(bikeid IS NULL) AS nulos_bikeid,
COUNTIF(usertype IS NULL) AS nulos_usertype,
COUNTIF(birth_year IS NULL) AS nulos_birth_year,
COUNTIF(gender IS NULL) AS nulos_gender,
COUNTIF(customer_plan IS NULL) AS nulos_customer_plan,
 FROM `city-bikes-1.dataset.trips`
```
| nulos_tripduration | nulos_stoptime | nulos_start_station_id | nulos_start_station_name | nulos_start_station_latitude | nulos_start_station_longitude | nulos_end_station_id | nulos_end_station_name | nulos_end_station_latitude | nulos_end_station_longitude | nulos_bikeid | nulos_usertype | nulos_birth_year | nulos_gender | nulos_customer_plan |
|--------------------|----------------|------------------------|--------------------------|------------------------------|------------------------------|----------------------|------------------------|----------------------------|----------------------------|--------------|----------------|------------------|--------------|---------------------|
| 0                  | 0              | 0                      | 0                        | 0                            | 0                            | 0                    | 0                      | 0                          | 0                          | 0            | 0              | 4639             | 0            | 50000               |

## Análisis de Valores Nulos y Decisiones de Imputación

Observé que la variable `customer_plan` tenía **50,000** valores nulos, lo que indicaba que no contenía ningún valor válido. Por lo tanto, decidí excluirla del análisis. Además, al explorar más a fondo los valores nulos en la variable `birth_year`, descubrí lo siguiente:

- **4,319** de estos valores nulos pertenecían al tipo de usuario `Customer`, lo cual representaba el **71.31%**, dado que había un total de **6,057** usuarios de este tipo. Además, estos mismos 4,319 valores nulos coinciden con la variable gender = "unknown", que cuenta con un total de 5,119 registros, lo que equivale al 84% del total de usuarios con género desconocido. Por lo tanto, se ha decidido no eliminarlos ni imputarles valores, sino tratarlos de forma aislada mediante filtros por género
- 
- Los **320** valores nulos restantes correspondían a usuarios del tipo `Subscriber`, lo que representaba el **0.73%** del total de **43,943** usuarios de este tipo.

En vista de lo anterior, se tomó la decisión de mantener todos los valores nulos y manejarlos como una categoría de datos desconocidos para preservar la integridad del análisis


## Identificar Valores Duplicados
``` sql
SELECT 
bikeid,
stoptime,
 COUNT (*) AS cantidad
FROM `city-bikes-1.dataset.trips_procesada` 
GROUP BY
 bikeid,
stoptime
HAVING
  COUNT (*)>1;
```
Al ejecutar la consulta para identificar registros duplicados basados en bikeid y stoptime, no se encontraron duplicados en los datos.

## Procesos de Conversión y Cálculo de Variables

### 1. Cambio de Tipo de Variables

#### Conversión de Variables Numéricas a Tipo `STRING`

Las siguientes variables están originalmente en formato numérico. Para facilitar el análisis y la visualización en gráficos, las convertiremos a tipo `STRING`:

- **`bikeid`**: Identificador de la bicicleta.
- **`start_station_id`**: Identificador de la estación de inicio.
- **`end_station_id`**: Identificador de la estación de fin.

**Razón para la Conversión:**
Convertir estas variables a tipo `STRING` permite una mejor manipulación y visualización de los datos en gráficos y tablas, especialmente cuando estos identificadores se usan como etiquetas en informes y dashboards.

#### 2. Cambio de Tipo para `stoptime`

La variable `stoptime` está en formato `TIMESTAMP`, lo que puede complicar la visualización en gráficos debido a la inclusión de la fecha y hora en un solo campo. Por lo tanto, vamos a convertir `stoptime` a tipo `DATE` y separaremos la fecha de la hora, creando dos nuevas variables:

- **`stop_trip`**: Representará la fecha en la que terminó el viaje.
- **`stop_time`**: Indicará la hora exacta en la que terminó el viaje.

**Razón para la Conversión y Separación:**
Separar la fecha y la hora facilita la creación de gráficos y reportes donde la visualización de datos por fecha y hora es crucial. Permite un análisis más granular y la creación de informes más claros.

#### 3. Conversión de `tripduration` de Segundos a Minutos

La variable `tripduration` está en segundos y se convertirá a minutos para facilitar la interpretación de la duración de los viajes. La conversión se realiza dividiendo la duración en segundos por 60.

- **`tripduration_minutes`**: La duración del viaje en minutos.

**Razón para la Conversión:**
Convertir la duración del viaje de segundos a minutos hace que los datos sean más fáciles de interpretar y comparar, ya que los minutos son una unidad de tiempo más común para describir la duración de eventos en contextos de transporte y movilidad.

#### 4. Cálculo de Fecha y Hora de Comienzo

A partir de la variable `stoptime` y la duración del viaje en segundos (`tripduration`), calcularemos la fecha y la hora en que comenzó el viaje. Crearemos dos nuevas variables:

- **`start_trip`**: La fecha en la que comenzó el viaje.
- **`start_time`**: La hora en la que comenzó el viaje.

**Razón para el Cálculo:**
Calcular la fecha y hora de inicio del viaje es fundamental para analizar la duración de los viajes, identificar patrones y realizar estudios de tiempos de uso. Esta información es útil para entender mejor los hábitos de los usuarios y optimizar la gestión de la flota de bicicletas.

#### Resumen

- **Conversión de Identificadores**: Se cambia el formato numérico a `STRING` para una mejor visualización y manipulación en gráficos.
- **Conversión de `stoptime`**: Se separa en fecha (`stop_trip`) y hora (`stop_time`) para simplificar el análisis de datos.
- **Conversión de `tripduration`**: Se transforma de segundos a minutos para facilitar la interpretación.
- **Cálculo de Inicio del Viaje**: Se determinan `start_trip` y `start_time` para permitir un análisis detallado de la duración y los patrones de uso de las bicicletas.

Estos pasos mejorarán la claridad y utilidad de los datos, facilitando la visualización y el análisis en gráficos y reportes.

## Identificación y Manejo de Datos Atípicos

En esta sección, identifiqué y manejé datos atípicos. Utilicé Google Colab para graficar boxplots que me ayudaron a visualizar los datos atípicos.

### **`start_station_latitude`**

![Captura de pantalla 2024-08-17 195749](https://github.com/user-attachments/assets/fc702188-2052-455f-8b38-0a1ed6c2c08b)

A partir del boxplot, se observaron valores atípicos en `start_station_latitude`, específicamente aquellos menores a 40 y mayores a 41. Los cuales corresponden a los siguientes registros 

| tripduration | stoptime                        | start_station_id | start_station_name | start_station_latitude | start_station_longitude | end_station_id | end_station_name | end_station_latitude | end_station_longitude | bikeid | usertype   | birth_year | gender |
|--------------|---------------------------------|------------------|---------------------|------------------------|--------------------------|----------------|------------------|----------------------|------------------------|--------|------------|------------|--------|
| 102          | 2017-12-21 10:57:28.000000 UTC | 3650             | 8D Mobile 01        | 45.50585064             | -73.56910944             | 3650           | 8D Mobile 01     | 45.50585064           | -73.56910944             | 21928  | Subscriber | 1987       | male   |
| 379          | 2017-11-16 11:40:45.000000 UTC | 3650             | 8D Mobile 01        | 45.50585064             | -73.56910944             | 3650           | 8D Mobile 01     | 45.50585064           | -73.56910944             | 24354  | Subscriber | 1989       | female |
| 318          | 2017-10-01 21:01:53.000000 UTC | 3480             | WS Don't Use        | 0.0                     | 0.0                      | 3214           | Essex Light Rail | 40.7127742            | -74.0364857             | 31698  | Subscriber | 1989       | male   |

**Decisión de Exclusión**

Observé que solo había tres registros con valores atípicos en `start_station_latitude`. Debido a su bajo número, decidí excluir estos registros para mejorar la distribución de los datos y asegurar una mejor representación en los análisis posteriores.


### **`tripduration_m`**

![Captura de pantalla 2024-08-17 203527](https://github.com/user-attachments/assets/1f05aaa0-9792-4b94-9970-b66e80c8da95)


| tripduration_m | start_trip | start_time | stop_trip | stop_time | start_station_id | start_station_name  | start_station_latitude | start_station_longitude | end_station_id | end_station_name   | end_station_latitude | end_station_longitude | bikeid | usertype   | birth_year | gender |
|----------------|------------|------------|-----------|-----------|------------------|-----------------------|------------------------|--------------------------|----------------|--------------------|----------------------|------------------------|--------|------------|------------|--------|
| 54566          | 2017-12-23 | 11:07:45   | 2018-01-30 | 08:33:49 | 3427             | Lafayette St & Jersey St | 40.72430527             | -73.99600983             | 3042           | Fulton St & Utica Ave | 40.6794268            | -73.9298911             | 17401  | Subscriber | 1963       | male   |
| 12815          | 2017-12-04 | 17:49:40   | 2017-12-13 | 15:24:52 | 3469             | India St & West St     | 40.73181402             | -73.95995021             | 3432           | NYCBS Depot - GOW | 40.66906014           | -73.99463654             | 31021  | Subscriber | 1989       | male   |

**Decisión de Exclusión**

Observé que los valores atípicos mayores a 12.000 minutos en `tripduration_m` solo corresponden a 2 registros. Para mejorar la calidad de los datos y la distribución en el análisis, decidí eliminar estos registros.

### **`birth_year`**

![Captura de pantalla 2024-08-17 210032](https://github.com/user-attachments/assets/1519116c-cf25-46f1-9a1e-6a2525a4a9bd)

El gráfico revela valores atípicos para los años de nacimiento anteriores a 1941. Al analizar la cantidad de usuarios en esta categoría, se identificaron 72 casos, lo que representa solo el 0.14% del total de la base de datos. Dado que todos estos usuarios pertenecían al tipo Subscriber y eran pocos, se decidió eliminarlos del análisis para mantener la precisión y relevancia de los resultados.

## Histogramas

Manejé algunos valores outliers y así es como se ve la distribución de los datos en las variables. Observé que siguen existiendo sesgos y datos outliers, pero estos se encuentran en una cantidad representativa, por lo que decidí no eliminar ningún dato adicional. En lugar de eliminarlos, traté los outliers mediante segmentaciones con filtros en el dashboard.

![Captura de pantalla 2024-08-18 200817](https://github.com/user-attachments/assets/6ee40938-bd17-402d-a731-d2f54e383e22)


[Dashboard](https://lookerstudio.google.com/reporting/c3c25d71-65d2-4d50-9f05-005e491d11d2)
