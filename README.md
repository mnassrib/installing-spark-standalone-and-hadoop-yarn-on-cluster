# Installing Spark on Hadoop Yarn Multi Node Cluster

> This repository describes all the required steps to install Spark on Hadoop Yarn multi node cluster.

> **To start this tutorial, we need a ready-made hadoop cluster. For this, we can provide the cluster that we described in detail in a previous tutorial: [Installing Hadoop on single node as well multi node cluster using Debian 9 Linux VMs][verifsudo].** 

[verifsudo]: https://github.com/mnassrib/installing-hadoop-cluster




## 1- Using Hadoop User		       	
> login as hdpuser user

``hdpuser@master-node:~$``

## 2- Installing Anaconda on both servers (master-node & slave-node-1)	

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

- Add the attached Spark config files:

``hdpuser@master-node:~$ cd $SPARK_HOME/conf``  --check the environment variables you just added

-- Modify file: **spark-env.sh** 

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


``hdpuser@master-node:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi spark-defaults.conf``  --copy the spark-defaults.conf file

``hdpuser@master-node:/bigdata/spark-2.4.5-bin-hadoop2.7/conf$ vi slaves``  --copy the slaves file












- Create the required directories on "Hadoop cluster" for "Spark"
			
``hdpuser@master-node:~$ hdfs dfs -mkdir /spark-history/``
			
``hdpuser@master-node:~$ hdfs dfs -mkdir -p /user/spark-2.4.5/jars/``





##############################################
##	Using Hadoop User  						##
##############################################
##	Installing Spark						##
##############################################
	Download Spark version, and follow installation steps:
		Extract the archive to installation path, ex: "tar –xzvf  spark-2.4.5-bin-hadoop2.7.tar.gz -C /bigdata"
		
		## Setup Environment variables
			cd --to move to your home directory
			vi .bash_profile -- Check the bash_profile file for variables
				# Setup SPARK Env
				export SPARK_HOME=/bigdata/spark-2.4.5-bin-hadoop2.7
				export PATH=$PATH:$SPARK_HOME/bin
				export CLASSPATH=$CLASSPATH:$SPARK_HOME/jars/*
				export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH

				# Control Spark
				alias Start_SPARK='$SPARK_HOME/sbin/start-all.sh;$SPARK_HOME/sbin/start-history-server.sh'
				alias Stop_SPARK='$SPARK_HOME/sbin/stop-all.sh;$SPARK_HOME/sbin/stop-history-server.sh'

				# Setup PYSPARK Environment variables
				export PYTHONPATH=$PYTHONPATH:$SPARK_HOME/python/:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip
				export PYSPARK_PYTHON=/bigdata/anaconda3/bin/python
				export PYSPARK_DRIVER_PYTHON=jupyter
				export PYSPARK_DRIVER_PYTHON_OPTS='notebook'
				
			--after save the bash_profile load it 
				. .bash_profile

		## Create directories on "Hadoop cluster" for "Spark"
			hdfs dfs -mkdir /spark-history/
			hdfs dfs -mkdir -p /user/spark-2.4.5/jars/
			
		## Add the attached Spark config files:
			cd $SPARK_HOME/conf
			vi spark-env.sh 				-- Copy the spark-env.sh file
			vi spark-defaults.conf 			-- Copy the spark-defaults.conf file
			vi slaves						-- Copy the slaves file
			
		## Add the below config to yarn-site.xml
<property>
	<name>yarn.nodemanager.pmem-check-enabled</name>
	<value>false</value>
</property>

<property>
	<name>yarn.nodemanager.vmem-check-enabled</name>
	<value>false</value>
</property>

############################################################################################	
		upload to hdfs the needed jars by Yarn & SPARK
		# Put jar to HDFS
			hdfs dfs -put $SPARK_HOME/jars/* /user/spark-2.4.5/jars/
############################################################################################
		## Start Spark
			Start_SPARK
			http://master-node:6064/
			
		## Stop Spark
			$Stop_SPARK