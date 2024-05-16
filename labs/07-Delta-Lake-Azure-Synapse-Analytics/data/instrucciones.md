# Delta Lake con Spark en Azure Synapse Analytics

Ejecutar el siguiente script:

```Python
%%pyspark
df = spark.read.load('abfss://files@xxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
## If header exists uncomment line below
##, header=True
)
display(df.limit(10))
```

Descomente la línea, *header=True* (porque el archivo productos.csv tiene los encabezados de columna en la primera línea), para que su código se vea así:

```Python
%%pyspark
df = spark.read.load('abfss://files@xxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
## If header exists uncomment line below
, header=True
)
display(df.limit(10))
```
Utilice el icono **▷** a la izquierda de la celda del código para ejecutarlo y espere los resultados. La primera vez que ejecuta una celda en un cuaderno, se inicia el grupo de Spark, por lo que puede tardar aproximadamente un minuto en devolver resultados. Finalmente, los resultados deberían aparecer debajo de la celda y deberían ser similares a este:

| ProductID | ProductName | Category | ListPrice |
| -- | -- | -- | -- |
| 771 | Mountain-100 Silver, 38 | Mountain Bikes | 3399.9900 |
| 772 | Mountain-100 Silver, 42 | Mountain Bikes | 3399.9900 |
| ... | ... | ... | ... |

  ### Cargue los datos del archivo en una tabla delta

Debajo de los resultados devueltos por la primera celda de código, use el botón + **Code** para agregar una nueva celda de código. Luego ingrese el siguiente código en la nueva celda y ejecútelo:

```Python
delta_table_path = "/delta/products-delta"
df.write.format("delta").save(delta_table_path)
```

En la pestaña de archivos , use el ícono ↑ en la barra de herramientas para regresar a la raíz del contenedor de **files** y observe que se ha creado una nueva carpeta llamada **delta**. Abra esta carpeta y la tabla delta de productos que contiene, donde debería ver los archivos en formato parquet que contienen los datos.

Regrese a la pestaña Cuaderno 1 y agregue otra celda de código nueva. Luego, en la nueva celda, agregue el siguiente código y ejecútelo:

```Python
from delta.tables import *
from pyspark.sql.functions import *

# Create a deltaTable object
deltaTable = DeltaTable.forPath(spark, delta_table_path)

# Update the table (reduce price of product 771 by 10%)
deltaTable.update(
    condition = "ProductID == 771",
    set = { "ListPrice": "ListPrice * 0.9" })

# View the updated data as a dataframe
deltaTable.toDF().show(10)
```

Los datos se cargan en un objeto **DeltaTable** y se actualizan. Puede ver la actualización reflejada en los resultados de la consulta.

Agregue otra celda de código nueva con el siguiente código y ejecútelo:

```Python
new_df = spark.read.format("delta").load(delta_table_path)
new_df.show(10)
```

El código carga los datos de la tabla delta en un marco de datos desde su ubicación en el lago de datos, verificando que el cambio realizado a través de un objeto **DeltaTable** haya persistido.

Modifique el código que acaba de ejecutar de la siguiente manera, especificando la opción para usar la función de viaje en el tiempo de delta lake para ver una versión anterior de los datos.

```Python
new_df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
new_df.show(10)
```

Cuando ejecuta el código modificado, los resultados muestran la versión original de los datos.

Agregue otra celda de código nueva con el siguiente código y ejecútelo:

```Python
deltaTable.history(10).show(20, False, True)
```
Se muestra el historial de los últimos 20 cambios en la tabla; debería haber dos (la creación original y la actualización que realizó).

## Crear tablas de catálogo

Hasta ahora ha trabajado con tablas delta cargando datos de la carpeta que contiene los archivos de parquet en los que se basa la tabla. Puede definir tablas de catálogo (*catalog tables*) que encapsulan los datos y proporcionan una entidad de tabla con nombre a la que puede hacer referencia en código SQL. Spark admite dos tipos de tablas de catálogo para delta lake:

-  *External* Tablas externas que están definidas por la ruta a los archivos de parquet que contienen los datos de la tabla.
-  *Managed* Tablas administradas , que se definen en el metastore de Hive para el grupo de Spark.

### Crear una tabla externa
En una nueva celda de código, agregue y ejecute el siguiente código:

```Python
spark.sql("CREATE DATABASE AdventureWorks")
spark.sql("CREATE TABLE AdventureWorks.ProductsExternal USING DELTA LOCATION '{0}'".format(delta_table_path))
spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsExternal").show(truncate=False)
```

