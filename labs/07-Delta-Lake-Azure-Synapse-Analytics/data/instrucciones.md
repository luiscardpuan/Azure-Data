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
