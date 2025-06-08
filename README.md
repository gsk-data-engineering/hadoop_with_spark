# HDFS In-Depth Learning Track with Hands-On Examples

## Phase 1: HDFS Administration Fundamentals (Week 1-2)

### Module 1.1: HDFS Architecture Deep Dive

#### Theory Topics
- NameNode vs SecondaryNameNode vs DataNode roles
- Block storage mechanism and replication
- Metadata management and FSImage/EditLogs
- Rack awareness and network topology
- HDFS federation concepts

#### Hands-On Exercise 1: Cluster Health Monitoring
```bash
# Check cluster status
hdfs dfsadmin -report

# Check filesystem health
hdfs fsck /

# Monitor specific directory
hdfs fsck /user/data -files -blocks -locations

# Check NameNode status
hdfs dfsadmin -printTopology

# View DataNode information
hdfs dfsadmin -listOpenFiles
```

**Practice Task**: Set up monitoring scripts to track cluster health metrics and create alerts for low disk space.

### Module 1.2: NameNode Administration

#### Theory Topics
- NameNode startup process and safe mode
- Checkpoint operations and SecondaryNameNode
- NameNode high availability (HA) setup
- Manual vs automatic failover
- Journal nodes and shared storage

#### Hands-On Exercise 2: NameNode Operations
```bash
# Enter/exit safe mode manually
hdfs dfsadmin -safemode enter
hdfs dfsadmin -safemode get
hdfs dfsadmin -safemode leave

# Force checkpoint
hdfs dfsadmin -saveNamespace

# Check NameNode logs
tail -f $HADOOP_HOME/logs/hadoop-*-namenode-*.log

# Analyze FSImage
hdfs oiv -p XML -i /opt/hadoop/data/hdfs/namenode/current/fsimage_* -o fsimage.xml
hdfs oev -p XML -i /opt/hadoop/data/hdfs/namenode/current/edits_* -o edits.xml
```

**Practice Task**: Simulate NameNode failure recovery and practice checkpoint operations.

### Module 1.3: DataNode Management

#### Theory Topics
- DataNode registration and heartbeat mechanism
- Block report generation and processing
- DataNode decommissioning process
- Disk failure handling
- DataNode security and permissions

#### Hands-On Exercise 3: DataNode Operations
```bash
# List all DataNodes
hdfs dfsadmin -listPools

# Decommission a DataNode
echo "worker2" > /opt/hadoop/etc/hadoop/exclude
hdfs dfsadmin -refreshNodes

# Check decommissioning status
hdfs dfsadmin -report

# Recommission DataNode
rm /opt/hadoop/etc/hadoop/exclude
hdfs dfsadmin -refreshNodes

# Check DataNode logs
tail -f $HADOOP_HOME/logs/hadoop-*-datanode-*.log
```

**Practice Task**: Practice adding/removing DataNodes and handling disk failures.

### Module 1.4: HDFS Configuration Management

#### Theory Topics
- Core configuration files and parameters
- Memory and performance tuning
- Replication factor management
- Block size optimization
- Quota management

#### Hands-On Exercise 4: Configuration Tuning
```bash
# Set namespace quota
hdfs dfsadmin -setQuota 100 /user/testquota
hdfs dfsadmin -setSpaceQuota 1G /user/testquota

# Check quota usage
hdfs dfs -count -q /user/testquota

# Change replication factor
hdfs dfs -setrep 3 /user/data/largefile.txt

# Check replication status
hdfs fsck /user/data/largefile.txt -files -blocks -racks
```

**Practice Task**: Create different storage policies and test quota enforcement.

---

## Phase 2: HDFS Core Operations (Week 3-4)

### Module 2.1: File System Operations Mastery

#### Theory Topics
- HDFS command-line interface deep dive
- File permissions and ACLs
- Trash management
- Snapshot operations

