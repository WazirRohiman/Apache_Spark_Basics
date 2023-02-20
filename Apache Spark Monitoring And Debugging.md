# Apache Spark Monitoring and Debugging

This tutorial will cover the following:
- Initialize a Spark Standalone Cluster with a Master and one Worker.
- Start a PySpark shell that connects to the cluster and open the Spark Application Web UI to monitor it. 
- Use the terminal to run commands and docker-based containers to launch the Spark processes.

### Get the code
```
wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-BD0225EN-SkillsNetwork/labs/data/cars.csv
```

## **Initialize the cluster**

### Stop any previously running containers with the command below
```
for i in `docker ps | awk '{print $1}' | grep -v CONTAINER`; do docker kill $i; done
```

### Remove any previously used containers
```
docker rm spark-master spark-worker-1 spark-worker-2
```

### Start Spark Master server
```
docker run \
    --name spark-master \
    -h spark-master \
    -e ENABLE_INIT_DAEMON=false \
    -p 4040:4040 \
    -p 8080:8080 \
    -v `pwd`:/home/root \
    -d bde2020/spark-master:3.1.1-hadoop3.2
```

### Start Spark Worker that will connect to the Master server
```
docker run \
    --name spark-worker-1 \
    --link spark-master:spark-master \
    -e ENABLE_INIT_DAEMON=false \
    -p 8081:8081 \
    -v `pwd`:/home/root \
    -d bde2020/spark-worker:3.1.1-hadoop3.2
```

## **Connect PySpark Shell to the Cluster and open UI**

### Launch PySpark  Shell in the running Spark Master Container
```
docker exec \
    -it `docker ps | grep spark-master | awk '{print $1}'` \
    /spark/bin/pyspark \
    --master spark://spark-master:7077
```

### Once the shell is launched, create a DataFrame in the shell
```
df = spark.read.csv("/home/root/cars.csv", header=True, inferSchema=True) \
    .repartition(32) \
    .cache()
df.show()
```

### Launch the UI by connecting to application port: 4040

## **Run an SQL Query and Debug in the Application UI**

### The steps are as follows
- Define a user-defined function (UDF) and run a query that results in an error. 
- Locate that error in the application UI and find the root cause. 
- Finally, correct the error and re-run the query.

### Run the (faulty) SQL query
Define a UDF(user-defined function) to show engine type. Use the following udf
```
from pyspark.sql.functions import udf
import time

@udf("string")
def engine(cylinders):
    time.sleep(0.2)  # Intentionally delay task
    eng = {6: "V6", 8: "V8"}
    return eng[cylinders]
```

Add the UDF as a column in the dataframe
```
df = df.withColumn("engine", engine("cylinders"))
```

Group the DataFrame by 'cylinders' and aggregate the other columns
```
dfg = df.groupby(cylinders)
```
```
dfa = dfg.agg({"mpg": "avg", "engine": "first"})
```
```
dfa.show()
```

#### The queries above will have failed. Located the errors in application UI and determine the root cause

## **Debug the error in the Application UI**

- Locate the failed stages in the application UI
- Read the 'stderr' of the first error in the log 

### Fix the error in the UDF
Fix the UDF by adding an entry to the dictionary of engine types and provide a
default for all other types.

```
@udf("string")
def engine(cylinders):
    time.sleep(0.2)  # Intentionally delay task
    eng = {4: "inline-four", 6: "V6", 8: "V8"}
    return eng.get(cylinders, "other")
```

Re-run the queries from above
```
dfg = df.groupby(cylinders)
```
```
dfa = dfg.agg({"mpg": "avg", "engine": "first"})
```
```
dfa.show()
```

### If there are no errors, the output should look like this
```
+---------+------------------+-------------+                                    
|cylinders|          avg(mpg)|first(engine)|
+---------+------------------+-------------+
|        6|19.985714285714288|           V6|
|        3|             20.55|        other|
|        5|27.366666666666664|        other|
|        4|29.286764705882348|  inline-four|
|        8|14.963106796116506|           V8|
+---------+------------------+-------------+
```

## **Monitor Application Performance with the UI**
Scale up the application by adding a worker to the cluster. This will allow the cluster to run more tasks in parallel and improve the overall performance.

### In a new terminal, add a second worker to the cluster
```
docker run \
    --name spark-worker-2 \
    --link spark-master:spark-master \
    -e ENABLE_INIT_DAEMON=false \
    -p 8082:8082 \
    -d bde2020/spark-worker:3.1.1-hadoop3.2
```
If the command is successful there will be a single output similar to the one below
```
14dea0524afe2c3c4bf05d47d6c204f4d77198d949cbd5f4adf2269642b20f2d
```

Re-run the query and check the UI for improvement
```
dfa.show()
```




