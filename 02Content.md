# Hadoop Mastery Roadmap - Day 2
**Hadoop Architecture and HDFS Deep Dive**

## Learning Agenda (2-3 hours)
- Understand Hadoop's distributed architecture
- Master HDFS (Hadoop Distributed File System) concepts
- Install Hadoop in pseudo-distributed mode
- Perform HDFS operations using command line and Python
- Explore HDFS internals and block management

## Learning Outcomes Expected
By the end of Day 2, you should be able to:
- Explain Hadoop's master-slave architecture
- Understand HDFS components (NameNode, DataNode, Secondary NameNode)
- Install and configure Hadoop in pseudo-distributed mode
- Perform basic HDFS operations (create, read, write, delete files)
- Use Python libraries to interact with HDFS
- Monitor HDFS health and storage usage

## Materials to Read (50 minutes)

### Core Concepts
1. **Hadoop Architecture**
   - [Hadoop Architecture Overview - Apache Foundation](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)
   - [Understanding Hadoop Clusters - Cloudera](https://blog.cloudera.com/blog/2013/07/hadoop-101-what-is-hadoop/)

2. **HDFS Deep Dive**
   - [HDFS Architecture Guide - Apache](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
   - [HDFS Commands Guide - Tutorialspoint](https://www.tutorialspoint.com/hadoop/hadoop_hdfs_operations.htm)

3. **Installation Guide**
   - [Single Node Setup - Apache Hadoop](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

## Hands-on Exercises (90-100 minutes)

### Exercise 1: Hadoop Installation and Configuration (45 minutes)
**Objective**: Install Hadoop in pseudo-distributed mode on Ubuntu

**Step 1: Download and Install Hadoop**
```bash
# Switch to hadoop user
su - hadoop

# Download Hadoop (using version 3.3.4)
cd /opt
sudo wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz

# Extract Hadoop
sudo tar -xzf hadoop-3.3.4.tar.gz
sudo mv hadoop-3.3.4 hadoop
sudo chown -R hadoop:hadoop /opt/hadoop

# Set up environment variables
echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ~/.bashrc
echo 'export HADOOP_HOME=/opt/hadoop' >> ~/.bashrc
echo 'export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop' >> ~/.bashrc
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> ~/.bashrc

# Source the bashrc
source ~/.bashrc

# Verify installation
hadoop version
```

**Step 2: Configure Hadoop Files**
```bash
# Configure hadoop-env.sh
echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

# Configure core-site.xml
cat << 'EOF' > $HADOOP_HOME/etc/hadoop/core-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoop/tmp</value>
    </property>
</configuration>
EOF

# Configure hdfs-site.xml
cat << 'EOF' > $HADOOP_HOME/etc/hadoop/hdfs-site.xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop/data/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop/data/datanode</value>
    </property>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>localhost:9870</value>
    </property>
</configuration>
EOF

# Create necessary directories
sudo mkdir -p /opt/hadoop/data/namenode
sudo mkdir -p /opt/hadoop/data/datanode
sudo mkdir -p /opt/hadoop/tmp
sudo chown -R hadoop:hadoop /opt/hadoop/data
sudo chown -R hadoop:hadoop /opt/hadoop/tmp
```

**Step 3: Format NameNode and Start Services**
```bash
# Format the namenode (only do this once!)
hdfs namenode -format

# Start HDFS services
start-dfs.sh

# Verify services are running
jps

# Check HDFS web interface at http://localhost:9870
```

### Exercise 2: Basic HDFS Operations (25 minutes)
**Objective**: Master fundamental HDFS commands

**File Operations**
```bash
# Check HDFS status
hdfs dfsadmin -report

# Create directories in HDFS
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop
hdfs dfs -mkdir /user/hadoop/input
hdfs dfs -mkdir /user/hadoop/output

# List HDFS directories
hdfs dfs -ls /
hdfs dfs -ls /user/hadoop

# Create a sample file locally
echo "Hello HDFS World!" > sample.txt
echo "This is line 2" >> sample.txt
echo "Learning Hadoop is fun" >> sample.txt

# Copy file to HDFS
hdfs dfs -put sample.txt /user/hadoop/input/

# List files in HDFS
hdfs dfs -ls /user/hadoop/input/

# View file content in HDFS
hdfs dfs -cat /user/hadoop/input/sample.txt

# Copy file from HDFS to local
hdfs dfs -get /user/hadoop/input/sample.txt downloaded_sample.txt

# Check file sizes
hdfs dfs -du -h /user/hadoop/input/

# Move file within HDFS
hdfs dfs -mv /user/hadoop/input/sample.txt /user/hadoop/sample_moved.txt

# Copy file within HDFS
hdfs dfs -cp /user/hadoop/sample_moved.txt /user/hadoop/input/sample_copy.txt

# Remove file from HDFS
hdfs dfs -rm /user/hadoop/sample_moved.txt

# Remove directory from HDFS
hdfs dfs -mkdir /user/hadoop/temp
hdfs dfs -rmdir /user/hadoop/temp
```

### Exercise 3: Python HDFS Integration (30 minutes)
**Objective**: Use Python to interact with HDFS programmatically

**Install Required Python Libraries**
```bash
pip3 install hdfs3 pyhdfs pydoop
```

**Create Python Scripts for HDFS Operations**

**Script 1: `hdfs_basic_operations.py`**
```python
#!/usr/bin/env python3
"""
Basic HDFS operations using Python
"""

import subprocess
import os
from pathlib import Path

class HDFSManager:
    def __init__(self, hdfs_url="hdfs://localhost:9000"):
        self.hdfs_url = hdfs_url
        self.base_path = "/user/hadoop"
    
    def run_hdfs_command(self, command):
        """Execute HDFS command and return output"""
        try:
            result = subprocess.run(command, shell=True, capture_output=True, text=True)
            if result.returncode == 0:
                return True, result.stdout.strip()
            else:
                return False, result.stderr.strip()
        except Exception as e:
            return False, str(e)
    
    def create_directory(self, path):
        """Create directory in HDFS"""
        command = f"hdfs dfs -mkdir -p {path}"
        success, output = self.run_hdfs_command(command)
        print(f"Create directory {path}: {'Success' if success else 'Failed'}")
        return success
    
    def upload_file(self, local_path, hdfs_path):
        """Upload file to HDFS"""
        command = f"hdfs dfs -put {local_path} {hdfs_path}"
        success, output = self.run_hdfs_command(command)
        print(f"Upload {local_path} to {hdfs_path}: {'Success' if success else 'Failed'}")
        return success
    
    def download_file(self, hdfs_path, local_path):
        """Download file from HDFS"""
        command = f"hdfs dfs -get {hdfs_path} {local_path}"
        success, output = self.run_hdfs_command(command)
        print(f"Download {hdfs_path} to {local_path}: {'Success' if success else 'Failed'}")
        return success
    
    def list_files(self, path):
        """List files in HDFS directory"""
        command = f"hdfs dfs -ls {path}"
        success, output = self.run_hdfs_command(command)
        if success:
            print(f"Contents of {path}:")
            print(output)
        return success, output
    
    def read_file(self, hdfs_path):
        """Read file content from HDFS"""
        command = f"hdfs dfs -cat {hdfs_path}"
        success, output = self.run_hdfs_command(command)
        if success:
            print(f"Content of {hdfs_path}:")
            print(output)
            return output
        return None
    
    def get_file_info(self, path):
        """Get file/directory information"""
        command = f"hdfs dfs -stat '%n %o %r %u %g %b %y' {path}"
        success, output = self.run_hdfs_command(command)
        if success:
            info = output.split()
            return {
                'name': info[0],
                'owner': info[1],
                'replication': info[2],
                'user': info[3],
                'group': info[4],
                'size': info[5],
                'modification_time': ' '.join(info[6:])
            }
        return None
    
    def get_hdfs_usage(self):
        """Get HDFS usage statistics"""
        command = "hdfs dfsadmin -report"
        success, output = self.run_hdfs_command(command)
        if success:
            print("HDFS Usage Report:")
            print(output)
        return success, output

# Demo usage
if __name__ == "__main__":
    hdfs = HDFSManager()
    
    print("=== HDFS Python Operations Demo ===\n")
    
    # Create test directory
    test_dir = "/user/hadoop/python_test"
    hdfs.create_directory(test_dir)
    
    # Create sample files locally
    with open("test_file1.txt", "w") as f:
        f.write("This is test file 1\nCreated using Python\nHDFS operations demo")
    
    with open("test_file2.txt", "w") as f:
        f.write("This is test file 2\nAnother sample file\nFor HDFS testing")
    
    # Upload files
    hdfs.upload_file("test_file1.txt", f"{test_dir}/test_file1.txt")
    hdfs.upload_file("test_file2.txt", f"{test_dir}/test_file2.txt")
    
    # List files
    hdfs.list_files(test_dir)
    
    # Read file content
    hdfs.read_file(f"{test_dir}/test_file1.txt")
    
    # Get file information
    info = hdfs.get_file_info(f"{test_dir}/test_file1.txt")
    if info:
        print(f"\nFile Info: {info}")
    
    # Download file
    hdfs.download_file(f"{test_dir}/test_file1.txt", "downloaded_test_file1.txt")
    
    # Clean up local files
    os.remove("test_file1.txt")
    os.remove("test_file2.txt")
    
    print("\n=== Demo completed ===")
```

**Script 2: `hdfs_monitoring.py`**
```python
#!/usr/bin/env python3
"""
HDFS monitoring and health check script
"""

import subprocess
import json
import time
from datetime import datetime

class HDFSMonitor:
    def __init__(self):
        self.namenode_url = "http://localhost:9870"
    
    def run_command(self, command):
        """Execute command and return output"""
        try:
            result = subprocess.run(command, shell=True, capture_output=True, text=True)
            return result.returncode == 0, result.stdout.strip()
        except Exception as e:
            return False, str(e)
    
    def check_hdfs_health(self):
        """Check HDFS cluster health"""
        print("🏥 HDFS Health Check")
        print("=" * 40)
        
        # Check if services are running
        success, output = self.run_command("jps")
        if success:
            services = output.split('\n')
            namenode_running = any('NameNode' in service for service in services)
            datanode_running = any('DataNode' in service for service in services)
            
            print(f"NameNode: {'✓ Running' if namenode_running else '✗ Not Running'}")
            print(f"DataNode: {'✓ Running' if datanode_running else '✗ Not Running'}")
        
        # Check HDFS status
        success, output = self.run_command("hdfs dfsadmin -report")
        if success:
            lines = output.split('\n')
            for line in lines:
                if 'Live datanodes' in line or 'Dead datanodes' in line:
                    print(line)
        
        print()
    
    def get_storage_usage(self):
        """Get HDFS storage usage"""
        print("💾 Storage Usage")
        print("=" * 40)
        
        success, output = self.run_command("hdfs dfs -df -h")
        if success:
            print(output)
        
        print()
    
    def get_file_system_stats(self):
        """Get file system statistics"""
        print("📊 File System Statistics")
        print("=" * 40)
        
        # Count files and directories
        success, output = self.run_command("hdfs dfs -count /")
        if success:
            parts = output.split()
            if len(parts) >= 3:
                print(f"Directories: {parts[0]}")
                print(f"Files: {parts[1]}")
                print(f"Total Size: {parts[2]} bytes")
        
        print()
    
    def monitor_block_info(self):
        """Monitor HDFS block information"""
        print("🧱 Block Information")
        print("=" * 40)
        
        success, output = self.run_command("hdfs fsck / -blocks -locations")
        if success:
            lines = output.split('\n')
            total_blocks = 0
            corrupt_blocks = 0
            
            for line in lines:
                if 'Total blocks' in line:
                    print(line)
                elif 'Corrupt blocks' in line:
                    print(line)
                elif 'Status:' in line:
                    print(line)
        
        print()
    
    def generate_health_report(self):
        """Generate comprehensive health report"""
        print(f"📋 HDFS Health Report - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        print("=" * 60)
        
        self.check_hdfs_health()
        self.get_storage_usage()
        self.get_file_system_stats()
        self.monitor_block_info()
        
        # Save report to file
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        report_file = f"hdfs_health_report_{timestamp}.txt"
        
        with open(report_file, 'w') as f:
            f.write(f"HDFS Health Report - {datetime.now()}\n")
            f.write("=" * 50 + "\n\n")
            
            # Capture all monitoring data
            for method in [self.check_hdfs_health, self.get_storage_usage, 
                          self.get_file_system_stats, self.monitor_block_info]:
                try:
                    method()
                except Exception as e:
                    f.write(f"Error in {method.__name__}: {e}\n")
        
        print(f"💾 Report saved to: {report_file}")

if __name__ == "__main__":
    monitor = HDFSMonitor()
    monitor.generate_health_report()
```

## Exercise Solutions

### Solution 1: Hadoop Installation Verification
```bash
# Verify Hadoop installation
echo "=== Hadoop Installation Verification ==="
echo "Hadoop Version:"
hadoop version

echo -e "\nJava Processes:"
jps

echo -e "\nHDFS Status:"
hdfs dfsadmin -report | head -20

echo -e "\nHDFS Web Interface:"
echo "Open http://localhost:9870 in your browser"
```

### Solution 2: HDFS Operations Verification
```bash
# Create verification script
cat << 'EOF' > verify_hdfs_operations.sh
#!/bin/bash
echo "=== HDFS Operations Verification ==="

# Test file operations
echo "Testing file operations..."
echo "Sample data for testing" > test_hdfs.txt
hdfs dfs -put test_hdfs.txt /user/hadoop/
hdfs dfs -ls /user/hadoop/
hdfs dfs -cat /user/hadoop/test_hdfs.txt
hdfs dfs -rm /user/hadoop/test_hdfs.txt
rm test_hdfs.txt

echo "All operations completed successfully!"
EOF

chmod +x verify_hdfs_operations.sh
./verify_hdfs_operations.sh
```

### Solution 3: Python Integration Test
```bash
# Run the Python scripts
python3 hdfs_basic_operations.py
python3 hdfs_monitoring.py
```

## Quiz (15 minutes)

### Questions:
1. What are the three main components of HDFS architecture?
2. What is the default block size in HDFS and why is it large?
3. What is the role of Secondary NameNode?
4. How do you check if HDFS services are running?
5. What command would you use to format the NameNode?
6. What happens when you write a file to HDFS?
7. How does HDFS ensure fault tolerance?

### Answers:
1. **HDFS Components**: NameNode (master), DataNode (slaves), Secondary NameNode (checkpoint)
2. **Block Size**: 128MB by default (was 64MB in older versions). Large blocks reduce metadata overhead and improve performance for large files
3. **Secondary NameNode**: Performs periodic checkpoints of NameNode metadata, not a backup NameNode
4. **Check Services**: Use `jps` command to see Java processes including NameNode and DataNode
5. **Format NameNode**: `hdfs namenode -format` (only done once during initial setup)
6. **Writing to HDFS**: File is split into blocks, each block is replicated to multiple DataNodes based on replication factor
7. **Fault Tolerance**: Through data replication (default 3 copies) and automatic recovery mechanisms


## Day 2 Assignment: Log analyzer - Part 1 - Data Ingestion Setup)
### Assignment objective: Set up a dedicated HDFS directory for ingesting raw log data.
Task:
    Create a directory in HDFS specifically for raw log files: /user/project/raw_logs
    Create a dummy log file on your local machine (e.g., dummy_log.txt) with a few lines of sample log data.

    2025-06-07 10:00:01 INFO User 'john_doe' logged in from 192.168.1.10
    2025-06-07 10:00:05 ERROR Failed to connect to database for 'jane_smith'
    2025-06-07 10:00:10 WARN High CPU usage detected on server 'web-01'

    Upload this dummy_log.txt file into the newly created HDFS directory /user/project/raw_logs.

Outcome of assignment (Part 1): You should have a dummy_log.txt file present in the /user/project/raw_logs directory within HDFS, ready for future processing.


## Day 2 Project: HDFS File System Analyzer

### Project Objective
Create a comprehensive HDFS analysis tool that monitors file distribution, storage usage, and provides recommendations for optimization.

### Project Implementation

**Create file: `hdfs_analyzer.py`**
```python
#!/usr/bin/env python3
"""
HDFS File System Analyzer
Analyzes HDFS usage patterns and provides optimization recommendations
"""

import subprocess
import re
import json
from datetime import datetime
from collections import defaultdict, Counter
import matplotlib.pyplot as plt
import pandas as pd

class HDFSAnalyzer:
    def __init__(self):
        self.analysis_results = {}
        self.recommendations = []
    
    def run_command(self, command):
        """Execute HDFS command"""
        try:
            result = subprocess.run(command, shell=True, capture_output=True, text=True)
            return result.returncode == 0, result.stdout.strip()
        except Exception as e:
            return False, str(e)
    
    def analyze_file_sizes(self):
        """Analyze file size distribution"""
        print("📊 Analyzing file size distribution...")
        
        success, output = self.run_command("hdfs dfs -find / -type f -exec hdfs dfs -stat '%n,%b' {} \\;")
        if not success:
            return
        
        files_data = []
        total_size = 0
        
        for line in output.split('\n'):
            if line and ',' in line:
                parts = line.split(',')
                if len(parts) == 2:
                    name, size = parts[0], int(parts[1])
                    files_data.append({'name': name, 'size': size})
                    total_size += size
        
        if files_data:
            sizes = [f['size'] for f in files_data]
            self.analysis_results['file_analysis'] = {
                'total_files': len(files_data),
                'total_size': total_size,
                'avg_size': sum(sizes) / len(sizes),
                'min_size': min(sizes),
                'max_size': max(sizes)
            }
            
            # Size categories
            small_files = sum(1 for s in sizes if s < 1024*1024)  # < 1MB
            medium_files = sum(1 for s in sizes if 1024*1024 <= s < 100*1024*1024)  # 1MB-100MB
            large_files = sum(1 for s in sizes if s >= 100*1024*1024)  # >= 100MB
            
            print(f"Total Files: {len(files_data)}")
            print(f"Small Files (<1MB): {small_files}")
            print(f"Medium Files (1MB-100MB): {medium_files}")
            print(f"Large Files (>=100MB): {large_files}")
            
            if small_files > len(files_data) * 0.5:
                self.recommendations.append("Consider using HAR (Hadoop Archive) for small files to reduce NameNode memory usage")
    
    def analyze_directory_structure(self):
        """Analyze directory structure and depth"""
        print("\n🗂️  Analyzing directory structure...")
        
        success, output = self.run_command("hdfs dfs -find / -type d")
        if not success:
            return
        
        directories = output.split('\n')
        depths = [len(d.split('/')) - 1 for d in directories if d]
        
        self.analysis_results['directory_analysis'] = {
            'total_directories': len(directories),
            'max_depth': max(depths) if depths else 0,
            'avg_depth': sum(depths) / len(depths) if depths else 0
        }
        
        print(f"Total Directories: {len(directories)}")
        print(f"Maximum Depth: {max(depths) if depths else 0}")
        print(f"Average Depth: {sum(depths) / len(depths) if depths else 0:.2f}")
    
    def analyze_replication_factor(self):
        """Analyze replication factors across files"""
        print("\n🔄 Analyzing replication factors...")
        
        success, output = self.run_command("hdfs dfs -find / -type f -exec hdfs dfs -stat '%n,%r' {} \\;")
        if not success:
            return
        
        replication_data = []
        for line in output.split('\n'):
            if line and ',' in line:
                parts = line.split(',')
                if len(parts) == 2:
                    name, replication = parts[0], int(parts[1])
                    replication_data.append({'name': name, 'replication': replication})
        
        if replication_data:
            replications = [f['replication'] for f in replication_data]
            replication_counter = Counter(replications)
            
            print("Replication Factor Distribution:")
            for factor, count in sorted(replication_counter.items()):
                print(f"  Factor {factor}: {count} files")
            
            self.analysis_results['replication_analysis'] = dict(replication_counter)
    
    def analyze_storage_usage(self):
        """Analyze storage usage patterns"""
        print("\n💾 Analyzing storage usage...")
        
        success, output = self.run_command("hdfs dfs -df -h")
        if success:
            lines = output.split('\n')
            for line in lines:
                if 'hdfs://' in line:
                    parts = line.split()
                    if len(parts) >= 4:
                        filesystem, size, used, available = parts[0], parts[1], parts[2], parts[3]
                        print(f"Filesystem: {filesystem}")
                        print(f"Size: {size}, Used: {used}, Available: {available}")
                        
                        # Parse usage percentage if available
                        if len(parts) >= 5:
                            usage_percent = parts[4].replace('%', '')
                            if usage_percent.isdigit():
                                usage = int(usage_percent)
                                if usage > 80:
                                    self.recommendations.append(f"Storage usage is {usage}% - consider adding more storage or cleaning up old files")
    
    def generate_optimization_recommendations(self):
        """Generate optimization recommendations"""
        print("\n💡 Optimization Recommendations:")
        print("=" * 40)
        
        if not self.recommendations:
            self.recommendations.append("Your HDFS setup looks good! Monitor regularly for optimal performance.")
        
        for i, recommendation in enumerate(self.recommendations, 1):
            print(f"{i}. {recommendation}")
    
    def create_summary_report(self):
        """Create comprehensive summary report"""
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        report = {
            'timestamp': timestamp,
            'analysis_results': self.analysis_results,
            'recommendations': self.recommendations
        }
        
        # Save to JSON
        report_file = f"hdfs_analysis_report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        with open(report_file, 'w') as f:
            json.dump(report, f, indent=2)
        
        # Create readable report
        readable_report = f"hdfs_analysis_readable_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"
        with open(readable_report, 'w') as f:
            f.write(f"HDFS Analysis Report\n")
            f.write(f"Generated: {timestamp}\n")
            f.write("=" * 50 + "\n\n")
            
            f.write("ANALYSIS RESULTS:\n")
            for key, value in self.analysis_results.items():
                f.write(f"\n{key.upper()}:\n")
                if isinstance(value, dict):
                    for k, v in value.items():
                        f.write(f"  {k}: {v}\n")
                else:
                    f.write(f"  {value}\n")
            
            f.write(f"\nRECOMMENDATIONS:\n")
            for i, rec in enumerate(self.recommendations, 1):
                f.write(f"{i}. {rec}\n")
        
        print(f"\n📄 Reports saved:")
        print(f"  JSON: {report_file}")
        print(f"  Readable: {readable_report}")
        
        return report_file, readable_report
    
    def run_full_analysis(self):
        """Run complete HDFS analysis"""
        print("🔍 Starting HDFS Analysis...")
        print("=" * 50)
        
        self.analyze_file_sizes()
        self.analyze_directory_structure()
        self.analyze_replication_factor()
        self.analyze_storage_usage()
        self.generate_optimization_recommendations()
        
        return self.create_summary_report()

if __name__ == "__main__":
    analyzer = HDFSAnalyzer()
    
    # Create some sample data for analysis
    print("Creating sample data for analysis...")
    
    # Create test files of different sizes
    test_commands = [
        "echo 'Small file content' > small_test.txt",
        "python3 -c \"print('Medium file content\\n' * 1000)\" > medium_test.txt",
        "python3 -c \"print('Large file content\\n' * 10000)\" > large_test.txt"
    ]
    
    for cmd in test_commands:
        subprocess.run(cmd, shell=True)
    
    # Upload to HDFS
    upload_commands = [
        "hdfs dfs -mkdir -p /user/hadoop/test_data",
        "hdfs dfs -put small_test.txt /user/hadoop/test_data/",
        "hdfs dfs -put medium_test.txt /user/hadoop/test_data/",
        "hdfs dfs -put large_test.txt /user/hadoop/test_data/"
    ]
    
    for cmd in upload_commands:
        subprocess.run(cmd, shell=True)
    
    # Run analysis
    json_report, readable_report = analyzer.run_full_analysis()
    
    # Clean up local test files
    cleanup_commands = [
        "rm -f small_test.txt medium_test.txt large_test.txt"
    ]
    
    for cmd in cleanup_commands:
        subprocess.run(cmd, shell=True)
    
    print(f"\n✅ Analysis completed successfully!")
```

### Project Execution Instructions
```bash
# Make the script executable
chmod +x hdfs_analyzer.py

# Install required Python packages (if not already installed)
pip3 install matplotlib pandas

# Run the analyzer
python3 hdfs_analyzer.py

# View the generated reports
cat hdfs_analysis_readable_*.txt
```

## Expected Project Outcomes
After completing this project, you should have:
1. A deep understanding of HDFS file distribution patterns
2. Experience with HDFS monitoring and analysis
3. Practical knowledge of storage optimization techniques
4. A reusable tool for ongoing HDFS maintenance
5. Understanding of HDFS performance characteristics

## Day 2 Summary
Today you've mastered:
- Hadoop architecture and HDFS fundamentals
- Successful Hadoop installation in pseudo-distributed mode
- HDFS command-line operations
- Python integration with HDFS
- HDFS monitoring and analysis techniques
- Performance optimization concepts

## Troubleshooting Common Issues

### Issue 1: NameNode fails to start
**Solution**:
```bash
# Check logs
cat $HADOOP_HOME/logs/hadoop-*-namenode-*.log

# If safe mode issue:
hdfs dfsadmin -safemode leave

# If formatting issue, reformat (WARNING: deletes all data):
stop-dfs.sh
rm -rf /opt/hadoop/data/namenode/*
rm -rf /opt/hadoop/data/datanode/*
hdfs namenode -format
start-dfs.sh
```

### Issue 2: Permission denied errors
**Solution**:
```bash
# Fix ownership
sudo chown -R hadoop:hadoop /opt/hadoop

# Fix permissions
chmod -R 755 /opt/hadoop/data
```

### Issue 3: Port already in use
**Solution**:
```bash
# Check what's using the port
sudo netstat -tulpn | grep :9000
sudo netstat -tulpn | grep :9870

# Kill conflicting processes if necessary
sudo kill -9 <PID>
```

### Issue 4: Java path issues
**Solution**:
```bash
# Verify Java installation
java -version
which java

# Update hadoop-env.sh with correct JAVA_HOME
echo 'export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")' >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

## Performance Tips

1. **Optimize Block Size**: For large files, consider increasing block size to 256MB or 512MB
2. **Monitor NameNode Memory**: Ensure adequate RAM for NameNode (rule of thumb: 1GB RAM per million blocks)
3. **Regular FSCK**: Run `hdfs fsck /` regularly to check file system health
4. **Balance DataNodes**: Use `hdfs balancer` to distribute data evenly

## Reference Links
- [Apache Hadoop HDFS Architecture](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
- [HDFS Commands Guide](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html)
- [Hadoop Configuration](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)
- [Python HDFS Client Documentation](https://hdfs3.readthedocs.io/en/stable/)
- [HDFS Best Practices](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/admin_hdfs_best_practices.html)

## Next Day Preview
Day 3 will cover:
- YARN (Yet Another Resource Negotiator) architecture
- MapReduce programming model
- Writing your first MapReduce program in Python using MRJob
- Job submission and monitoring
- Understanding MapReduce execution flow

---
**Time Investment**: 2-3 hours
**Difficulty Level**: Intermediate
**Hands-on Percentage**: 75%
**Key Skills Gained**: HDFS mastery, Python-HDFS integration, System monitoring