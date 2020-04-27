This document contains Python 3 code snippets for performing 
basic operations in Spark


1 . Firstly, we need to create spark session.
In Jupyter notebook press New button on the right and choose 'Python 3' option from drop down list.
 Copy and paste the code below into the opened page and press Run. It will initialize
 a new spark session for application 'SparkBootcampApp'
```
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("SparkBootcampApp").getOrCreate()
# check whether it works
spark.sql('SELECT "Test" as c1').show()
```
The following ouput confirms that spark is working:

+----+
|  c1|
+----+
|Test|
+----+

2a . Reading with pyspark - lets read from the data from csv file and output the content.
Copy and paste the command in the next section in jupyter, then press Run:
```
df_customer = spark.read.csv("/scripts/customer.csv", header=True)
df_customer.show()
df_customer.printSchema()
```

2b . Reading with pyspark - lets create a dataframe with a proper schema:
```
from pyspark.sql.types import StructType, StructField, IntegerType, StringType, Row
schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("phone", StringType(), True)
    ]
)
data = [
  Row(501, "Sue", "0332323232"),
  Row(607, "Anita", "0487878787"),
  Row(889, "Simon", "0222343434"),
]
df_customer = spark.createDataFrame(
  spark.sparkContext.parallelize(data),
  schema
)
df_customer.show()
df_customer.printSchema()
```

what is the difference with the output above have you noticed?

2c . Lets reread data from csv file and assign a proper schema:
```
from pyspark.sql.types import StructType, StructField, IntegerType, StringType, Row
schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("phone", StringType(), True)
    ]
)
df_customer = spark.read.csv("/scripts/customer.csv", schema=schema,header=True)
df_customer.show()
df_customer.printSchema()
```

3 . Writing in spark
```
from pyspark.sql.types import StructType, StructField, IntegerType, StringType, Row
import random
import string

letters = string.ascii_lowercase
FILE_NAME=f"{''.join(random.choice(letters) for i in range(10))}.csv"
schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("phone", StringType(), True)
    ]
)
data = [
  Row(501, "Sue", "0332323232"),
  Row(607, "Anita", "0487878787"),
  Row(889, "Simon", "0222343434"),
]
df_customer_2 = spark.createDataFrame(
  spark.sparkContext.parallelize(data),
  schema
)
df_customer_2.write.option("header","true").csv(f"/scripts/{FILE_NAME}")
df_customer_read = spark.read.csv(f"/scripts/{FILE_NAME}", schema=schema,header=True)
df_customer_read.show()
```
Notice how spark 'partinioned' the output file. This can be adjusted with coalesce function 
we will explore more in the Optimization session

4a . Basic transformations
Lets count how much each customer has spent on an order
```
order_schema = StructType([
    StructField("customer_id", IntegerType(), False),
    StructField("order_id", IntegerType(), True),
    StructField("product", StringType(), True),
    StructField("price", IntegerType(), True)
    ]
)
df_customer = spark.read.csv("/scripts/customer.csv", header=True)
df_order = spark.read.csv("/scripts/order.csv", header=True, schema=order_schema)
df_customer_order = df_customer.join(df_order, df_customer.id == df_order.customer_id, how='inner')
df_customer_order.select('name', 'product', 'price').show()
df_order_sum = df_customer_order.groupBy('id').sum('price').withColumnRenamed("sum(price)", "total_price")
df_order_sum = df_order_sum.join(df_customer, df_order_sum.id == df_customer.id, how='inner').select('name','total_price').show()
```

4b. Basic transformations in Spark - try to obtain the result by your own:
 -Find the contact number of the customer who did not make any order
 -Find the difference between customer who spent the most and customer who spent the least on an order (HINT. there is
 a function in pyspark called col if you want to refer to a specific column in spark. For example, the following code 
 assignes to the new column called 'new_column' a value from existing column called 'existing_column':
 
 from pyspark.sql.functions import col
 df_customer.withColumn('new_column', col("existing_column))
 )
 
5 . The next workshop session preview. Examples of partitioning and coalesce (No need to run, just listen):
```
order_schema = StructType([
    StructField("customer_id", IntegerType(), False),
    StructField("order_id", IntegerType(), True),
    StructField("product", StringType(), True),
    StructField("price", IntegerType(), True)
    ]
)
df_customer = spark.read.csv("/scripts/customer.csv", header=True)
df_order = spark.read.csv("/scripts/order.csv", header=True, schema=order_schema)

partitioned_customer_df = df_customer.repartition(4, 'id')
partitioned_order_df = df_order.repartition(4, 'customer_id')
```

Operation opposite to ```repartition()``` is ```coalesce()```:
```
coalesce_order_df = df_order.coalesce(2)
```

