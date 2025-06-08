# Hadoop Multi-Node Cluster Setup on Laptop

## Prerequisites

### System Requirements
- **RAM**: Minimum 8GB (16GB recommended)
- **Storage**: At least 50GB free space
- **CPU**: Multi-core processor (4+ cores recommended)
- **Virtualization**: VT-x/AMD-V enabled in BIOS

### Software Requirements
- **Virtualization Software**: VirtualBox or VMware Workstation
- **Operating System**: Ubuntu 20.04 LTS or CentOS 8 (for VMs)
- **Java**: OpenJDK 8 or 11
- **Hadoop**: Version 3.3.x (latest stable)

## Step 1: Create Virtual Machines

### VM Configuration (Create 3 VMs)
1. **Master Node (NameNode)**
   - RAM: 3GB
   - Storage: 20GB
   - CPU: 2 cores
   - Network: NAT + Host-only adapter

2. **Worker Node 1 (DataNode)**
   - RAM: 2GB
   - Storage: 15GB
   - CPU: 1 core
   - Network: NAT + Host-only adapter

3. **Worker Node 2 (DataNode)**
   - RAM: 2GB
   - Storage: 15GB
   - CPU: 1 core
   - Network: NAT + Host-only adapter

### VM Network Setup
1. Create Host-only network in VirtualBox:
   - File → Host Network Manager → Create
   - Configure IP range: 192.168.56.0/24

2. Assign static IPs:
   - Master: 192.168.56.10
   - Worker1: 192.168.56.11
   - Worker2: 192.168.56.12

## Step 2: Configure Operating System (On All VMs)

### Update System
```bash
sudo apt update && sudo apt upgrade -y
# For CentOS: sudo yum update -y
```

### Install Essential Packages
```bash
sudo apt install -y vim curl wget ssh rsync net-tools
```

### Configure Static IP (Ubuntu)
Edit `/etc/netplan/00-installer-config.yaml`:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses:
        - 192.168.56.10/24  # Change for each node
      dhcp4: false
```

Apply configuration:
```bash
sudo netplan apply
```

### Configure Hostname and Hosts File
```bash
# Set hostname (different for each node)
sudo hostnamectl set-hostname master  # or worker1, worker2

# Edit /etc/hosts (on all nodes)
sudo vim /etc/hosts
```

Add these lines to `/etc/hosts`:
```
192.168.56.10   master
192.168.56.11   worker1
192.168.56.12   worker2
```

## Step 3: Create Hadoop User

### Create User (On All Nodes)
```bash
sudo adduser hadoop
sudo usermod -aG sudo hadoop
su - hadoop
```

## Step 4: Configure SSH (On All Nodes)

### Generate SSH Keys
```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
```

### Copy Public Key to All Nodes
```bash
# From master node
ssh-copy-id hadoop@master
ssh-copy-id hadoop@worker1
ssh-copy-id hadoop@worker2

# From worker nodes to master
ssh-copy-id hadoop@master
```

### Test SSH Connectivity
```bash
ssh hadoop@master
ssh hadoop@worker1
ssh hadoop@worker2
```

## Step 5: Install Java (On All Nodes)

### Install OpenJDK
```bash
sudo apt install openjdk-8-jdk -y
```

### Set JAVA_HOME
```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> ~/.bashrc
source ~/.bashrc
```

### Verify Java Installation
```bash
java -version
javac -version
```

## Step 6: Download and Install Hadoop (On All Nodes)

### Download Hadoop
```bash
cd /opt
sudo wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
sudo tar -xzf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 hadoop
sudo chown -R hadoop:hadoop /opt/hadoop
```

### Set Hadoop Environment Variables
```bash
echo 'export HADOOP_HOME=/opt/hadoop' >> ~/.bashrc
echo 'export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop' >> ~/.bashrc
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> ~/.bashrc
echo 'export HADOOP_MAPRED_HOME=$HADOOP_HOME' >> ~/.bashrc
echo 'export HADOOP_COMMON_HOME=$HADOOP_HOME' >> ~/.bashrc
echo 'export HADOOP_HDFS_HOME=$HADOOP_HOME' >> ~/.bashrc
echo 'export YARN_HOME=$HADOOP_HOME' >> ~/.bashrc
source ~/.bashrc
```

## Step 7: Configure Hadoop (Master Node Only)

### Edit hadoop-env.sh
```bash
vim $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```
Add:
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

### Configure core-site.xml
```bash
vim $HADOOP_HOME/etc/hadoop/core-site.xml
```
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoop/tmp</value>
    </property>
</configuration>
```

