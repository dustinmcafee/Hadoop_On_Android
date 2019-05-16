Set up Hadoop Cluster on Ubuntu machines running over Android using CHroot method.

#########################################################################################################
Refer to https://hadoop.apache.org/docs/r2.7.5/hadoop-project-dist/hadoop-common/ClusterSetup.html	#
#########################################################################################################
Another Good Guide:											#
https://linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/			#
#########################################################################################################

To generate ssh key with no password:
ssh-keygen -f id_rsa -t rsa -N ''

Edit /etc/hosts and add ip addresses for all known nodes in all machines

Make sure ssh connection between nodes are setup without password prompt:
ssh-copy-id remotemachine_username@remotemachine

Edit /etc/hostname for master node and change from localhost to something unique: like "node-master"

Put this in ~/.bashrc file for each machine:

export JAVA_HOME=/usr/lib/jvm/default-java
export HADOOP_CLASSPATH=$JAVA_HOME/lib/tools.jar
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$JAVA_HOME/bin
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop


Edit etc/hadoop/hadoop-env.sh on each machine:
export JAVA_HOME=/usr/lib/jvm/default-java


Edit etc/hadoop/core-site.xml on each machine:
	<configuration>
	    <property>
	        <name>fs.defaultFS</name>
	        <value>hdfs://node-master:9000</value>
	    </property>
	</configuration>

Edit etc/hadoop/hdfs-site.xml on each machine:
	<configuration>
	    <property>
	        <name>dfs.replication</name>
	        <value>1</value>
	    </property>
            <property>
                <name>dfs.namenode.name.dir</name>
                <value>/usr/local/hadoop/dfs/nameNode</value>
            </property>
            <property>
                <name>dfs.datanode.data.dir</name>
                <value>/usr/local/hadoop/dfs/dataNode</value>
            </property>
            <property>
                <name>dfs.blocksize</name>
                <value>8388608</value>
            </property>
	</configuration>

Edit etc/hadoop/mapred-site.xml on each machine:
            <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
            </property>
            <property>
                <name>mapreduce.task.io.sort.mb</name>
                <value>1</value>
            </property>
            <property>
                <name>mapreduce.map.sort.spill.percent</name>
                <value>0.5</value>
            </property>
            <property>
                <name>yarn.app.mapreduce.am.resource.mb</name>
                <value>16</value>
            </property>
            <property>
                <name>mapreduce.map.memory.mb</name>
                <value>16</value>
            </property>
            <property>
                <name>mapreduce.reduce.memory.mb</name>
                <value>16</value>
            </property>
            <property>
                <name>mapred.job.tracker</name>
                <value>node-master:9001</value>
            </property>

Edit etc/hadoop/yarn-site.xml on each machine:	****DIFFERENT FOR EACH MACHINE
            <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
            </property>
            <property>
                <name>yarn.acl.enable</name>
                <value>0</value>
            </property>
            <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>node-master</value>
            </property>
            <property>
                <name>yarn.nodemanager.hostname</name>			*****Should be the name of the Slave node that this file is on, or not here at all if on node-master
                <value>node1</value>
            </property>
            <property>
                <name>yarn.nodemanager.resource.cpu-vcores</name>
                <value>4</value>
            </property>
            <property>
                <name>yarn.nodemanager.resource.memory-mb</name>
                <value>32</value>
            </property>
            <property>
                <name>yarn.scheduler.maximum-allocation-mb</name>
                <value>16</value>
            </property>
            <property>
                <name>yarn.scheduler.minimum-allocation-mb</name>
                <value>1</value>
            </property>




Command to start namenode:
hdfs namenode -format

Command to start Hadoop:
start-dfs.sh && start-yarn.sh

Commands to get hdfs directories setup:
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop_test
hdfs dfs -put patterns.txt /user/hadoop_test
hdfs dfs -put 100-0.txt /user/hadoop_test




###############################################################################################
The hadoop daemon log output is written to the $HADOOP_LOG_DIR directory (defaults to $HADOOP_HOME/logs).
Browse the web interface for the NameNode and ResourceManager; by default it is available at:
NameNode - http://node-manager:50070/
ResourceManager - http://node-manager:8088/
###############################################################################################



Command to Run precompiled mapred program from hadoop:
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.0.jar grep input output 'dfs[a-z.]+'




Commands to Compile and run java hadoop program:
hadoop com.sun.tools.javac.Main InvertedIndex.java -Xdiags:verbose
jar cf InvertedIndex.jar InvertedIndex*.class
hadoop fs -rm -r /user/hadoop_test/output ; hadoop jar InvertedIndex.jar InvertedIndex -Dinvertedindex.case.sensitive=false /user/hadoop_test/ /user/hadoop_test/output -skip /user/hadoop_test/patterns.txt
hadoop fs -cat /user/hadoop_test/output/part-r-00000



Command to stop Hadoop:
stop-dfs.sh && stop-yarn.sh







Benchmark:
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.5-tests.jar TestDFSIO -write -nrFiles 2 -fileSize 256MB

hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.5-tests.jar TestDFSIO -read -nrFiles 2 -fileSize 256MB

hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar teragen 5000000 /user/test/output		//512MB of data

time hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar terasort /user/test/output /user/test/output_sort

time hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar wordcount /user/test/input /user/test/output_wordcount
