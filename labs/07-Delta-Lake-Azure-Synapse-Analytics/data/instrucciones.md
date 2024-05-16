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
  
