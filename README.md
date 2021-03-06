# Installing Spark Standalone and Hadoop Yarn modes on Multi-Node Cluster

> Spark supports pluggable cluster management. In this tutorial on Apache Spark cluster managers, we are going to install and using a multi-node cluster with two modes of managers ([Standalone][Spark Standalone Mode] and [YARN][Running Spark on YARN]). Standalone mode is a simple cluster manager incorporated with Spark. It makes it easy to setup a cluster that Spark itself manages and can run on Linux, Windows, or Mac OSX. Often it is the simplest way to run Spark application in a clustered environment. YARN is a software rewrite that decouples MapReduce’s resource management and scheduling capabilities from the data processing component, enabling Hadoop to support more varied processing approaches and a broader array of applications. YARN data computation framework is a combination of the ResourceManager, the NodeManager. It can run on Linux and Windows. 

In the literature and by searching on the internet, we find several sites, articles, social networks as well as academic works that speak and describe in detail Hadoop and Spark frameworks. Unfortunately, we realize that all these tools provide powerful information only on the theoretical side. Passing to practice, we discover a lot of blockages because of the lack of practical information in the literature especially regarding the deployment of Spark applications in a cluster and how can interact with Hadoop. For this reasons, I would highlight these points by deploying very simple examples. This repository describes all the required steps to install Spark Standalone and Hadoop Yarn modes on multi-node cluster.

| :point_up:    | **To start this tutorial, we need a ready-to-use Hadoop cluster. For this, we can use the cluster that we created and described in a previous tutorial: [Installing Hadoop on single node as well multi-node cluster based on VMs running Debian 9 Linux][hadooptuto]. We're going to install Spark so it will support at the same time both modes (Standalone and YARN).** |
|---------------|:------------------------|

[Spark Standalone Mode]: https://spark.apache.org/docs/latest/spark-standalone.html
[Running Spark on YARN]: https://spark.apache.org/docs/latest/running-on-yarn.html
[hadooptuto]: https://github.com/mnassrib/installing-hadoop-cluster 

## 1- Using Hadoop User		       	
> login as hdpuser user

``hdpuser@master-namenode:~$``

## 2- Installing Anaconda3 on all the servers (master-namenode & slave-datanode-1 & slave-datanode-2)	

``hdpuser@master-namenode:~$ cd /bigdata``	       	

- Download Anaconda version "[Anaconda3-2020.02-Linux-x86_64.sh][anaconda3]", and follow installation steps:

[anaconda3]: https://www.anaconda.com/distribution/

``hdpuser@master-namenode:/bigdata$ bash Anaconda3-2020.02-Linux-x86_64.sh``
		
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
		
``hdpuser@master-namenode:/bigdata$ cd ~``

``hdpuser@master-namenode:~$ vi .bashrc``  --add the below at the end of the file
			
	# Setup Python & Anaconda Environment variables
	export PYTHONPATH=/bigdata/anaconda3/bin
	export PATH=/bigdata/anaconda3/bin:$PATH
			
``hdpuser@master-namenode:~$ source .bashrc`` --load the .bashrc file

``hdpuser@master-namenode:~$ python --version`` --to check which version of python

![python-master-namenode](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/python-master-namenode.png)

![python-slave-datanode-1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/python-slave-datanode-1.png) 

![python-slave-datanode-2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/python-slave-datanode-2.png) 

## 3- Installing Spark on all the servers (master-namenode & slave-datanode-1 & slave-datanode-2)

- Download Spark archive file "[spark-2.4.5-bin-hadoop2.7.tar.gz][spark]", and follow installation steps:

[spark]: https://spark.apache.org/downloads.html

``hdpuser@master-namenode:~$ cd /bigdata``
		
- Extract the archive "spark-2.4.5-bin-hadoop2.7.tar.gz", 
		
``hdpuser@master-namenode:/bigdata$ tar -zxvf spark-2.4.5-bin-hadoop2.7.tar.gz``
		
- Setup Environment variables 

``hdpuser@master-namenode:/bigdata$ cd``  --to move to your home directory

