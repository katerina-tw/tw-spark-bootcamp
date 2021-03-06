This document contains Python 3 code snippets for performing 
basic operations in Spark


1 . Firstly, we need to create spark session.
In Jupyter notebook press New button on the right and choose 'Python 3' option from drop down list.
 Copy and paste the code below into the opened page and press Run. It will initialize
 a new spark session for application 'SparkBootcampApp'. The very first initialization of spark session is taking
 time, so, please, be patient before output of the command appears.
 The subsequent runs with already created Spark Session are way more faster.
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
Now open a new tab in the browser and access historical spark jobs:
http://127.0.0.1:4040

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
DIR_NAME=f"{''.join(random.choice(letters) for i in range(10))}"
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
df_customer_2.write.option("header","true").csv(f"/scripts/{DIR_NAME}")
df_customer_read = spark.read.csv(f"/scripts/{DIR_NAME}", schema=schema,header=True)
df_customer_read.show()
```

Explore the directory structure of tw-spark-bootcamp and under script directory find newly created directory -  that's
where Spark has written the data. Notice how spark split the output file. This can be adjusted 
with coalesce function we will run later in the session.

4a . Basic transformations - lets count how much each customer has spent on an order.
Firstly, we need to join two dataframes - customer and order:
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
print("Joined dataframe:")
df_customer_order.show()
```
Secondly, run the aggregation with sum() function:
```
df_order_sum = df_customer_order.groupBy('id').sum('price').withColumnRenamed("sum(price)", "total_price")
print("Aggregated dataframe:")
df_order_sum.show()
```
And finally we join aggregated order dataframe back to customer to get the customer name:
```
print("Final dataframe:")
df_order_sum = df_order_sum.join(df_customer, df_order_sum.id == df_customer.id, how='inner').select('name','total_price').show()
```

4b. Basic transformations in Spark - try to obtain the result by your own:

 -Find the contact number of the customer who did not make any order
 
 -Find the difference between customer who spent the most and customer who spent the least on an order (HINT. there are
 functions in pyspark called 'min' and 'max' you can to refer to. For example:
 
 from pyspark.sql.functions import min, max
 
 test_df.select(min("test_column"))
 )
 
5 . In order to set a different degree of parallelism functions partitioning() and coalesce() are used:
```
df_customer = spark.read.csv("/scripts/customer.csv", header=True)
partitioned_customer_df = df_customer.repartition(4, 'id')
print ("Number of partitions for customer dataframe:")
partitioned_customer_df.rdd.getNumPartitions()
```

Operation opposite to ```repartition()``` is ```coalesce()```:

```
import random
import string

letters = string.ascii_lowercase
DIR_NAME=f"{''.join(random.choice(letters) for i in range(10))}"

coalesce_customer_df = partitioned_customer_df.coalesce(2)
coalesce_customer_df.write.option("header","true").csv(f"/scripts/{DIR_NAME}")
```

After running the last commands explore the content of the directories 
under tw-spark-bootcamp/scripts, find newly created directory and have a look whether there 
is any difference in the number of output files