Este código crea una nueva base de datos llamada **AdventureWorks** y luego crea una tabla externa llamada **ProductsExternal** en esa base de datos basada en la ruta a los archivos de parquet que definió anteriormente. Luego muestra una descripción de las propiedades de la tabla. Tenga en cuenta que la propiedad **Location** es la ruta que especificó.

Agregue una nueva celda de código y luego ingrese y ejecute el siguiente código:

```sql
%%sql

USE AdventureWorks;

SELECT * FROM ProductsExternal;
```

El código utiliza SQL para cambiar el contexto a la base de datos **AdventureWorks** (que no devuelve datos) y luego consulta la tabla ProductsExternal (que devuelve un conjunto de resultados que contiene los datos de los productos en la tabla Delta Lake).

### Crear una tabla administrada

En una nueva celda de código, agregue y ejecute el siguiente código:

```Python
df.write.format("delta").saveAsTable("AdventureWorks.ProductsManaged")
spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsManaged").show(truncate=False)
```

Este código crea una tabla administrada denominada **ProductsManaged**  basada en el DataFrame que cargó originalmente desde el archivo **products.csv** (antes de actualizar el precio del producto 771). No especifica una ruta para los archivos de parquet utilizados por la tabla; esto se administra automáticamente en el metastore de Hive y se muestra en la propiedad **Location** en la descripción de la tabla (en la ruta **files/synapse/workspaces/synapsexxxxxxx/warehouse** ).

Agregue una nueva celda de código y luego ingrese y ejecute el siguiente código:

```sql
%%sql

USE AdventureWorks;

SELECT * FROM ProductsManaged;
```

El código utiliza SQL para consultar la tabla **ProductsManaged**

### Comparar tablas externas y administradas

En una nueva celda de código, agregue y ejecute el siguiente código:

```sql
%%sql

USE AdventureWorks;

SHOW TABLES;
```
Este código enumera las tablas en la base de datos **AdventureWorks**

Modifique la celda del código de la siguiente manera, agregue ejecútelo:

```sql
%%sql

USE AdventureWorks;

DROP TABLE IF EXISTS ProductsExternal;
DROP TABLE IF EXISTS ProductsManaged;
```

Este código elimina las tablas del metastore

Regrese a la pestaña de archivos y vea la carpeta **files/delta/products-delta**. Tenga en cuenta que los archivos de datos todavía existen en esta ubicación. Al eliminar la tabla externa, se eliminó la tabla del metastore, pero se dejaron los archivos de datos intactos.

Vea la carpeta files/synapse/workspaces/synapsexxxxxxx/warehouse y tenga en cuenta que no hay ninguna carpeta para los datos de la tabla **ProductsManaged**. Al eliminar una tabla administrada, se elimina la tabla del metastore y también se eliminan los archivos de datos de la tabla.

### Crear una tabla usando SQL

Agregue una nueva celda de código y luego ingrese y ejecute el siguiente código:

```sql
%%sql

USE AdventureWorks;

CREATE TABLE Products
USING DELTA
LOCATION '/delta/products-delta';
```
Agregue una nueva celda de código y luego ingrese y ejecute el siguiente código:

```sql
%%sql

USE AdventureWorks;

SELECT * FROM Products;
```

Observe que se creó la nueva tabla de catálogo para la carpeta de tablas de Delta Lake existente, que refleja los cambios que se realizaron anteriormente.

## Utilice tablas delta para transmitir datos

El lago Delta admite la transmisión de datos. Las tablas delta pueden ser un *sink* o una *source* para flujos de datos creados con la API Spark Structured Streaming. En este ejemplo, utilizará una tabla delta como receptor para algunos datos de streaming en un escenario simulado de Internet de las cosas (IoT).

Regrese a la pestaña **Notebook 1** y agregue una nueva celda de código. Luego, en la nueva celda, agregue el siguiente código y ejecútelo:

```python
from notebookutils import mssparkutils
from pyspark.sql.types import *
from pyspark.sql.functions import *

# Create a folder
inputPath = '/data/'
mssparkutils.fs.mkdirs(inputPath)

# Create a stream that reads data from the folder, using a JSON schema
jsonSchema = StructType([
StructField("device", StringType(), False),
StructField("status", StringType(), False)
])
iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

# Write some event data to the folder
device_data = '''{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev2","status":"error"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"error"}
{"device":"Dev2","status":"ok"}
{"device":"Dev2","status":"error"}
{"device":"Dev1","status":"ok"}'''
mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
print("Source stream created...")
```