``hdpuser@master-namenode:~$ vi .bashrc``  --add the below at the end of the file

	# Setup SPARK Environment variables
	export SPARK_HOME=/bigdata/spark-2.4.5-bin-hadoop2.7
	export PATH=$SPARK_HOME/bin:$PATH
	export PATH=$SPARK_HOME/sbin:$PATH
	export CLASSPATH=$SPARK_HOME/jars/*:$CLASSPATH
	export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH

	# Control Spark
	alias Start_SPARK='$SPARK_HOME/sbin/start-all.sh;$SPARK_HOME/sbin/start-history-server.sh'
	alias Stop_SPARK='$SPARK_HOME/sbin/stop-all.sh;$SPARK_HOME/sbin/stop-history-server.sh'

	# Setup PYSPARK Environment variables
	export PYTHONPATH=$SPARK_HOME/python/:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip:$SPARK_HOME/python/lib/pyspark.zip:$PYTHONPATH
	export PYSPARK_PYTHON=/bigdata/anaconda3/bin/python
	export PYSPARK_DRIVER_PYTHON=/bigdata/anaconda3/bin/python
	
``hdpuser@master-namenode:~$ source .bashrc`` --after save the .bashrc file, load it

### Add the attached Spark config files:

``hdpuser@master-namenode:~$ cd $SPARK_HOME/conf``  --check the environment variables you just added

- Modify file: **spark-env.sh** 

> *on master-namenode server* 

``hdpuser@master-namenode:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-env.sh``  --copy the spark-env.sh file

	#PYSPARK Environment variables
	SPARK_CONF_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/conf
	SPARK_LOG_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/logs

	#IP for Local node
	SPARK_LOCAL_IP=master-namenode #or 192.168.1.72
	HADOOP_CONF_DIR=/bigdata/hadoop-3.1.2/etc/hadoop
	YARN_CONF_DIR=/bigdata/hadoop-3.1.2/etc/hadoop
	SPARK_EXECUTOR_CORES=1
	SPARK_EXECUTOR_MEMORY=512m
	SPARK_DRIVER_MEMORY=512m

	SPARK_MASTER_HOST=master-namenode #or 192.168.1.72
	SPARK_MASTER_PORT=6066
	SPARK_MASTER_WEBUI_PORT=6064

	SPARK_WORKER_PORT=7077
	SPARK_WORKER_WEBUI_PORT=7074

> *on slave-datanode-1 server* 

``hdpuser@slave-datanode-1:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-env.sh``  --copy the spark-env.sh file

	#PYSPARK Environment variables
	SPARK_CONF_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/conf
	SPARK_LOG_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/logs

	#IP for Local node
	SPARK_LOCAL_IP=slave-datanode-1 #or 192.168.1.73
	HADOOP_CONF_DIR=/bigdata/hadoop-3.1.2/etc/hadoop
	YARN_CONF_DIR=/bigdata/hadoop-3.1.2/etc/hadoop
	SPARK_EXECUTOR_CORES=1
	SPARK_EXECUTOR_MEMORY=512m
	SPARK_DRIVER_MEMORY=512m

	SPARK_MASTER_HOST=master-namenode #or 192.168.1.72
	SPARK_MASTER_PORT=6066
	SPARK_MASTER_WEBUI_PORT=6064

	SPARK_WORKER_PORT=7077
	SPARK_WORKER_WEBUI_PORT=7074

> *on slave-datanode-2 server* 

``hdpuser@slave-datanode-2:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-env.sh``  --copy the spark-env.sh file

	#PYSPARK Environment variables
	SPARK_CONF_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/conf
	SPARK_LOG_DIR=/bigdata/spark-2.4.5-bin-hadoop2.7/logs

	#IP for Local node
	SPARK_LOCAL_IP=slave-datanode-2 #or 192.168.1.74
	HADOOP_CONF_DIR=/bigdata/hadoop-3.1.2/etc/hadoop
	YARN_CONF_DIR=/bigdata/hadoop-3.1.2/etc/hadoop
	SPARK_EXECUTOR_CORES=1
	SPARK_EXECUTOR_MEMORY=512m
	SPARK_DRIVER_MEMORY=512m

	SPARK_MASTER_HOST=master-namenode #or 192.168.1.72
	SPARK_MASTER_PORT=6066
	SPARK_MASTER_WEBUI_PORT=6064

	SPARK_WORKER_PORT=7077
	SPARK_WORKER_WEBUI_PORT=7074

- Modify file: **spark-defaults.conf** on all the servers

``hdpuser@master-namenode:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-defaults.conf``  --copy the spark-defaults.conf file

	spark.eventLog.enabled 			true
	spark.eventLog.dir			hdfs://master-namenode:9000/spark-history
	spark.yarn.historyServer.address	master-namenode:19888/
	spark.yarn.am.memory			512m
	spark.executor.memoryOverhead		1g
	spark.history.provider			org.apache.spark.deploy.history.FsHistoryProvider
	spark.history.ui.port			18080
	spark.history.fs.logDirectory		hdfs://master-namenode:9000/spark-history
	spark.driver.cores			1
	spark.driver.memory			512m
	spark.executor.instances		1
	spark.executor.memory			512m
	spark.yarn.jars				hdfs://master-namenode:9000/user/spark-2.4.5/jars/*
	spark.serializer                	org.apache.spark.serializer.KryoSerializer
	spark.network.timeout			800

- Modify file: **slaves** on only the master-namenode server

| :warning: WARNING          |
|:---------------------------|
| The goal here is to configure in particular the slaves file on the master machine. Since the master-namenode orchestrates all the workers or slaves servers, it needs to know their hostnames by mentioning them in its slaves file. About the slaves files on the slave-datanode-1 and slave-datanode-2 servers, format by leaving them empty.     |

``hdpuser@master-namenode:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi slaves``  --copy the slaves file

	master-namenode	  #remove this line from the slaves file if this node is not a worker (slave) 
	slave-datanode-1
	slave-datanode-2

- Add the below config to **yarn-site.xml** file of Hadoop on all the servers   
		
``hdpuser@master-namenode:/bigdata/hadoop-3.1.2/etc/hadoop$ vi yarn-site.xml``  

	<property>
		<name>yarn.nodemanager.pmem-check-enabled</name>
		<value>false</value>
	</property>

	<property>
		<name>yarn.nodemanager.vmem-check-enabled</name>
		<value>false</value>
	</property>

## 4- Create the needed directories on Hadoop cluster for Spark on master-namenode server

- Start Hadoop 
 
``hdpuser@master-namenode:~$ Start_HADOOP``

![starthadoop](https://github.com/mnassrib/installing-hadoop-cluster/blob/master/images/starthadoop.png)

- Create directories
			
``hdpuser@master-namenode:~$ hdfs dfs -mkdir /spark-history/``
			
``hdpuser@master-namenode:~$ hdfs dfs -mkdir -p /user/spark-2.4.5/jars/``

![directories](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/directories.png)

## 5- Upload to HDFS the requirement jars by Yarn & Spark

<table>
  <thead>
    <tr>
      <td align="left">
        :information_source: According to the spark-defaults.conf file, spark (or pyspark) can be running correctly on Yarn mode only if:
      </td>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>
        <ul>
          <li>Hadoop is started</li>
          <li>The above needed directories are created</li>
		  <li>The jar files are sent to HDFS</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

- Put jar files to HDFS

``hdpuser@master-namenode:~$ hdfs dfs -put $SPARK_HOME/jars/* /user/spark-2.4.5/jars/``

![jarfiles](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/jarfiles.png)

- Check by running pyspark

``hdpuser@master-namenode:~$ pyspark``

![launchpysparkshell](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/launchpysparkshell.png)

To exit pyspark, type exit() or press Ctrl+D
		
## 6- Start Spark

``hdpuser@master-namenode:~$ Start_SPARK``

![startspark](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/startspark.png)

Using jps command to get all the details on the Java Virtual Machine Process Status:

``hdpuser@master-namenode:~$ jps -m``

![jpsmaster-namenode](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/jpsmaster-namenode.png)

``hdpuser@slave-datanode-1:~$ jps -m``

![jpsslave-datanode-1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/jpsslave-datanode-1.png)

``hdpuser@slave-datanode-2:~$ jps -m``

![jpsslave-datanode-2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/jpsslave-datanode-2.png)

###### Default Web Interfaces

> Spark Master web: http://master-namenode:6064/

![webspark](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/webspark.png)

> Spark History Server web: http://node-master:18080 

![websparkmaster](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/websparkmaster.png)

## 7- Launching Spark examples

### 7.1- On Standalone mode

#### 7.1.1- Example 1: Calculation of Pi

> The Spark installation package contains sample applications using jar files, like the parallel calculation of Pi

``hdpuser@master-namenode:~$ spark-submit --deploy-mode client --master 'spark://master-namenode:6066' --class org.apache.spark.examples.SparkPi /bigdata/spark-2.4.5-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.5.jar 10``		

![exp1sparkstandalone1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkstandalone1.png)
![exp1sparkstandalone2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkstandalone2.png)
![exp1sparkstandalone3](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkstandalone3.png)

&nbsp;

| :memo:        | In submission on Standalone mode cases, the Spark jobs can be viewed using both manners: Spark Master Web UI or Spark History Server. However, jobs of a Spark application that is submitted on Yarn mode can be viewed using only the Spark History Server to replace the Spark Master Web UI.|
|---------------|:------------------------|

The application is available at the Spark Master Web UI into the "Completed Applications" section. 

![exp1standaloneapplication](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1standaloneapplication.png)

&nbsp;

![exp1standapplicationdetails](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1standapplicationdetails.png)

&nbsp;

Let's see the application result

![exp1standaloneresults](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1standaloneresults.png)

&nbsp;

#### 7.1.2- Example 2: Counting the occurrences of each word in a given document using pyspark program 

> The goal of this example is to count the occurrences of each word in a given document. 

1. Let's write a python program and save it as ``wordcount_master_standalone.py`` into this directory ``/home/hdpuser/Desktop/`` on the master-namenode server

~~~~~~~ { .python .numberLines startFrom="10" }
############## /home/hdpuser/Desktop/wordcount_master_standalone.py ##############
from pyspark import SparkContext
sc = SparkContext(appName="Count words deplyed on standalone mode")
input_file = sc.textFile("hdfs:///user/shakespeare.txt")
words = input_file.flatMap(lambda x: x.split())
count = words.map(lambda x: (x,1)).reduceByKey(lambda a,b: a+b)
count.saveAsTextFile("file:///home/hdpuser/Desktop/count_result_standalone")
sc.stop()
~~~~~~~

2. Download the input file ``shakespeare.txt`` from this [link][shakespearefile] and save it at ``/home/hdpuser/Downloads`` 

[shakespearefile]: https://raw.githubusercontent.com/bbejeck/hadoop-algorithms/master/src/shakespeare.txt

3. Put the ``shakespeare.txt`` file into HDFS

``hdpuser@master-namenode:~$ hdfs dfs -put Downloads/shakespeare.txt /user/``

![inputfile](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/inputfile.png)

4. Submit application

| :warning: WARNING          |
|:---------------------------|
| Before submitting the application, check in ``/home/hdpuser/Desktop/`` of your three workers if you already have the count_result_standalone directory. If that is the case overwrite it and submit the application!      |

``hdpuser@master-namenode:~$ spark-submit --deploy-mode client --master 'spark://master-namenode:6066' /home/hdpuser/Desktop/wordcount_master_standalone.py``

![exp2sparkstandalone1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2sparkstandalone1.png)
![exp2sparkstandalone2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2sparkstandalone2.png)
![exp2sparkstandalone3](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2sparkstandalone3.png)

&nbsp;

The application is available at the Spark Web UI into the "Completed Applications" section.

![exp2standaloneapplication](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2standaloneapplication.png)

&nbsp;

![exp2standaloneapplicationdetails](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2standaloneapplicationdetails.png)

5. Let's see the application results

![exp2standaloneresults1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2standaloneresults1.png)
![exp2standaloneresults2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2standaloneresults2.png)

&nbsp;

### 7.2- On Yarn mode

#### 7.2.1- Example 1: Calculation of Pi

##### - Deploy on cluster mode

``hdpuser@master-namenode:~$ spark-submit --deploy-mode cluster --master yarn --class org.apache.spark.examples.SparkPi /bigdata/spark-2.4.5-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.5.jar 10``		

![exp1sparkyarncluster1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkyarncluster1.png)
![exp1sparkyarncluster2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkyarncluster2.png)
![exp1sparkyarncluster3](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkyarncluster3.png)

&nbsp;

The application can be viewed on the ResourceManager website.

![exp1yarnclusterapplication](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclusterapplication.png)

&nbsp;

| :memo:        | Remember that Spark application jobs submitted on Yarn mode can be viewed using only the Spark History Server to replace the Spark Master Web UI.|
|---------------|:------------------------|

The application is available at Spark History Server. 

![exp1yarnclustersparkhistoryserver](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclustersparkhistoryserver.png)

&nbsp;

![exp1yarnclusterapplicationjobs](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclusterapplicationjobs.png)

Let's see the application result

![exp1yarnclusterresults](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclusterresults.png)


##### - Deploy on client mode

``hdpuser@master-namenode:~$ spark-submit --deploy-mode client --master yarn --class org.apache.spark.examples.SparkPi /bigdata/spark-2.4.5-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.4.5.jar 10``		

![exp1sparkyarnclient1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkyarnclient1.png)
![exp1sparkyarnclient2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkyarnclient2.png)
![exp1sparkyarnclient3](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1sparkyarnclient3.png)

&nbsp;

The application can be viewed on the ResourceManager website.

![exp1yarnclientapplication](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclientapplication.png)

&nbsp;

The Spark jobs are available only on the Spark History Server to replace the Spark Web UI.

![exp1yarnclientsparkhistoryserver](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclientsparkhistoryserver.png)

&nbsp;

![exp1yarnclientapplicationjobs](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclientapplicationjobs.png)

Let's see the application result

![exp1yarnclientresults](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp1yarnclientresults.png)

&nbsp;

#### 7.2.2- Example 2: Counting the occurrences of each word in a given document using pyspark program 

1. Let's rewrite the python program by changing only the application name and save it as ``wordcount_master_yarn.py`` into this directory ``/home/hdpuser/Desktop/`` on the master-namenode server

~~~~~~~ { .python .numberLines startFrom="10" }
############## /home/hdpuser/Desktop/wordcount_master_yarn.py ##############
from pyspark import SparkContext
sc = SparkContext(appName="Count words deplyed on Yarn mode")
input_file = sc.textFile("hdfs:///user/shakespeare.txt")
words = input_file.flatMap(lambda x: x.split())
count = words.map(lambda x: (x,1)).reduceByKey(lambda a,b: a+b)
count.saveAsTextFile("file:///home/hdpuser/Desktop/count_result_yarn")
sc.stop()
~~~~~~~

2. Submit application

| :warning: WARNING          |
|:---------------------------|
| Before submitting the application, check in ``/home/hdpuser/Desktop/`` of your both workers if you already have the count_result_yarn directory. If that is the case overwrite it and submit the application!      |

``hdpuser@master-namenode:~$ spark-submit --deploy-mode cluster --master yarn /home/hdpuser/Desktop/wordcount_master_yarn.py``
	
![exp2sparkyarn1](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2sparkyarn1.png)
![exp2sparkyarn2](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2sparkyarn2.png)
![exp2sparkyarn3](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2sparkyarn3.png)

&nbsp;

The application is available on the ResourceManager website.

![exp2yarnapplication](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2yarnapplication.png)

&nbsp;

Since the application is submitted on Yarn mode, the Spark jobs can be viewed using only the Spark History Server to replace the Spark Web UI.

![exp2yarnsparkhistoryserver](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2yarnsparkhistoryserver.png)

&nbsp;

![exp2yarnapplicationjobs](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2yarnapplicationjobs.png)

3. Let's see the application results

![exp2yarnresults](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/exp2yarnresults.png)

## 8- Stop Spark & Hadoop

``hdpuser@master-namenode:~$ Stop_SPARK && Stop_HADOOP``

![stopspark&hadoop](https://github.com/mnassrib/installing-spark-standalone-and-hadoop-yarn-on-cluster/blob/master/images/stopspark&hadoop.png)
