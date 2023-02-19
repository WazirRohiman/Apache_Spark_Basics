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




