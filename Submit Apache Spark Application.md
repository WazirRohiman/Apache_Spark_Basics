# Submit an Apache Spark Application (from python script)

In the following tutorial will cover the following:
```
    Install a Spark Master and Worker using Docker Compose
    Create a python script containing a spark job
    Submit the job to the cluster directly from python
```

To be able to conduct this tutorial these are the pre-requisites:
```
    A working docker installation
    Docker Compose
    The git command line tool
    A python development environment
```

## **Start an Apache Spark Cluster using Docker Compose**

### Install PySpark
```
python3 -m pip install pyspark
```

### Get the latest 'docker-spark' from git
```
git clone https://github.com/big-data-europe/docker-spark.git
```

### Change the directory to the downloaded code
```
cd docker-spark
```

### Start the cluster
```
docker-compose up
```

After completion, a success message should appear similar to the one below
```
Successfully registered with master spark://<server address>:7077
```

## **Create python Code**
Code also available in a python file
```
from pyspark import SparkContext, SparkConf
from pyspark.sql import SparkSession
from pyspark.sql.types import StructField, StructType, IntegerType, StringType

sc = SparkContext.getOrCreate(SparkConf().setMaster('spark://localhost:7077'))
sc.setLogLevel("INFO")

spark = SparkSession.builder.getOrCreate()

spark = SparkSession.builder.getOrCreate()
df = spark.createDataFrame(
    [
        (1, "foo"),
        (2, "bar"),
    ],
    StructType(
        [
            StructField("id", IntegerType(), False),
            StructField("txt", StringType(), False),
        ]
    ),
)
print(df.dtypes)
df.show()
```

Run the python file. Launch the Spark UI to view the details of the worker and application