#### Hands-On Exercise 5: Advanced File Operations
```bash
# Create directory structure
hdfs dfs -mkdir -p /projects/{analytics,ml,etl}/{input,output,staging}

# Set permissions and ownership
hdfs dfs -chmod 755 /projects
hdfs dfs -chown hadoop:analytics /projects/analytics
hdfs dfs -chgrp ml /projects/ml

# Set ACLs
hdfs dfs -setfacl -m user:alice:rwx /projects/analytics
hdfs dfs -setfacl -m group:developers:r-x /projects/analytics
hdfs dfs -getfacl /projects/analytics

# Enable snapshots
hdfs dfsadmin -allowSnapshot /projects
hdfs dfs -createSnapshot /projects snapshot1
hdfs dfs -listSnapshots /projects
```

**Practice Task**: Implement a file archival system with snapshots and ACL-based access control.

### Module 2.2: Data Movement and Transfer

#### Theory Topics
- DistCp for large-scale data copying
- Inter-cluster data transfer
- Bandwidth throttling
- Checksum verification

#### Hands-On Exercise 6: Data Movement Operations
```bash
# Create test data
for i in {1..10}; do
    echo "Test data file $i content" > testfile$i.txt
    hdfs dfs -put testfile$i.txt /projects/analytics/input/
done

# Use DistCp for copying
hadoop distcp /projects/analytics/input /projects/analytics/backup

# Copy with bandwidth limit
hadoop distcp -bandwidth 10 /projects/analytics/input /projects/ml/input

# Verify checksums
hdfs dfs -checksum /projects/analytics/input/testfile1.txt
hdfs dfs -checksum /projects/analytics/backup/testfile1.txt

# Inter-cluster copy (if you have two clusters)
hadoop distcp hdfs://cluster1:9000/data hdfs://cluster2:9000/backup
```

**Practice Task**: Set up automated data synchronization between directories with monitoring.

### Module 2.3: Block Management and Storage

#### Theory Topics
- Block placement policies
- Erasure coding vs replication
- Storage types (RAM, SSD, DISK, ARCHIVE)
- Balancer operations

#### Hands-On Exercise 7: Block and Storage Management
```bash
# Check block information
hdfs fsck /projects/analytics/input -files -blocks -locations

# Run HDFS balancer
hdfs balancer -threshold 5

# Check balancer logs
tail -f $HADOOP_HOME/logs/hadoop-*-balancer-*.log

# Create files with different block sizes
hdfs dfs -D dfs.blocksize=67108864 -put largefile1.txt /projects/analytics/
hdfs dfs -D dfs.blocksize=134217728 -put largefile2.txt /projects/analytics/

# Set storage policy
hdfs storagepolicies -setStoragePolicy -path /projects/ml -policy HOT
hdfs storagepolicies -getStoragePolicy -path /projects/ml
```

**Practice Task**: Implement tiered storage strategy with different policies for different data types.

---

## Phase 3: HDFS Performance and Optimization (Week 5)

### Module 3.1: Performance Monitoring and Tuning

#### Theory Topics
- HDFS metrics and monitoring
- JVM tuning for NameNode and DataNode
- Network optimization
- I/O performance tuning

#### Hands-On Exercise 8: Performance Analysis
```bash
# Generate performance test data
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-*-tests.jar TestDFSIO \
    -write -nrFiles 10 -fileSize 100MB -resFile write_results.txt

# Read performance test
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-*-tests.jar TestDFSIO \
    -read -nrFiles 10 -fileSize 100MB -resFile read_results.txt

# Analyze results
hdfs dfs -cat TestDFSIO_results.log

# Monitor real-time metrics
hdfs dfsadmin -metasave metasave_output.txt
cat metasave_output.txt
```

**Practice Task**: Create performance benchmarking suite and optimize cluster based on results.

### Module 3.2: Troubleshooting and Debugging

#### Theory Topics
- Common HDFS issues and solutions
- Log analysis techniques
- Network connectivity problems
- Corrupt block recovery

#### Hands-On Exercise 9: Troubleshooting Scenarios
```bash
# Simulate corrupt blocks
# (This should be done carefully in a test environment)

# Check for corrupt blocks
hdfs fsck / -list-corruptfileblocks

# Find missing blocks
hdfs fsck -openforwrite

# Analyze NameNode heap usage
jmap -histo <namenode_pid>

# Debug slow operations
hdfs dfsadmin -printTopology
hdfs dfsadmin -fetchImage /tmp/fsimage_backup
```

