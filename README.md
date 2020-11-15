# 2020fall-aws-project

## Design structure
We will use Spark to do data analysis for covid-19.
The reasons for using Spark instead of Hadoop are:
1) Spark is faster, because it use memory dataset (called RDD) instead of HDFS(a Distributed File System) in Hadoop.
2) Spark is newer and more user friendly in Language Support (Python).

Spark cluster architecture reference :https://spark.apache.org/docs/latest/cluster-overview.html

The system currently supports several cluster managers. Though standalone type is simple, we will use Kubernetes build upon our previous work.

1) Kubernetes can manage the resources in fine grains,like cpu quotas, memory quotas etc.
2) Kubernetes can assign cluster network ip to pods(compare to docker swarm). We can use it to form our spark cluster(tens executors on our 2 vms). We only need to assign two floating ips for the console or monitor interface.

The Deployment of Spark on Kubernetes will be:
1) The Driver pod, which has SparkContext, is assigned as the master of Spark cluster
2) Several executor pods are assigned as workers of Spark cluter.

All pods can be launched by one command spark-submit as followowing:
./bin/spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///path/to/mypython.py
and the analysis begins.

## Challenge
There are some challenges we must deal with:

1) we must  supply a custom image instead of the default image from Spark package
We will install some dependencies into the image, such as the python required packages (the plotting tools and etc).
2) We will supply and config Kubernetes Volumes
We do not use HDFS. If the driver and executor pods share some disk storage , we cannot use hostPath and emptyDir. We must create our persistent Volume Claim and mount it to the pods.
We already apply a 10G volume in chameleon, but it is not claimed as persistent Volume yet.
(Maybe S3 on AWS is simpler).
3) We must deal with Authentication configurations. Since the Spark master will launch the executor pods automatically, it will use kubetnetes serverAPI and require Authentications.
4) Maybe we must supply Pod Template as well.

## Data analysis
For the data analysis，we will use Spark's MapReduce and other functionalities as the framework, design our map and reduce process. Finally, we will use plots to visualize the results.
The data is from CDC COVID-19 public site. 
 