Asegúrese de que se imprima el mensaje *Source stream created...* El código que acaba de ejecutar creó una fuente de datos de transmisión basada en una carpeta en la que se guardaron algunos datos, que representan lecturas de dispositivos IoT hipotéticos.

En una nueva celda de código, agregue y ejecute el siguiente código:

```python
# Write the stream to a delta table
delta_stream_table_path = '/delta/iotdevicedata'
checkpointpath = '/delta/checkpoint'
deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
print("Streaming to delta sink...")
```

Este código escribe los datos del dispositivo de transmisión en formato delta.

En una nueva celda de código, agregue y ejecute el siguiente código:

```python
# Read the data in delta format into a dataframe
df = spark.read.format("delta").load(delta_stream_table_path)
display(df)
```

Este código lee los datos transmitidos en formato delta en un marco de datos. Tenga en cuenta que el código para cargar datos de transmisión no es diferente al utilizado para cargar datos estáticos desde una carpeta delta.

En una nueva celda de código, agregue y ejecute el siguiente código:

```python
# create a catalog table based on the streaming sink
spark.sql("CREATE TABLE IotDeviceData USING DELTA LOCATION '{0}'".format(delta_stream_table_path))
```

Este código crea una tabla de catálogo denominada **IotDeviceData** (en la base de datos **default** ) basada en la carpeta delta. Nuevamente, este código es el mismo que se usaría para datos que no son de transmisión.

En una nueva celda de código, agregue y ejecute el siguiente código:

```sql
%%sql

SELECT * FROM IotDeviceData;
```

Este código consulta la tabla **IotDeviceData**, que contiene los datos del dispositivo de la fuente de transmisión.

En una nueva celda de código, agregue y ejecute el siguiente código:

```python
# Add more data to the source stream
more_data = '''{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"ok"}
{"device":"Dev1","status":"error"}
{"device":"Dev2","status":"error"}
{"device":"Dev1","status":"ok"}'''

mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
```

Este código escribe más datos hipotéticos del dispositivo en la fuente de transmisión.

En una nueva celda de código, agregue y ejecute el siguiente código:

```sql
%%sql

SELECT * FROM IotDeviceData;
```

Este código vuelve a consultar la tabla **IotDeviceData**, que ahora debería incluir los datos adicionales que se agregaron a la fuente de transmisión.

In a new code cell, add and run the following code:

```python
deltastream.stop()
```

Este código detiene la transmisión.

## Consultar una tabla delta desde un grupo de SQL sin servidor

Además de los grupos de Spark, Azure Synapse Analytics incluye un grupo de SQL sin servidor integrado. Puede utilizar el motor de base de datos relacional de este grupo para consultar tablas delta mediante SQL.

En la pestaña de **files** , busque la carpeta **files/delta** .

Seleccione la carpeta **products-delta** y, en la barra de herramientas, en la lista desplegable Nuevo script SQL, **New SQL script** drop-down list, select **Select TOP 100 rows** SUPERIORES .

En el panel **Select TOP 100 rows**, en la lista **File type**, seleccione **Delta format** y luego seleccione **Apply**.

Revise el código SQL que se genera, que debería verse así:

```sql
-- This is auto-generated code
SELECT
    TOP 100 *
FROM
    OPENROWSET(
        BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/delta/products-delta/',
        FORMAT = 'DELTA'
    ) AS [result]
```

Utilice el ícono **▷** Ejecutar para ejecutar el script y revisar los resultados. Deberían verse similares a este:

  | ProductID | ProductName | Category | ListPrice |
  | -- | -- | -- | -- |
  | 771 | Mountain-100 Silver, 38 | Mountain Bikes | 3059.991 |
  | 772 | Mountain-100 Silver, 42 | Mountain Bikes | 3399.9900 |
  | ... | ... | ... | ... |

Esto demuestra cómo puede utilizar un grupo de SQL sin servidor para consultar archivos de formato delta que se crearon con Spark y utilizar los resultados para informes o análisis.

Reemplace la consulta con el siguiente código SQL:

```sql
USE AdventureWorks;

SELECT * FROM Products;
```

Ejecute el código y observe que también puede usar el grupo de SQL sin servidor para consultar datos de Delta Lake en tablas de catálogo definidas en el metastore de Spark.