**Practice Task**: Create troubleshooting runbook with common scenarios and solutions.

---

## Phase 4: Advanced HDFS Topics (Week 6-7)

### Module 4.1: HDFS High Availability and Disaster Recovery

#### Theory Topics
- NameNode HA architecture
- Automatic failover configuration
- Backup and recovery strategies
- Cross-datacenter replication

#### Hands-On Exercise 10: HA Setup Simulation
```bash
# Configure HA (requires additional setup)
# This is a simulation of commands you would use

# Check HA status
hdfs haadmin -getServiceState nn1
hdfs haadmin -getServiceState nn2

# Manual failover
hdfs haadmin -failover nn1 nn2

# Check Journal node status
hdfs haadmin -listOpenFiles
```

**Practice Task**: Design and document disaster recovery procedures.

### Module 4.2: HDFS Security Implementation

#### Theory Topics
- Kerberos authentication
- HDFS encryption at rest and in transit
- Ranger integration for authorization
- Audit logging

#### Hands-On Exercise 11: Security Configuration
```bash
# Set up basic security (without Kerberos for simplicity)

# Enable audit logging
# Add to hdfs-site.xml:
# dfs.namenode.audit.log.async = true

# Check audit logs
tail -f $HADOOP_HOME/logs/hdfs-audit.log

# Set up HTTPS for web UIs
# Configure ssl-server.xml and ssl-client.xml

# Test secure operations
hdfs dfs -ls /secure_data
```

**Practice Task**: Implement comprehensive security strategy with monitoring.

### Module 4.3: HDFS Federation and Scalability

#### Theory Topics
- NameSpace federation concepts
- ViewFS configuration
- Router-based federation
- Scaling beyond single NameNode limits

#### Hands-On Exercise 12: Federation Concepts
```bash
# Create namespace-specific directories
hdfs dfs -mkdir /federation-ns1
hdfs dfs -mkdir /federation-ns2

# Set different storage policies
hdfs storagepolicies -setStoragePolicy -path /federation-ns1 -policy HOT
hdfs storagepolicies -setStoragePolicy -path /federation-ns2 -policy COLD

# Monitor namespace usage
hdfs dfs -du -s /federation-ns1
hdfs dfs -du -s /federation-ns2
```

**Practice Task**: Design multi-tenant HDFS architecture with federation.

---

## Phase 5: HDFS Programming and Integration (Week 8)

### Module 5.1: HDFS Java API Programming

#### Theory Topics
- HDFS Java API fundamentals
- FileSystem class usage
- Streaming API for large files
- Custom InputFormat/OutputFormat

#### Hands-On Exercise 13: Java Programming
```java
// Create HDFSOperations.java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;
import java.io.*;

public class HDFSOperations {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://master:9000");
        
        FileSystem fs = FileSystem.get(conf);
        
        // Create directory
        Path dir = new Path("/java-test");
        fs.mkdirs(dir);
        
        // Write file
        Path file = new Path("/java-test/sample.txt");
        FSDataOutputStream out = fs.create(file);
        out.writeUTF("Hello HDFS from Java!");
        out.close();
        
        // Read file
        FSDataInputStream in = fs.open(file);
        String content = in.readUTF();
        System.out.println("Content: " + content);
        in.close();
        
        // List files
        FileStatus[] status = fs.listStatus(dir);
        for (FileStatus s : status) {
            System.out.println("File: " + s.getPath().getName());
        }
        
        fs.close();
    }
}
```

**Compilation and Execution:**
```bash
# Compile
javac -cp $(hadoop classpath) HDFSOperations.java

# Run
java -cp $(hadoop classpath):. HDFSOperations
```

**Practice Task**: Create a file management utility with Java API.

### Module 5.2: HDFS REST API and Integration

#### Theory Topics
- WebHDFS REST API
- HttpFS gateway
- Integration with external applications
- Authentication with REST API

