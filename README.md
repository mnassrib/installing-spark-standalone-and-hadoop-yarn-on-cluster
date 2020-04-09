# Installing Spark on Hadoop Yarn Multi Node Cluster

> This repository describes all the required steps to install Spark on Hadoop Yarn multi node cluster.

> **To start this tutorial, we need a ready-made hadoop cluster. For this, we can provide the cluster that we described in detail in a previous tutorial: [Installing Hadoop on single node as well multi node cluster using Debian 9 Linux VMs][verifsudo].** 

[verifsudo]: https://github.com/mnassrib/installing-hadoop-cluster




## 1- Using Hadoop User		       	
> login as hdpuser user

``hdpuser@master-node:~$``

## 2- Installing Anaconda3 on both servers (master-node & slave-node-1)	

``hdpuser@master-node:~$ cd /bigdata``	       	

- Download Anaconda version "Anaconda3-2020.02-Linux-x86_64.sh", and follow installation steps:

``hdpuser@master-node:/bigdata$ bash Anaconda3-2020.02-Linux-x86_64.sh``
		
	In order to continue the installation process, please review the license
	agreement.
	please, press ENTER to continue
	>>> yes
>
	Do you accept the license terms? [yes|no]
	[no] >>> yes
>
	Anaconda3 will now be installed in this location:
	/root/anaconda3
	
		- Press ENTER to confirm the location
		- Press CTRL-C to abord the installation
		- Or specify a different location below
	
	[/root/anaconda3] >>> /bigdata/anaconda3
		
- Setup Environment variables
		
``hdpuser@master-node:/bigdata$ cd ~``

``hdpuser@master-node:~$ vi .bashrc``  --add the below at the end of the file
			
	# Setup Python & Anaconda Environment variables
	export PYTHONPATH=/bigdata/anaconda3/bin
	export PATH=/bigdata/anaconda3/bin:$PATH
			
``hdpuser@master-node:~$ source .bashrc`` --load the .bashrc file

``hdpuser@master-node:~$ python --version`` --to check which version of python

![python-master-node](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/python-master-node.png)

![python-slave-node-1](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/python-slave-node-1.png) 

## 3- Installing Spark on both servers (master-node & slave-node-1)

- Download Spark archive file "spark-2.4.5-bin-hadoop2.7.tar.gz", and follow installation steps:

``hdpuser@master-node:~$ cd /bigdata``
		
- Extract the archive "spark-2.4.5-bin-hadoop2.7.tar.gz", 
		
``hdpuser@master-node:/bigdata$ tar -zxvf spark-2.4.5-bin-hadoop2.7.tar.gz``
		
- Setup Environment variables 

``hdpuser@master-node:/bigdata$ cd``  --to move to your home directory

