# Installing Spark on Hadoop Yarn Multi Node Cluster

> This repository describes all required steps to install Spark on Hadoop Yarn multi node cluster.

> **To start this tutorial, we need a ready-made hadoop cluster. For this, we can provide the cluster that we described in detail in a previous tutorial: [Installing Hadoop on single node as well multi node cluster using Debian 9 Linux VMs][verifsudo].** 

[verifsudo]: https://github.com/mnassrib/installing-hadoop-cluster


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