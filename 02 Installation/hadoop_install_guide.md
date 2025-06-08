# Complete Hadoop Installation Guide - Ubuntu 20.04 with JDK 11

This comprehensive guide will walk you through installing Apache Hadoop on a clean Ubuntu 20.04 system with JDK 11, from initial system preparation to running your first Hadoop cluster.

## 📋 Prerequisites

- Fresh Ubuntu 20.04 LTS installation
- Root or sudo access
- Minimum 4GB RAM (8GB recommended)
- At least 20GB free disk space
- Stable internet connection

---

## 🔧 Step 1: System Preparation and Updates

### Update System Packages

```bash
# Update package index
sudo apt update

# Upgrade existing packages
sudo apt upgrade -y

# Install essential tools
sudo apt install -y wget curl vim nano software-properties-common apt-transport-https
```

### Create Hadoop User

It's recommended to run Hadoop with a dedicated user account:

```bash
# Create hadoop user
sudo adduser hadoop

# Add hadoop user to sudo group
sudo usermod -aG sudo hadoop

# Switch to hadoop user
su - hadoop
```

---

## ☕ Step 2: Install Java Development Kit (JDK 11)

### Install OpenJDK 11

```bash
# Install OpenJDK 11
sudo apt install -y openjdk-11-jdk openjdk-11-jre

# Verify Java installation
java -version
javac -version
```

Expected output should show Java 11:
```
openjdk version "11.0.x" 2023-xx-xx
OpenJDK Runtime Environment (build 11.0.x+x-Ubuntu-xubuntu1~20.04)
OpenJDK 64-Bit Server VM (build 11.0.x+x-Ubuntu-xubuntu1~20.04, mixed mode, sharing)
```

### Set JAVA_HOME Environment Variable

```bash
# Find Java installation path
sudo find /usr -name "java" -type f 2>/dev/null | grep -E "jdk.*bin/java$"

# Typically it's: /usr/lib/jvm/java-11-openjdk-amd64
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

# Verify JAVA_HOME
echo $JAVA_HOME
$JAVA_HOME/bin/java -version
```

---

## 🔐 Step 3: Configure SSH for Hadoop

Hadoop requires SSH access to manage its nodes:

### Install SSH Server

```bash
# Install OpenSSH server
sudo apt install -y openssh-server

# Start and enable SSH service
sudo systemctl start ssh
sudo systemctl enable ssh

# Check SSH status
sudo systemctl status ssh
```

### Setup SSH Key-based Authentication

```bash
# Generate SSH key pair (press Enter for all prompts)
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa

# Add public key to authorized keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 0600 ~/.ssh/authorized_keys

# Test SSH connection
ssh localhost
# Type 'yes' when prompted, then exit
exit
```

---

## 📦 Step 4: Download and Install Hadoop

### Download Hadoop

```bash
# Navigate to home directory
cd ~

# Download Hadoop 3.3.6 (latest stable version)
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz

# Verify download (optional but recommended)
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz.sha512
sha512sum -c hadoop-3.3.6.tar.gz.sha512
```

### Extract and Install Hadoop

```bash
# Extract Hadoop
tar -xzf hadoop-3.3.6.tar.gz

# Move to standard location
sudo mv hadoop-3.3.6 /opt/hadoop

# Change ownership to hadoop user
sudo chown -R hadoop:hadoop /opt/hadoop

# Create symbolic link for easier updates
sudo ln -s /opt/hadoop /opt/hadoop-current
```

---

## ⚙️ Step 5: Configure Environment Variables

### Edit Bash Profile

```bash
# Edit .bashrc file
nano ~/.bashrc

# Add the following lines at the end of the file:
```

Add these lines to `~/.bashrc`:

```bash
# Java Environment
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

# Hadoop Environment
export HADOOP_HOME=/opt/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

### Apply Environment Variables

```bash
# Reload bash profile
source ~/.bashrc

# Verify environment variables
echo $HADOOP_HOME
echo $JAVA_HOME

