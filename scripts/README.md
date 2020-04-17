This document contains Python 3 code snippets for performing 
basic operations in Spark


1. Firstly, we need to create spark session.
In Jupyter notebook press New button on the right and choose 'Python 3' option from drop down list.
 Copy and paste the code below into the opened page and run it. It will initialize
 a new spark session for application 'SparkBootcampApp'
```
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("SparkBootcampApp").getOrCreate()
# check whether it works
spark.sql('SELECT "Test" as c1').show()
```

2. Reading with pyspark - lets create a dataframe with a proper schema:
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
df_customer.schema
```

3. Lets read from the data from csv file and output the content:
```
df_customer = spark.read.csv("/scripts/customer.csv", header=True)
df_customer.show()
df_customer.schema
```
what is the difference with the output above have you noticed?

4. Lets reread data from csv file and assign a proper schema:
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
df_customer.schema
```