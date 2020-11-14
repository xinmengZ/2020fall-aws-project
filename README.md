# 2020fall-aws-project

We will use Spark to do data analysis for covid-19.
The reasion of using Spark instead of Hadoop is:
1) Spark is more faster , for it use memory dataset (called RDD) instead of HDFS(a Distributed File System)
 in Hadoop
2) Spark is more newer, and more user friendly in Language Support. we wiil use python


A spark cluster architecture is as :https://spark.apache.org/docs/latest/cluster-overview.html

The system currently supports several cluster managers. Although
standalone type is simple, we will use Kubernetes based upon our previous work.

1) Kubernetes can manage the resources in fine grains,like cpu quotas, memory quotas etc.
2) Kubernetes can assign cluster network ip to pods(compare to docker swarm), we can use it to form our spark cluster(tens of executors on our 2 vms). The only two floating ip can be used as console or monitor interface.

The Deployment of Spark on Kubernetes will be:
1) The Driver pod which has SparkContext, run  as the master of Spark cluter
2) Several executor pods run as works of Spark cluter.

All these pods can be launched by one command spark-submit as follow:
./bin/spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///path/to/mypython.py
and the analysis begins.

But there are some challenges we must deal with:

1) we must  supply a custom image instead of the default image in Spark package
We will install some dependencies into the image, for example the cvs python lib, the plotting tools etc.
2) We will supply and config Kubernetes Volumes
for we do not use HDFS, if the driver and executor pods must share some disk storage , we can not use hostPath and emptyDir , we must create our persistentVolumeClaim and mount it to the pods.
we already apply a 10G volume in chameleon, but not claimed as persistentVolume yet.
Maybe S3 on AWS is simple.
3) We must deal with  Authentication configurations
for the Spark master will launch the executor pods automatically。It will use kubetnetes serverAPI, and has Authentications.
4) Maybe we must suppply Pod Template as well.

For the analysis，we will use Spark's MapReduce and others as the framework, design our map and reduce process, and plot the result user friendly.
The data is from CDC public site. 
 