### Configure hdfs-site.xml
```bash
vim $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop/data/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop/data/hdfs/datanode</value>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>master:9870</value>
    </property>
</configuration>
```

### Configure mapred-site.xml
```bash
vim $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

### Configure yarn-site.xml
```bash
vim $HADOOP_HOME/etc/hadoop/yarn-site.xml
```
```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

### Configure Workers File
```bash
vim $HADOOP_HOME/etc/hadoop/workers
```
Replace content with:
```
worker1
worker2
```

## Step 8: Copy Configuration to Worker Nodes

### Copy Hadoop Directory
```bash
scp -r /opt/hadoop hadoop@worker1:/opt/
scp -r /opt/hadoop hadoop@worker2:/opt/

# On worker nodes, fix ownership
sudo chown -R hadoop:hadoop /opt/hadoop
```

### Copy .bashrc
```bash
scp ~/.bashrc hadoop@worker1:~/
scp ~/.bashrc hadoop@worker2:~/
```

## Step 9: Create Required Directories (On All Nodes)

```bash
mkdir -p /opt/hadoop/tmp
mkdir -p /opt/hadoop/data/hdfs/namenode
mkdir -p /opt/hadoop/data/hdfs/datanode
```

## Step 10: Start Hadoop Cluster

### Format HDFS (Master Node Only - First Time)
```bash
hdfs namenode -format -force
```

### Start Hadoop Services (From Master Node)
```bash
# Start HDFS
start-dfs.sh

# Start YARN
start-yarn.sh

# Or start all services
start-all.sh
```

### Verify Cluster Status
```bash
# Check running processes
jps

# Check HDFS status
hdfs dfsadmin -report

# Check cluster nodes
yarn node -list
```

## Step 11: Access Web Interfaces

### URLs to Access
- **HDFS NameNode**: http://192.168.56.10:9870
- **YARN ResourceManager**: http://192.168.56.10:8088
- **MapReduce JobHistory**: http://192.168.56.10:19888

## Step 12: Test Hadoop Installation

### Create Test Directory
```bash
hdfs dfs -mkdir /test
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop
```

### Upload Test File
```bash
echo "Hello Hadoop Multi-Node Cluster" > testfile.txt
hdfs dfs -put testfile.txt /test/
hdfs dfs -ls /test/
```

### Run WordCount Example
```bash
# Create input directory
hdfs dfs -mkdir /input

# Upload input files
hdfs dfs -put $HADOOP_HOME/etc/hadoop/*.xml /input/

# Run WordCount
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /input /output

# Check output
hdfs dfs -cat /output/part-r-00000
```

## Troubleshooting Tips

### Common Issues
1. **SSH Connection Issues**: Ensure passwordless SSH is configured
2. **Java Path Issues**: Check JAVA_HOME in hadoop-env.sh
3. **Port Conflicts**: Use `netstat -tlnp` to check port usage
4. **Firewall Issues**: Disable firewall or configure proper rules
5. **Memory Issues**: Adjust VM memory allocation

### Useful Commands
```bash
# Stop all services
stop-all.sh

# Check logs
tail -f $HADOOP_HOME/logs/*.log

# Check cluster health
hdfs fsck /

# Safe mode operations
hdfs dfsadmin -safemode leave
```

### Performance Optimization
- Increase VM memory if available
- Use SSD storage for better performance
- Adjust Hadoop memory settings in configuration files
- Monitor resource usage with htop

## Conclusion

You now have a fully functional Hadoop multi-node cluster running on your laptop. This setup provides hands-on experience with distributed computing concepts and Hadoop administration in a controlled environment.
