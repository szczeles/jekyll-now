---
layout: post
title: Reading JSON, CSV and XML files efficiently in Apache Spark
excerpt: With Apache Spark you can easily read semi-structured files like JSON, CSV using standard library and XML files with spark-xml package. Sadly, the process of loading files may be long, as Spark needs to infer schema of underlying records by reading them. That's why I'm going to explain possible improvements and show an idea of handling semi-structured files in a very efficient and elegant way.
---

Data sources in Apache Spark can be divided into three groups:

* **structured data** like Avro files, Parquet files, ORC files, Hive tables, JDBC sources
* **semi-structured data** like JSON, CSV or XML
* **unstructured data**: log lines, images, binary files

The main advantage of structured data sources over semi-structured ones is that we know the schema in advance (field names, their types and "nullability"). These metadata are stored in files headers or are accessible via fast "describe" API for table-based sources.

JSON, XML and CSV are still widely used formats in data ingestion processes, though. Their popularity results from two major characteristics: they are all easy to read and they can be modified with lots of tools (starting with notepads, ending with Excel or `jq` command). The schema of semi-structured formats is not strict. That means we don't know what field names and data types to expect until we read at least part of the file. Well, in CSV we may have column names in the first row, but this is not enough in most cases.

That's why each time you open a JSON/CSV/XML dataset in Spark using the simplest API, you wait some time and see jobs executed in WebUI:

{% highlight python %}
# JSON
spark.read.json("sample/json/")

# CSV
spark.read.csv("sample/csv/", header=True, inferSchema=True)

# XML
spark.read.format("com.databricks.spark.xml") \
    .options(rowTag="book").load("sample/xml/")
{% endhighlight %}

The bigger datasets are, the longer you wait. Even if you need only the first record from the file, Spark  (by default) reads its whole content to create a valid schema consisting of the superset of used fields and their types. Let's see how to improve the process with three simple hints.

## Hint #1: play with `samplingRatio`

If your datasets have mostly static schema, there is no need to read all the data. You can speed up loading files with `samplingRatio` option for [JSON](https://github.com/apache/spark/blob/v2.2.0/sql/core/src/main/scala/org/apache/spark/sql/execution/datasources/json/JsonUtils.scala#L42) and [XML](https://github.com/databricks/spark-xml/blob/v0.4.1/src/main/scala/com/databricks/spark/xml/util/InferSchema.scala#L69) readers - the value is from range `(0,1]` and specifies what fraction of data will be loaded by scheme inferring job.

{% highlight python %}
# JSON
spark.read.options(samplingRatio=0.1).json("sample/json/")

# XML
spark.read.format("com.databricks.spark.xml") \
    .options(rowTag="book") \
    .options(samplingRatio=0.1) \
    .load("sample/xml/")

# similiar option for CSV does not exist :-(
{% endhighlight %}

Now the process of loading files is faster, but it still could be better. What's more, if your code relies on the schema and schema in the files changes (it's allowed in semi-structured formats), the application may fail - Spark will not be able to build execution plan. I faced the problem many times maintaining ETLs based on daily partitioned datasets in JSON.

## Hint #2: define static schema

The solution to these problems already exists in spark codebase - all mentioned DataFrame readers take the `schema` parameter. If you pass the schema, Spark context will not need to read underlying data to create DataFrames. Still, definifing schema is a very tedious job...

{% highlight python %}
from pyspark.sql.types import *

# This is a simplified schema of stackoverflow's posts collection
schema = StructType([
    StructField('Id', IntegerType()),
    StructField('AcceptedAnswerId', IntegerType()),
    StructField('AnswerCount', IntegerType()),
    StructField('ClosedDate', TimestampType()),
    StructField('CommentCount', IntegerType()),
    StructField('CreationDate', TimestampType()),
    StructField('FavoriteCount', IntegerType()),
    StructField('LastActivityDate', TimestampType()),
    StructField('OwnerDisplayName', StringType()),
    StructField('OwnerUserId', IntegerType()),
    StructField('ParentId', IntegerType()),
    StructField('Score', IntegerType()),
    StructField('Title', StringType()),
    StructField('ViewCount', IntegerType())
])

# JSON
df = spark.read.json("sample/json/", schema=schema)

# CSV
df = spark.read.csv("sample/csv/", schema=schema, header=True)

# XML
df = spark.read.format("com.databricks.spark.xml") \
       .options(rowTag="post").load("sample/xml/", schema=schema)
{% endhighlight %}

Now loading the files is as fast as possible - Spark won't run any job to create dataframes. For complex datasets schema definition may take even hunderts of code lines and it's very easy to make a mistake when writing them. The datasets metadata in application code doesn't really look like an elegant way to handle the problem.

## Hint #3: store schemas outside code

Fortunately, schemas are serializable objects and they serialize nicely to python dictionaries using standard pyspark library:

{% highlight python %}
> import json
> print(schema.json())
{"fields":[{"metadata":{},"name":"Id","nullable":true,"type":"integer"},{"metadata":{},"name":"AcceptedAnswerId","nullable":true,"type":"integer"} ...
> print(json.dumps(schema.jsonValue(), indent=2))
{
  "fields": [
    {
      "name": "Id",
      "nullable": true,
      "metadata": {},
      "type": "integer"
    },
    {
      "name": "AcceptedAnswerId",
      "nullable": true,
      "metadata": {},
      "type": "integer"
    },
    ...
}
{% endhighlight %}

If you paste the JSON output (compressed one, from `schema.json()`) into the file, you will be able to re-create schema objects based on the data using the following instructions:

{% highlight python %}
schema_json = spark.read.text("/.../sample.schema").first()[0]
schema = StructType.fromJson(json.loads(schema_json))
{% endhighlight %}

Using this trick you can easily store schemas on filesystem supported by spark (HDFS, local, S3, ...) and load them into the applications using a very quick job. 

## Getting it all together

Reading semi-structured files in Spark can be efficient if you know the schema before accesing the data. But defining the schema manually is hard and tedious... 

Next time you are building ETL application based on CSV, JSON or XML files, try the following approach:

1. Locate a small, representative subset of input data (so that it contains a superset of possible fields and their types). If you want to be 100% sure that you don't miss any field, use whole dataset. For really big but consistent sources consider using `samplingRatio` parameter.
2. Load the above dataset as dataframe and extract JSON representation of the schema into a file. If you store data on distributed storage like HDFS or S3, it's good to store this file there, too.
3. In your application add a code that reads schema file into a variable.
4. Load your input dataset passing schema parameter pointing to the variable.

And finally, a handy class that can simplify the procedure:

{% highlight python %}
import json
from pyspark.sql.types import StructType

class SchemaTools:
    def __init__(self, spark):
        self.spark = spark
    
    def dump(self, df):
        return df.schema.json()
    
    def load(self, path):
        schema_json = self.spark.read.text(path).first()[0]
        return StructType.fromJson(json.loads(schema_json))
    
schema_tools = SchemaTools(spark)

# Dump df dataframe schema into JSON string
print(schema_tools.dump(df))

# Load HDFS file /schema_json/sample.schema into schema object
schema = schema_tools.load("hdfs:///schema_json/sample.schema")
{% endhighlight %}

> All above code is pyspark 2.X. If you still codeg in pyspark 1.X, replace `spark` with `sqlContext`. Some of above snippets may even work in scala ;-)
