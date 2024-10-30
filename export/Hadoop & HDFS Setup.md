## Setup từ đầu

1. Pull ubuntu về

````shell
docker pull ubuntu:14.04
````

2. Tải về những thứ cần thiết (hadoop, openssh, openjdk)

````shell
# install openssh-server, openjdk and wget
apt-get update && apt-get install -y openssh-server openjdk-7-jdk wget

# install hadoop 2.7.2
wget https://github.com/kiwenlau/compile-hadoop/releases/download/2.7.2/hadoop-2.7.2.tar.gz && \
tar -xzvf hadoop-2.7.2.tar.gz && \
mv hadoop-2.7.2 /usr/local/hadoop && \
rm hadoop-2.7.2.tar.gz

````

3. Setup environment (chỉnh sửa file `~/.bashrc`)

````shell
# set environment variable
EXPORT JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64 
EXPORT HADOOP_HOME=/usr/local/hadoop 
EXPORT PATH=$PATH:/usr/local/hadoop/bin:/usr/local/hadoop/sbin 
````

4. Setup SSH

````shell
ssh-keygen -t rsa -f ~/.ssh/id_rsa -P '' && \
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
````

5. Setup Hadoop (tạo file DataNode, NameNode, Logs)

````shell
mkdir -p ~/hdfs/namenode && \ 
mkdir -p ~/hdfs/datanode && \
mkdir $HADOOP_HOME/logs
````

6. Điều chỉnh SSH Config (truy cập đường dẫn `~/.ssh/config`) và chạy  `vi ssh_config`

````shell
Host localhost
  StrictHostKeyChecking no

Host 0.0.0.0
  StrictHostKeyChecking no
  
Host hadoop-*
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
````

7. Setup hadoop-env (tại `/usr/local/hadoop/etc/hadoop/hadoop-env.sh`)

````shell
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Set Hadoop-specific environment variables here.

# The only required environment variable is JAVA_HOME.  All others are
# optional.  When running a distributed configuration it is best to
# set JAVA_HOME in this file, so that it is correctly defined on
# remote nodes.

# The java implementation to use.
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64

# The jsvc implementation to use. Jsvc is required to run secure datanodes
# that bind to privileged ports to provide authentication of data transfer
# protocol.  Jsvc is not required if SASL is configured for authentication of
# data transfer protocol using non-privileged ports.
#export JSVC_HOME=${JSVC_HOME}

export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-"/etc/hadoop"}