# Test Hadoop installation
hadoop version
```

---

## 🔧 Step 6: Configure Hadoop

### Configure Java Path in Hadoop

```bash
# Edit hadoop-env.sh
nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh

# Find the line with JAVA_HOME and update it:
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

### Configure Core Site

```bash
# Edit core-site.xml
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Replace the content with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoop/logs/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:54310</value>
        <description>The name of the default file system.</description>
    </property>
</configuration>
```

### Configure HDFS Site

```bash
# Create necessary directories
sudo mkdir -p /opt/hadoop/logs/tmp
sudo mkdir -p /home/hadoop/hdfs/namenode
sudo mkdir -p /home/hadoop/hdfs/datanode
sudo chown -R hadoop:hadoop /home/hadoop/hdfs
sudo chown -R hadoop:hadoop /opt/hadoop/logs

# Edit hdfs-site.xml
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Replace the content with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>file:///home/hadoop/hdfs/namenode</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>file:///home/hadoop/hdfs/datanode</value>
    </property>
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
</configuration>
```

### Configure MapReduce Site

```bash
# Edit mapred-site.xml
nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

Add the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
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

### Configure YARN Site

```bash
# Edit yarn-site.xml
nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

Replace the content with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

---

## 🚀 Step 7: Initialize and Start Hadoop

### Format HDFS Namenode

```bash
# Format the namenode (only needed for first-time setup)
hdfs namenode -format -y

# You should see "Storage directory has been successfully formatted"
```

### Start Hadoop Services

```bash
# Start HDFS services
start-dfs.sh

# Start YARN services
start-yarn.sh

# Alternatively, start all services at once
# start-all.sh
```

### Verify Hadoop Services

```bash
# Check running Java processes
jps

# You should see processes like:
# - NameNode
# - DataNode
# - SecondaryNameNode
# - ResourceManager
# - NodeManager
```

Expected output:
```
12345 NameNode
12346 DataNode
12347 SecondaryNameNode
12348 ResourceManager
12349 NodeManager
12350 Jps
```

---

## 🌐 Step 8: Access Hadoop Web Interfaces

### Configure Firewall (if enabled)

```bash
# Check if UFW is active
sudo ufw status

# If active, allow Hadoop ports
sudo ufw allow 9870
sudo ufw allow 8088
sudo ufw allow 19888
```

### Access Web UIs

Open your web browser and navigate to:

- **HDFS NameNode UI**: http://localhost:9870
- **YARN ResourceManager UI**: http://localhost:8088
- **MapReduce JobHistory Server**: http://localhost:19888

---

## ✅ Step 9: Test Hadoop Installation

### Create Test Directories in HDFS

```bash
# Create user directory in HDFS
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop

# Create input directory for testing
hdfs dfs -mkdir /user/hadoop/input
```

### Run WordCount Example

```bash
# Create a test file
echo "Hello World Hadoop" > ~/testfile.txt
echo "Hello Hadoop World" >> ~/testfile.txt

# Copy test file to HDFS
hdfs dfs -put ~/testfile.txt /user/hadoop/input/

# Run WordCount example
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar wordcount /user/hadoop/input /user/hadoop/output

# View results
hdfs dfs -cat /user/hadoop/output/part-r-00000
```

Expected output:
```
Hadoop  2
Hello   2
World   2
```

### Test HDFS Commands

```bash
# List files in HDFS
hdfs dfs -ls /

# Check HDFS status
hdfs dfsadmin -report

# Create a directory
hdfs dfs -mkdir /user/hadoop/test

# Copy file from local to HDFS
hdfs dfs -put /etc/passwd /user/hadoop/test/

# List files
hdfs dfs -ls /user/hadoop/test/

# View file content
hdfs dfs -cat /user/hadoop/test/passwd
```

---

## 🛑 Step 10: Managing Hadoop Services

### Stop Hadoop Services

```bash
# Stop YARN services
stop-yarn.sh

# Stop HDFS services
stop-dfs.sh

# Or stop all services at once
# stop-all.sh
```

### Start Services on Boot (Optional)

Create systemd service files for automatic startup:

