# Práctico HDFS, Hive y Spark

## 1. Preparación del entorno

Se preparó la plataforma Docker del práctico con Hadoop (HDFS y YARN), Hive, Spark y Jupyter. Se agregaron los archivos de configuración requeridos y el dataset `breweries.csv`.

La configuración se validó con `docker compose config` y luego se inició con Docker Compose. La comprobación mostró los servicios activos; Namenode, Datanode, ResourceManager, NodeManager e HistoryServer aparecieron saludables.

![Plataforma Big Data iniciada en Docker Desktop](evidencia/docker-plataforma.png)

## 2. Carga y consulta en Hive

Se copió `breweries.csv` al Namenode, se guardó en HDFS en `/data/openbeer/breweries/breweries.csv` y se creó la base `openbeer`. Luego se definió la tabla externa `breweries` con columnas tipadas para el número de fila, nombre, ciudad, estado e identificador; se omitió la fila de encabezados del CSV.

La consulta muestra diez cervecerías con su ciudad y estado. El resultado confirma que Hive lee la tabla externa ubicada en HDFS.

```sql
SELECT name, city, state
FROM breweries
LIMIT 10;
```

![Consulta de cervecerías en Hive desde DBeaver](evidencia/hive-consulta-breweries.png)

## 3. Exploración y medida con PySpark

Los tres TSV comprimidos se descargaron desde el sitio oficial de IMDb y se leyeron con esquemas explícitos: años y cantidades como enteros, calificaciones como double y los demás campos como texto. La ejecución se hizo en JupyterLab dentro de Docker, con Spark 3.3.1 en modo local de cuatro hilos, opción permitida por la consigna. El clúster `spark-master` rechazó la ejecución por los recursos disponibles.

### Exploración de las fuentes

- **`name.basics` (personas):** selección de nombre y profesiones para revisar registros; filtro de personas con profesión de actor o actriz; conteo agrupado de personas y revisión de nulos en año de nacimiento y profesión.
- **`title.basics` (títulos):** selección de identificador, tipo, título y año; filtro de películas recientes (desde 2020); conteo de registros agrupado por tipo de título.
- **`title.ratings` (calificaciones):** selección de identificador, promedio y votos; ordenamiento de títulos con más votos; agrupación por tramo entero de calificación.

Estas transformaciones permitieron reconocer el contenido, la escala, los tipos de títulos y la presencia de datos faltantes antes de cruzar las fuentes.

### Pregunta e indicador

**Pregunta:** ¿Cómo cambia la calificación ponderada al incluir todos los registros actorales con el nombre exacto Morgan Freeman frente a seguir a una sola persona identificada por IMDb como `nm0000151`?

En `name.basics`, el filtro por nombre exacto y profesión actoral devuelve tres identificadores. Para el segundo caso se usa el identificador estable `nm0000151`. En ambos, los títulos de `knownForTitles` se enlazan por `tconst` con `title.basics` y `title.ratings`, y se limitan a películas con calificación.

Para el cruce se usa `broadcast` sobre la lista pequeña de títulos conocidos de la persona, evitando redistribuir innecesariamente las tablas grandes. El indicador se calculó como:

```
suma(calificación × cantidad de votos) / suma(cantidad de votos)
```

El filtro por nombre reunió **8 películas**, **6.317.253 votos** y un rating ponderado de **8,85**. Al limitarlo a `nm0000151`, el resultado es de **4 películas**, **6.246.565 votos** y **8,88**. Los cuatro títulos del segundo caso son *The Shawshank Redemption*, *Se7en*, *Million Dollar Baby* y *Driving Miss Daisy*; concentran casi todos los votos, por eso la diferencia entre indicadores es pequeña.

El primer caso agrega los títulos conocidos de tres registros actorales con el mismo nombre exacto; el segundo sigue una sola identidad de IMDb. `knownForTitles` es una selección de obras destacadas y no representa necesariamente la filmografía completa.

![Ejecución visible en JupyterLab: películas asociadas a Morgan Freeman y calificación ponderada por votos](evidencia/spark-indicador-morgan-freeman.png)

**Cuaderno ejecutado:** [analisis_imdb.ipynb](notebooks/analisis_imdb.ipynb)