#### Hands-On Exercise 14: REST API Operations
```bash
# Enable WebHDFS
# Add to hdfs-site.xml: dfs.webhdfs.enabled = true

# REST API operations
# Create directory
curl -i -X PUT "http://master:9870/webhdfs/v1/rest-test?op=MKDIRS&user.name=hadoop"

# Upload file
curl -i -X PUT -T localfile.txt \
    "http://master:9870/webhdfs/v1/rest-test/uploaded.txt?op=CREATE&user.name=hadoop"

# Read file
curl -i -L "http://master:9870/webhdfs/v1/rest-test/uploaded.txt?op=OPEN&user.name=hadoop"

# List directory
curl -i "http://master:9870/webhdfs/v1/rest-test?op=LISTSTATUS&user.name=hadoop"

# Get file status
curl -i "http://master:9870/webhdfs/v1/rest-test/uploaded.txt?op=GETFILESTATUS&user.name=hadoop"
```

**Practice Task**: Build a web application that interfaces with HDFS via REST API.

---

## Phase 6: Real-World Projects and Best Practices (Week 9-10)

### Project 1: Data Lake Implementation
**Objective**: Design and implement a data lake architecture using HDFS

**Requirements**:
- Multi-tenant environment with proper isolation
- Different retention policies for different data types
- Automated data lifecycle management
- Monitoring and alerting system

**Deliverables**:
- Architecture document
- Implementation scripts
- Monitoring dashboard
- User access documentation

### Project 2: HDFS Performance Optimization
**Objective**: Optimize an existing HDFS cluster for specific workloads

**Tasks**:
- Baseline performance measurement
- Identify bottlenecks using profiling tools
- Implement optimizations
- Measure improvements
- Document best practices

### Project 3: Disaster Recovery System
**Objective**: Implement comprehensive backup and recovery system

**Components**:
- Regular backup automation
- Cross-cluster replication
- Recovery testing procedures
- RTO/RPO measurement
- Documentation and runbooks

---

## Assessment and Certification Path

### Weekly Assessments
1. **Week 1-2**: HDFS Administration Quiz (50 questions)
2. **Week 3-4**: Practical File Operations Test
3. **Week 5**: Performance Tuning Case Study
4. **Week 6-7**: Security Implementation Project
5. **Week 8**: Programming Assessment
6. **Week 9-10**: Capstone Project Presentation

### Final Certification Requirements
- Complete all hands-on exercises
- Pass all weekly assessments (80%+ score)
- Successfully implement one major project
- Present capstone project with live demonstration

---

## Resources and References

### Essential Reading
- "Hadoop: The Definitive Guide" by Tom White (Chapters 3-5)
- Apache Hadoop Official Documentation
- HDFS Architecture Guide
- Cloudera/Hortonworks Administration Guides

### Useful Tools
- **Monitoring**: Ambari, Cloudera Manager, Ganglia
- **Performance**: HDFSTop, Hadoop Profiler
- **Development**: IntelliJ IDEA with Hadoop plugin
- **Testing**: MRUnit, Hadoop MiniCluster

### Online Resources
- Apache Hadoop Wiki
- Stack Overflow Hadoop tags
- Hadoop User Mailing Lists
- YouTube Hadoop Conference Videos

### Practice Datasets
- NYC Taxi Data
- Wikipedia Page Views
- Apache Log Files
- Stock Market Data
- Weather Station Data

---

## Daily Practice Routine

### Morning (30 minutes)
- Check cluster health
- Review overnight logs
- Practice command-line operations

### Afternoon (60 minutes)
- Work on current module exercises
- Read theory materials
- Watch relevant video tutorials

### Evening (30 minutes)
- Review learned concepts
- Update personal notes
- Plan next day's activities

---

## Success Metrics

### Technical Skills
- Execute 95% of HDFS commands without reference
- Troubleshoot common issues independently
- Optimize cluster performance by 20%+
- Implement security measures effectively

### Professional Development
- Ability to architect HDFS solutions
- Document and present technical solutions
- Mentor junior team members
- Lead HDFS migration projects

This comprehensive learning track provides a structured path from HDFS administration basics to advanced enterprise implementations, with practical hands-on experience at every step.