``hdpuser@master-node:~$ vi .bashrc``  --add the below at the end of the file

	# Setup SPARK Environment variables
	export SPARK_HOME=/bigdata/spark-2.4.5-bin-hadoop2.7
	export PATH=$SPARK_HOME/bin:$PATH
	export CLASSPATH=$SPARK_HOME/jars/*:$CLASSPATH
	export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH

	# Control Spark
	alias Start_SPARK='$SPARK_HOME/sbin/start-all.sh;$SPARK_HOME/sbin/start-history-server.sh'
	alias Stop_SPARK='$SPARK_HOME/sbin/stop-all.sh;$SPARK_HOME/sbin/stop-history-server.sh'

	# Setup PYSPARK Environment variables
	export PYTHONPATH=$SPARK_HOME/python/:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip:$SPARK_HOME/python/lib/pyspark.zip:$PYTHONPATH
	export PYSPARK_PYTHON=/bigdata/anaconda3/bin/python
	export PYSPARK_DRIVER_PYTHON=/bigdata/anaconda3/bin/python
	
``hdpuser@master-node:~$ source .bashrc`` --after save the .bashrc file, load it

### Add the attached Spark config files:

``hdpuser@master-node:~$ cd $SPARK_HOME/conf``  --check the environment variables you just added

- Modify file: **spark-env.sh** 

> *on master-node server* 

``hdpuser@master-node:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-env.sh``  --copy the spark-env.sh file

	#IP for Local node
	SPARK_LOCAL_IP=192.xxx.x.**1**
	HADOOP_CONF_DIR=/bigdata/hadoop-3.1.1/etc/hadoop
	YARN_CONF_DIR=/bigdata/hadoop-3.1.1/etc/hadoop
	SPARK_EXECUTOR_CORES=1
	SPARK_EXECUTOR_MEMORY=512m
	SPARK_DRIVER_MEMORY=512m

	SPARK_MASTER_HOST=192.xxx.x.1
	SPARK_MASTER_PORT=6066
	SPARK_MASTER_WEBUI_PORT=6064

	SPARK_WORKER_PORT=7077
	SPARK_WORKER_WEBUI_PORT=7074

	#PYSPARK Environment variables
	SPARK_CONF_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/conf
	SPARK_LOG_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/logs

> *on slave-node-1 server* 

``hdpuser@slave-node-1:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-env.sh``  --copy the spark-env.sh file

	#IP for Local node
	SPARK_LOCAL_IP=192.xxx.x.**2**
	HADOOP_CONF_DIR=/bigdata/hadoop-3.1.1/etc/hadoop
	YARN_CONF_DIR=/bigdata/hadoop-3.1.1/etc/hadoop
	SPARK_EXECUTOR_CORES=1
	SPARK_EXECUTOR_MEMORY=512m
	SPARK_DRIVER_MEMORY=512m

	SPARK_MASTER_HOST=192.xxx.x.1
	SPARK_MASTER_PORT=6066
	SPARK_MASTER_WEBUI_PORT=6064

	SPARK_WORKER_PORT=7077
	SPARK_WORKER_WEBUI_PORT=7074

	#PYSPARK Environment variables
	SPARK_CONF_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/conf
	SPARK_LOG_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/logs

- Modify file: **spark-defaults.conf** on both servers

``hdpuser@master-node:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-defaults.conf``  --copy the spark-defaults.conf file

	spark.master 				yarn
	spark.eventLog.enabled 			true
	spark.eventLog.dir			hdfs://master-node:9000/spark-history
	spark.yarn.historyServer.address	master-node:19888/
	spark.yarn.am.memory			512m
	spark.executor.memoryOverhead		1g
	spark.history.provider			org.apache.spark.deploy.history.FsHistoryProvider
	spark.history.ui.port			18080
	spark.history.fs.logDirectory		hdfs://master-node:9000/spark-history
	spark.driver.cores			1
	spark.driver.memory			512m
	spark.executor.instances		1
	spark.executor.memory			512m
	spark.yarn.jars				hdfs://master-node:9000/user/spark-2.4.5/jars/*
	spark.serializer                	org.apache.spark.serializer.KryoSerializer
	spark.network.timeout			800

- Modify file: **slaves** on both servers

``hdpuser@master-node:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi slaves``  --copy the slaves file

	master-node		#if this node is not a DataNode (slave), remove this line from the slaves file
	slave-node-1

- Add the below config to **yarn-site.xml** file of Hadoop on both servers   
		
``hdpuser@master-node:/bigdata/hadoop-3.1.1/etc/hadoop$ vi yarn-site.xml``  

	<property>
		<name>yarn.nodemanager.pmem-check-enabled</name>
		<value>false</value>
	</property>

	<property>
		<name>yarn.nodemanager.vmem-check-enabled</name>
		<value>false</value>
	</property>

## 4- Create the needed directories on Hadoop cluster for Spark on master-node
			
``hdpuser@master-node:~$ hdfs dfs -mkdir /spark-history/``
			
``hdpuser@master-node:~$ hdfs dfs -mkdir -p /user/spark-2.4.5/jars/``

![directories](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/directories.png)

## 5- Upload to hdfs the needed jars by Yarn & Spark

```diff
- In order pyspark running correctly on yarn, and after starting Hadoop and creating the above needed directories, it is needed to put the jar files to HDFS:
```

- Put jar files to HDFS

``hdpuser@master-node:~$ hdfs dfs -put $SPARK_HOME/jars/* /user/spark-2.4.5/jars/``

![jarfiles](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/jarfiles.png)

- Checking by running pyspark

``hdpuser@master-node:~$ pyspark``

![launchpysparkshell](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/launchpysparkshell.png)
		
## 6- Start Spark

``hdpuser@master-node:~$ Start_SPARK``

![startspark](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/startspark.png)

Using jps command to get all the details of the Java Virtual Machine Process Status:

``hdpuser@master-node:~$ jps -m``

![jpsmaster-node](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/jpsmaster-node.png)

``hdpuser@slave-node-1:~$ jps -m``

![jpsslave-node-1](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/jpsslave-node-1.png)

###### Default Web Interfaces

> Spark Master web: http://master-node:6064/

![webspark](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/webspark.png)

> Spark History Server web: http://node-master:18080 

![websparkmaster](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/websparkmaster.png)

## 7- Running spark examples

- Example 1: The Spark installation package contains sample applications using jar files, like the parallel calculation of Pi.

``hdpuser@slave-node-1:~$ spark-submit --deploy-mode cluster --class org.apache.spark.examples.SparkPi /bigdata/spark-2.4.5-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.5.jar 10``		

![exp1spark1](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp1spark1.png)
![exp1spark2](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp1spark2.png)
![exp1spark3](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp1spark3.png)

![exp1application](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp1application.png)

- Example 2: Counting the occurrences of each word in a document using pyspark program 

> The goal of this example is to count the occurrences of each word in a given document. For this, let's write a python program and save it as "wordcount.py" into this directory "/home/hdpuser/Desktop/" on the master-node server

~~~~~~~ { .python .numberLines startFrom="10" }
############## /home/hdpuser/Desktop/wordcount.py ##############
from pyspark import SparkContext
sc = SparkContext(appName="Count words")
input_file = sc.textFile("hdfs:///user/shakespeare.txt")
b = input_file.flatMap(lambda x: x.split()).map(lambda x: (x,1)).reduceByKey(lambda a,b: a+b)
b.saveAsTextFile("file:///home/hdpuser/Desktop/count_result.txt")
~~~~~~~

Download the input file "shakespeare.txt" from this [link][shakespearefile] and save it at "/home/hdpuser/Downloads" 

[shakespearefile]: https://raw.githubusercontent.com/bbejeck/hadoop-algorithms/master/src/shakespeare.txt

Put the "shakespeare.txt" file in HDFS

``hdpuser@master-node:~$ hdfs dfs -put Downloads/shakespeare.txt /user/``


1. raisin
0. pomme
	* golden
	- granny smith
	+ boskoop
0. abricot

1291\. Signature du pacte fédéral Suisse


![inputfile](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/inputfile.png)

![Avertis] Before submitting the application, check in /home/hdpuser/Desktop/ of your both workers if you already have the count_result.txt file. If that is the case overwrite it and submit the application!

[avertis]: https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/avertis.png 'Avertis'

``hdpuser@master-node:~$ spark-submit --deploy-mode cluster --master yarn /home/hdpuser/Desktop/test_spark_yarn.py``
	
![exp2spark1](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp2spark1.png)
![exp2spark2](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp2spark2.png)
![exp2spark3](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp2spark3.png)

![exp2application](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp2application.png)

![exp2sparkjobs](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/exp2sparkjobs.png)

## 8- Stop Spark

``hdpuser@master-node:~$ Stop_SPARK``

![stopspark](https://github.com/mnassrib/installing-spark-on-hadoop-yarn-cluster/blob/master/images/stopspark.png)