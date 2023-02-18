# Submit an Apache Spark Application on Kubernetes

The tutorial below will cover the following:
```
    Create a Kubernetes Pod - a set of containers running inside Kubernetes - here, containing Apache Spark which we use to submit jobs against Kubernetes
    Submit Apache Spark jobs to Kubernetes
```

To be able to conduct this tutorial these are the pre-requisites:
```
    A working docker installation
    A working kubernetes installation
    The git command line tool
```

A short and quick intro on Kubernetes
```
Kubernetes is a container orchestrator which allows to schedule millions of “docker” containers on huge compute clusters containing thousands of compute nodes. Originally invented and open-sourced by Google, Kubernetes became the de-facto standard for cloud-native application development and deployment inside and outside IBM. With RedHat OpenShift, IBM is the leader in hybrid cloud Kubernetes and within the top three companies contributing to Kubernetes’ open source code base.
```

## **Setup in the CLI**

### Get the code
```
git clone https://github.com/ibm-developer-skills-network/fgskh-new_horizons.git
```

### Change directory to the newly created folder(downloaded code)
```
cd fgskh-new_horizons
```

### Add an alias for less typing
```
alias k='kubectl'
```

### Save the current namespace in an environment variable that will be used later
```
my_namespace=$(kubectl config view --minify -o jsonpath='{..namespace}')
```

## **Deploy Apache Spark Kubernetes Pod**

In the CLI
### Install Apache Spark POD
```
k apply -f spark/pod_spark.yaml
```

### Check status of the pod
```
k get po
```

### Check output. If output is as follows then it was successful
```
NAME  READY   STATUS    RESTARTS   AGE
spark 2/2     Running   0          10m
```

### If you get an error message you can delete the pod and recreate a pod using the command above to install the Apache Spark pod
```
k delete po spark
```

## **Submit Apache Spark Job to Kubernetes**
Run a command inside the 'spark' container of this Pod.
The command exec is told to provide access to the container called spark (-c). With –- we execute a command, in the example below we just echo a message.
```
k exec spark -c spark  -- echo "Hello from inside the container"
```


### What just happened? 

You just ran a command in spark container residing in spark pod inside Kubernetes. We will use this container to submit Spark applications to the Kubernetes cluster. This container is based on an image with the Apache Spark distribution and the kubectl command pre-installed.

If you are interested you can have a look at the [Dockerfile](https://github.com/romeokienzler/new_horizons/blob/main/spark/Dockerfile) to understand what’s really inside.

You can also check out the [pod.yaml](https://github.com/romeokienzler/new_horizons/blob/main/spark/pod_spark.yaml). You’ll notice that it contains two containers. One is Apache Spark, another one is providing a Kubernetes Proxy - a so called side car container - allowing to interact with the Kubernetes cluster from inside a Pod.

Inside the container you can use the spark-submit command which makes use of the new native Kubernetes scheduler that has been added to Spark recently.

### Submit a SparkPi command to the cluster
```
k exec spark -c spark -- ./bin/spark-submit \
--master k8s://http://127.0.0.1:8001 \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=1 \
--conf spark.kubernetes.container.image=romeokienzler/spark-py:3.1.2 \
--conf spark.kubernetes.executor.request.cores=0.2 \
--conf spark.kubernetes.executor.limit.cores=0.3 \
--conf spark.kubernetes.driver.request.cores=0.2 \
--conf spark.kubernetes.driver.limit.cores=0.3 \
--conf spark.driver.memory=512m \
--conf spark.kubernetes.namespace=${my_namespace} \
local:///opt/spark/examples/jars/spark-examples_2.12-3.1.2.jar \
10
```
### Understanding the command above
- ```./bin/spark-submit``` is the command to submit applications to a Apache Spark cluster
- ```–master k8s://http://127.0.0.1:8001``` is the address of the Kubernetes API server - the way kubectl but also the Apache Spark native Kubernetes scheduler interacts with the Kubernetes cluster
- ```–name spark-pi``` provides a name for the job and the subsequent Pods created by the Apache Spark native Kubernetes scheduler are prefixed with that name
- ```–class org.apache.spark.examples.SparkPi``` provides the canonical name for the Spark application to run (Java package and class name)
- ```–conf spark.executor.instances=1``` tells the Apache Spark native Kubernetes scheduler how many Pods it has to create to parallelize the application. Note that on this single node development Kubernetes cluster increasing this number doesn’t make any sense (besides adding overhead for parallelization)
- ```–conf spark.kubernetes.container.image=romeokienzler/spark-py:3.1.2``` tells the Apache Spark native Kubernetes scheduler which container image it should use for creating the driver and executor Pods. This image can be custom build using the provided Dockerfiles in kubernetes/dockerfiles/spark/ and bin/docker-image-tool.sh in the Apache Spark distribution
- ```–conf spark.kubernetes.executor.limit.cores=0.3``` tells the Apache Spark native Kubernetes scheduler to set the CPU core limit to only use 0.3 core per executor Pod
- ```–conf spark.kubernetes.driver.limit.cores=0.3``` tells the Apache Spark native Kubernetes scheduler to set the CPU core limit to only use 0.3 core for the driver Pod
- ```–conf spark.driver.memory=512m``` tells the Apache Spark native Kubernetes scheduler to set the memory limit to only use 512MBs for the driver Pod
- ```–conf spark.kubernetes.namespace=${my_namespace}``` tells the Apache Spark native Kubernetes scheduler to set the namespace to my_namespace environment variable that we set before.
- ```local:///opt/spark/examples/jars/spark-examples_2.12-3.1.2.jar``` indicates the jar file the application is contained in. Note that the local:// prefix addresses a path within the container images provided by the spark.kubernetes.container.image option. Since we’re using a jar provided by the Apache Spark distribution this is not a problem, otherwise the spark.kubernetes.file.upload.path option has to be set and an appropriate storage subsystem has to be configured, as described in the documentation
- ```10``` tells the application to run for 10 iterations, then output the computed value of Pi