# Extra Java CLASSPATH elements.  Automatically insert capacity-scheduler.
for f in $HADOOP_HOME/contrib/capacity-scheduler/*.jar; do
  if [ "$HADOOP_CLASSPATH" ]; then
    export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$f
  else
    export HADOOP_CLASSPATH=$f
  fi
done

# The maximum amount of heap to use, in MB. Default is 1000.
#export HADOOP_HEAPSIZE=
#export HADOOP_NAMENODE_INIT_HEAPSIZE=""

# Extra Java runtime options.  Empty by default.
export HADOOP_OPTS="$HADOOP_OPTS -Djava.net.preferIPv4Stack=true"

# Command specific options appended to HADOOP_OPTS when specified
export HADOOP_NAMENODE_OPTS="-Dhadoop.security.logger=${HADOOP_SECURITY_LOGGER:-INFO,RFAS} -Dhdfs.audit.logger=${HDFS_AUDIT_LOGGER:-INFO,NullAppender} $HADOOP_NAMENODE_OPTS"
export HADOOP_DATANODE_OPTS="-Dhadoop.security.logger=ERROR,RFAS $HADOOP_DATANODE_OPTS"

export HADOOP_SECONDARYNAMENODE_OPTS="-Dhadoop.security.logger=${HADOOP_SECURITY_LOGGER:-INFO,RFAS} -Dhdfs.audit.logger=${HDFS_AUDIT_LOGGER:-INFO,NullAppender} $HADOOP_SECONDARYNAMENODE_OPTS"

export HADOOP_NFS3_OPTS="$HADOOP_NFS3_OPTS"
export HADOOP_PORTMAP_OPTS="-Xmx512m $HADOOP_PORTMAP_OPTS"

# The following applies to multiple commands (fs, dfs, fsck, distcp etc)
export HADOOP_CLIENT_OPTS="-Xmx512m $HADOOP_CLIENT_OPTS"
#HADOOP_JAVA_PLATFORM_OPTS="-XX:-UsePerfData $HADOOP_JAVA_PLATFORM_OPTS"

# On secure datanodes, user to run the datanode as after dropping privileges.
# This **MUST** be uncommented to enable secure HDFS if using privileged ports
# to provide authentication of data transfer protocol.  This **MUST NOT** be
# defined if SASL is configured for authentication of data transfer protocol
# using non-privileged ports.
export HADOOP_SECURE_DN_USER=${HADOOP_SECURE_DN_USER}

# Where log files are stored.  $HADOOP_HOME/logs by default.
#export HADOOP_LOG_DIR=${HADOOP_LOG_DIR}/$USER

# Where log files are stored in the secure data environment.
export HADOOP_SECURE_DN_LOG_DIR=${HADOOP_LOG_DIR}/${HADOOP_HDFS_USER}

###
# HDFS Mover specific parameters
###
# Specify the JVM options to be used when starting the HDFS Mover.
# These options will be appended to the options specified as HADOOP_OPTS
# and therefore may override any similar flags set in HADOOP_OPTS
#
# export HADOOP_MOVER_OPTS=""

###
# Advanced Users Only!
###

# The directory where pid files are stored. /tmp by default.
# NOTE: this should be set to a directory that can only be written to by 
#       the user that will run the hadoop daemons.  Otherwise there is the
#       potential for a symlink attack.
export HADOOP_PID_DIR=${HADOOP_PID_DIR}
export HADOOP_SECURE_DN_PID_DIR=${HADOOP_PID_DIR}

# A string representing this instance of hadoop. $USER by default.
export HADOOP_IDENT_STRING=$USER
````

8. Setup hdfs-site.xml (tại `$HADOOP_HOME/etc/hadoop/hdfs-site.xml`)

````xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///root/hdfs/namenode</value>
        <description>NameNode directory for namespace and transaction logs storage.</description>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///root/hdfs/datanode</value>
        <description>DataNode directory</description>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
</configuration>
````

9. Setup core-site.xml (tại `$HADOOP_HOME/etc/hadoop/core-site.xml`)

````xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop-master:9000/</value>
    </property>
</configuration>
````

10. Setup mapred-site.xml (Tại `$HADOOP_HOME/etc/hadoop/mapred-site.xml`)

````xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
````

11. Setup yarn-site.xml (Tại `$HADOOP_HOME/etc/hadoop/yarn-site.xml`)

````xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop-master</value>
    </property>
</configuration>
````

12. Setup slaves (tại `$HADOOP_HOME/etc/hadoop/slaves`)

````
hadoop-slave1
hadoop-slave2
hadoop-slave3
hadoop-slave4
````

13. Setup start-hadoop.sh (tại `/start-hadoop.sh`)

````
#!/bin/bash

echo -e "\n"

$HADOOP_HOME/sbin/start-dfs.sh

echo -e "\n"

$HADOOP_HOME/sbin/start-yarn.sh

echo -e "\n"
````

14. Cấp quyền cho các script

````shell
chmod +x ~/start-hadoop.sh && \
chmod +x ~/run-wordcount.sh && \
chmod +x $HADOOP_HOME/sbin/start-dfs.sh && \
chmod +x $HADOOP_HOME/sbin/start-yarn.sh 
````

15. Format NameNode

````shell
/usr/local/hadoop/bin/hdfs namenode -format
````

16. Chạy service ssh

````shell
sh -c "service ssh start; bash
````

### Kết luận:

Chúng ta có thể cho các lệnh này vào một Dockerfile cho thuận tiện cho việc sử dụng.

Bằng cách `docker pull kiwenlau/hadoop:1.0`

Và `git clone https://github.com/kiwenlau/hadoop-cluster-docker`

## Setup

1. Tạo Hadoop network

````
docker network create --driver=bridge hadoop
````

2. Khởi động Hadoop master container (Bảo đảm rằng xóa tất cả hadoop-master đang có sẵn hoặc đang chạy)

````
sudo docker rm -f hadoop-master &> /dev/null
````

3. Chạy script `sudo ./start-container.sh`, file này có nội dung

````bash
#!/bin/bash

# the default node number is 3
N=${1:-3}


# start hadoop master container
sudo docker rm -f hadoop-master &> /dev/null
echo "start hadoop-master container..."
sudo docker run -itd \
                --net=hadoop \
                -p 50070:50070 \
                -p 8088:8088 \
                --name hadoop-master \
                --hostname hadoop-master \
                kiwenlau/hadoop:1.0 &> /dev/null


# start hadoop slave container
i=1
while [ $i -lt $N ]
do
	sudo docker rm -f hadoop-slave$i &> /dev/null
	echo "start hadoop-slave$i container..."
	sudo docker run -itd \
	                --net=hadoop \
	                --name hadoop-slave$i \
	                --hostname hadoop-slave$i \
	                kiwenlau/hadoop:1.0 &> /dev/null
	i=$(( $i + 1 ))
done 

# get into hadoop master container
sudo docker exec -it hadoop-master bash
````

Về cơ bản script này sẽ chạy 3 node, trong đó có 1 node master (hadoop-master) và 2 node slaves (hadoop-slave1 và hadoop-slave2). 3 node sẽ cùng 1 network hadoop.

4. Tiếp tục chạy script `./start-hadoop.sh` với nội dung:

````bash
#!/bin/bash

echo -e "\n"

$HADOOP_HOME/sbin/start-dfs.sh

echo -e "\n"

$HADOOP_HOME/sbin/start-yarn.sh

echo -e "\n"

````

Script này sẽ khởi động yarn và start-dfs.

5. Sau khi chạy xong ta check master và các node bằng cách chạy `hadoop dfsadmin -report`

![Pasted image 20241030123515.png](image/Pasted%20image%2020241030123515.png)

Như trong hình ta có 2 node slave (datanode). 
=> Hệ phân bố