```bash
# Create Hadoop service file
sudo nano /etc/systemd/system/hadoop.service
```

Add the following content:

```ini
[Unit]
Description=Hadoop Service
After=network.target

[Service]
Type=forking
User=hadoop
Group=hadoop
ExecStart=/opt/hadoop/sbin/start-all.sh
ExecStop=/opt/hadoop/sbin/stop-all.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable the service:

```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable Hadoop service
sudo systemctl enable hadoop

# Start Hadoop service
sudo systemctl start hadoop

# Check status
sudo systemctl status hadoop
```

---

## 🔧 Troubleshooting Common Issues

### Issue 1: Java Process Not Starting

**Solution**:
```bash
# Check Java installation
java -version

# Verify JAVA_HOME
echo $JAVA_HOME

# Check hadoop-env.sh configuration
grep JAVA_HOME $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

### Issue 2: Permission Denied Errors

**Solution**:
```bash
# Fix ownership
sudo chown -R hadoop:hadoop /opt/hadoop
sudo chown -R hadoop:hadoop /home/hadoop/hdfs

# Fix permissions
chmod 755 /home/hadoop/hdfs/namenode
chmod 755 /home/hadoop/hdfs/datanode
```

### Issue 3: SSH Connection Issues

**Solution**:
```bash
# Test SSH
ssh localhost

# If fails, regenerate SSH keys
rm ~/.ssh/id_rsa*
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Issue 4: Port Already in Use

**Solution**:
```bash
# Check what's using the port
sudo netstat -tulpn | grep :9000

# Kill the process if needed
sudo kill -9 <PID>

# Or change ports in configuration files
```

---

## 📚 Additional Configuration Options

### Enable JobHistory Server

```bash
# Start JobHistory Server
mapred --daemon start historyserver

# Add to startup script
echo "mapred --daemon start historyserver" >> $HADOOP_HOME/sbin/start-all.sh
```

### Configure Memory Settings

Edit `yarn-site.xml` for memory optimization:

```xml
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>4096</value>
</property>
<property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>4096</value>
</property>
<property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>512</value>
</property>
```

---

## 🎯 Next Steps

After successful installation, consider:

1. **Learn HDFS Commands**: Practice file operations in HDFS
2. **Explore MapReduce**: Run built-in examples and create custom jobs
3. **Install Ecosystem Tools**: Add Hive, Pig, Spark, or HBase
4. **Configure Multi-node Cluster**: Expand to multiple machines
5. **Security Configuration**: Implement Kerberos authentication
6. **Performance Tuning**: Optimize for your specific use case

---

## 📖 Useful Commands Reference

### HDFS Commands
```bash
hdfs dfs -ls /                    # List directory contents
hdfs dfs -mkdir /directory        # Create directory
hdfs dfs -put localfile /hdfs/    # Copy from local to HDFS
hdfs dfs -get /hdfs/file local/   # Copy from HDFS to local
hdfs dfs -rm /hdfs/file          # Remove file
hdfs dfs -rmdir /hdfs/directory  # Remove directory
```

### Hadoop Administration
```bash
hadoop version                    # Check Hadoop version
hdfs dfsadmin -report            # HDFS cluster status
yarn node -list                 # List YARN nodes
yarn application -list           # List running applications
```

### Log Locations
- **Hadoop Logs**: `/opt/hadoop/logs/`
- **HDFS Logs**: `/opt/hadoop/logs/hadoop-hadoop-*.log`
- **YARN Logs**: `/opt/hadoop/logs/yarn-hadoop-*.log`

---

## 🆘 Support and Resources

- **Official Documentation**: https://hadoop.apache.org/docs/
- **Community Support**: https://hadoop.apache.org/mailing_lists.html
- **Stack Overflow**: Tag your questions with `apache-hadoop`
- **GitHub Issues**: https://github.com/apache/hadoop

Remember to backup your configuration files before making changes, and always test in a development environment before applying to production systems.

---

**Congratulations! You now have a fully functional Hadoop installation on Ubuntu 20.04 with JDK 11.** 🎉