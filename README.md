# Práctico HDFS, Hive y Spark

## 1. Preparación del entorno

Se preparó la plataforma Docker del práctico con Hadoop (HDFS y YARN), Hive, Spark y Jupyter. Se agregaron los archivos de configuración requeridos y el dataset `breweries.csv`.

La configuración se validó con `docker compose config` y luego se inició con Docker Compose. La comprobación mostró los servicios activos; Namenode, Datanode, ResourceManager, NodeManager e HistoryServer aparecieron saludables.

![Plataforma Big Data iniciada en Docker Desktop](Captura%20de%20pantalla%202026-11-17%20160442.png)

## 2. Carga y consulta en Hive

Se copió `breweries.csv` al Namenode, se guardó en HDFS en `/data/openbeer/breweries/breweries.csv` y se creó la base `openbeer`. Luego se definió la tabla externa `breweries` con columnas tipadas para el número de fila, nombre, ciudad, estado e identificador; se omitió la fila de encabezados del CSV.

La consulta muestra diez cervecerías con su ciudad y estado. El resultado confirma que Hive lee la tabla externa ubicada en HDFS.

```sql
SELECT name, city, state
FROM breweries
LIMIT 10;
```

![Consulta de cervecerías en Hive desde DBeaver](evidencia-hive-breweries.png)
