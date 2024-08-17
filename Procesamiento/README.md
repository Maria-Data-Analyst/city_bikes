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

- **4,319** de estos valores nulos pertenecían al tipo de usuario `Customer`, lo cual representaba el **71.31%**, dado que había un total de **6,057** usuarios de este tipo.
- Los **320** valores nulos restantes correspondían a usuarios del tipo `Subscriber`, lo que representaba el **0.73%** del total de **43,943** usuarios de este tipo.

Dado lo anterior, tomé la decisión de mantener los valores nulos de los usuarios `Customer`, mientras que los valores nulos en `birth_year` para los usuarios `Subscriber` los imputé con el promedio.

### Imputación

Para realizar un procedimiento adecuado, primero visualicé los datos en Google Colab para observar la distribución de los años de nacimiento, filtrando únicamente a los usuarios de tipo `Subscriber`.

![Distribución de Usuarios Subscriber](https://github.com/user-attachments/assets/e137e599-4d50-482d-abad-6cf9771356d1)

Dado que la imagen mostraba valores atípicos, específicamente aquellos menores a 1941, decidí excluirlos temporalmente para calcular un promedio más preciso. Luego, imputé los datos faltantes con este valor promedio y finalmente volví a unir todos los datos.

```sql
-- 1. Valores con los que se va a imputar
WITH valores_imputacion AS (
  SELECT
    CAST(ROUND(AVG(birth_year)) AS INT64) AS avg_subscriber
  FROM `city-bikes-1.dataset.trips`
  WHERE usertype = "Subscriber" AND birth_year IS NOT NULL AND birth_year > 1940
),

-- 2. Imputar valores nulos
imputacion AS (
  SELECT
    tripduration,
    stoptime,
    start_station_id,
    start_station_name,
    start_station_latitude,
    start_station_longitude,
    end_station_id,
    end_station_name,
    end_station_latitude,
    end_station_longitude,
    bikeid,
    usertype,
    CASE 
      WHEN usertype = "Subscriber" AND birth_year IS NULL THEN (SELECT avg_subscriber FROM valores_imputacion)
      ELSE birth_year
    END AS birth_year,
    gender
    
  FROM `city-bikes-1.dataset.trips`
)

-- 3. Seleccionar las variables 
SELECT 
  tripduration,
  stoptime,
  start_station_id,
  start_station_name,
  start_station_latitude,
  start_station_longitude,
  end_station_id,
  end_station_name,
  end_station_latitude,
  end_station_longitude,
  bikeid,
  usertype,
  birth_year,
  gender
FROM imputacion;

```


