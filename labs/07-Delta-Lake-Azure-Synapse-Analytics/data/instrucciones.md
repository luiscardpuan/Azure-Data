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

  
