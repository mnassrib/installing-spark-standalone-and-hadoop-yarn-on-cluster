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
	
	Do you accept the license terms? [yes|no]
	[no] >>> yes
	
	Anaconda3 will now be installed in this location:
	/root/anaconda3
	
		- Press ENTER to confirm the location
		- Press CTRL-C to abord the installation
		- Or specify a different location below
	
	[/root/anaconda3] >>> /bigdata/anaconda3
	
		
- Setup Environment variables
		
``hdpuser@master-node:/bigdata$ cd ~``

``hdpuser@master-node:~$ vi .bashrc``  --add the below at the end of the file
			
	# User specific environment and startup programs
	export PATH=$PATH:$HOME/.local/bin:$HOME/bin

	# Setup JAVA Environment variables
	export JAVA_HOME=/bigdata/jdk1.8.0_241
	export PATH=$PATH:$JAVA_HOME/bin
			
``hdpuser@master-node:~$ source .bashrc`` --load the .bashrc file



## 3- Installing Spark


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