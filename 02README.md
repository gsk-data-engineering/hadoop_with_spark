# Hadoop Mastery Roadmap - Day 1
**Introduction to Big Data and Hadoop Ecosystem**

## Learning Agenda (2-3 hours)
- Understand the fundamentals of Big Data and its characteristics
- Learn about Hadoop ecosystem and its core components
- Set up development environment on Ubuntu
- Get familiar with basic Linux commands for Hadoop

## Learning Outcomes Expected
By the end of Day 1, you should be able to:
- Define Big Data and explain the 5 V's (Volume, Velocity, Variety, Veracity, Value)
- Understand Hadoop's role in solving Big Data challenges
- Identify core Hadoop components (HDFS, YARN, MapReduce)
- Have a functional Ubuntu environment ready for Hadoop development
- Execute basic Linux commands essential for Hadoop administration

## Materials to Read (45 minutes)

### Core Concepts
1. **Big Data Fundamentals**
   - [What is Big Data? - IBM](https://www.ibm.com/topics/big-data)
   - [5 V's of Big Data Explained](https://www.guru99.com/what-is-big-data.html)

2. **Hadoop Introduction**
   - [Apache Hadoop Official Documentation - Getting Started](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)
   - [Hadoop Ecosystem Overview - Tutorialspoint](https://www.tutorialspoint.com/hadoop/hadoop_enviornment_setup.htm)

3. **Ubuntu Preparation**
   - [Linux Commands for Hadoop - Edureka](https://www.edureka.co/blog/linux-commands-for-hadoop/)

## Hands-on Exercises (60-75 minutes)

### Exercise 1: Environment Setup (30 minutes)
**Objective**: Prepare Ubuntu system for Hadoop development

**Tasks**:
1. Update your Ubuntu system
2. Install essential packages
3. Create a dedicated hadoop user
4. Set up SSH keys for passwordless authentication

**Commands to Execute**:
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y openjdk-8-jdk python3 python3-pip ssh rsync curl wget vim

# Verify Java installation
java -version
javac -version

# Create hadoop user
sudo adduser hadoop
sudo usermod -aG sudo hadoop

# Switch to hadoop user
su - hadoop

# Generate SSH keys
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys

# Test SSH
ssh localhost
```

### Exercise 2: Python Environment Setup (15 minutes)
**Objective**: Set up Python environment for Hadoop integration

**Tasks**:
```bash
# Install Python packages for Hadoop
pip3 install hdfs3 pyhive snakebite-py3 pandas numpy

# Verify installations
python3 -c "import pandas; print('Pandas version:', pandas.__version__)"
python3 -c "import numpy; print('Numpy version:', numpy.__version__)"
```

### Exercise 3: Basic Linux Commands Practice (30 minutes)
**Objective**: Master essential Linux commands for Hadoop administration

**Tasks**: Practice these commands and understand their output
```bash
# File system navigation
pwd
ls -la
cd /home
mkdir test_hadoop
cd test_hadoop

# File operations
touch sample.txt
echo "Hello Hadoop World" > sample.txt
cat sample.txt
cp sample.txt sample_copy.txt
mv sample_copy.txt renamed_sample.txt
rm renamed_sample.txt

# System information
df -h
free -h
ps aux | grep java
netstat -tulpn

# Text processing (important for log analysis)
echo -e "line1\nline2\nline3" > test.txt
grep "line2" test.txt
wc -l test.txt
head -2 test.txt
tail -1 test.txt
```

## Exercise Solutions

### Solution 1: Environment Verification
```bash
# Check if everything is installed correctly
echo "Java Version:"
java -version

echo "Python Version:"
python3 --version

echo "SSH Status:"
ssh localhost 'echo "SSH working correctly"'

echo "User Information:"
whoami
groups
```

### Solution 2: Python Package Verification
```python
# Create verification script: verify_python.py
#!/usr/bin/env python3

import sys
try:
    import pandas as pd
    print(f"✓ Pandas installed: {pd.__version__}")
except ImportError:
    print("✗ Pandas not installed")

try:
    import numpy as np
    print(f"✓ Numpy installed: {np.__version__}")
except ImportError:
    print("✗ Numpy not installed")

print(f"Python version: {sys.version}")
```

### Solution 3: Linux Commands Cheat Sheet
Create a reference file with commonly used commands:
```bash
# Create hadoop_commands.txt
cat << 'EOF' > ~/hadoop_commands.txt
# Essential Hadoop Linux Commands
# File Operations
ls -la                    # List files with details
find /path -name "*.txt"  # Find files by pattern
grep -r "pattern" /path   # Search text in files

# System Monitoring
top                       # Running processes
htop                      # Better process viewer
iostat                    # I/O statistics
df -h                     # Disk usage

# Network
netstat -tulpn           # Network connections
ss -tulpn                # Socket statistics

# Log Analysis
tail -f /path/to/log     # Follow log file
grep ERROR /path/to/log  # Find errors in logs
EOF
```

## Quiz (15 minutes)

### Questions:
1. What are the 5 V's of Big Data? Explain each briefly.
2. What is the main problem that Hadoop solves?
3. Name three core components of Hadoop ecosystem.
4. What command would you use to check available disk space on Ubuntu?
5. How do you create passwordless SSH authentication?

### Answers:
1. **5 V's of Big Data**:
   - Volume: Large amounts of data
   - Velocity: Speed of data generation and processing
   - Variety: Different types of data (structured, unstructured)
   - Veracity: Quality and accuracy of data
   - Value: Business value derived from data

2. **Main problem Hadoop solves**: Processing and storing large volumes of data across distributed systems cost-effectively

3. **Three core components**: HDFS (storage), YARN (resource management), MapReduce (processing)

4. **Check disk space**: `df -h`

5. **Passwordless SSH**: Generate keys with `ssh-keygen`, copy public key to `authorized_keys`

## Day 1 Project: System Readiness Assessment

### Project Objective
Create a comprehensive system assessment script to verify your Ubuntu system is ready for Hadoop installation.

### Project Tasks
1. Create a Python script that checks system prerequisites
2. Generate a system readiness report
3. Document any issues found and their solutions

### Project Implementation

**Create file: `hadoop_readiness_check.py`**
```python
#!/usr/bin/env python3
"""
Hadoop System Readiness Checker
Verifies that Ubuntu system meets Hadoop requirements
"""

import subprocess
import sys
import os
from pathlib import Path

class HadoopReadinessChecker:
    def __init__(self):
        self.results = {}
        self.issues = []
    
    def run_command(self, command):
        """Execute shell command and return output"""
        try:
            result = subprocess.run(command, shell=True, capture_output=True, text=True)
            return result.returncode == 0, result.stdout.strip()
        except Exception as e:
            return False, str(e)
    
    def check_java(self):
        """Check Java installation"""
        success, output = self.run_command("java -version")
        if success:
            self.results['Java'] = f"✓ Installed: {output.split()[2] if output else 'Unknown version'}"
        else:
            self.results['Java'] = "✗ Not installed"
            self.issues.append("Install Java: sudo apt install openjdk-8-jdk")
    
    def check_python(self):
        """Check Python installation"""
        version = sys.version.split()[0]
        self.results['Python'] = f"✓ Version {version}"
        
        # Check required packages
        packages = ['pandas', 'numpy']
        for package in packages:
            try:
                __import__(package)
                self.results[f'Python-{package}'] = "✓ Installed"
            except ImportError:
                self.results[f'Python-{package}'] = "✗ Missing"
                self.issues.append(f"Install {package}: pip3 install {package}")
    
    def check_ssh(self):
        """Check SSH configuration"""
        success, _ = self.run_command("ssh localhost 'echo test'")
        if success:
            self.results['SSH'] = "✓ Passwordless SSH configured"
        else:
            self.results['SSH'] = "✗ SSH not configured"
            self.issues.append("Configure SSH keys: ssh-keygen -t rsa && ssh-copy-id localhost")
    
    def check_system_resources(self):
        """Check system resources"""
        # Check RAM
        success, output = self.run_command("free -m | grep '^Mem:' | awk '{print $2}'")
        if success:
            ram_mb = int(output)
            ram_gb = ram_mb / 1024
            if ram_gb >= 4:
                self.results['RAM'] = f"✓ {ram_gb:.1f} GB available"
            else:
                self.results['RAM'] = f"⚠ {ram_gb:.1f} GB (minimum 4GB recommended)"
        
        # Check disk space
        success, output = self.run_command("df -h / | tail -1 | awk '{print $4}'")
        if success:
            self.results['Disk Space'] = f"✓ {output} available"
    
    def generate_report(self):
        """Generate comprehensive readiness report"""
        print("="*50)
        print("HADOOP SYSTEM READINESS REPORT")
        print("="*50)
        
        print("\n📋 System Check Results:")
        for component, status in self.results.items():
            print(f"  {component}: {status}")
        
        if self.issues:
            print("\n🔧 Issues to Resolve:")
            for i, issue in enumerate(self.issues, 1):
                print(f"  {i}. {issue}")
        else:
            print("\n🎉 System is ready for Hadoop installation!")
        
        print("\n📊 System Summary:")
        success_count = sum(1 for status in self.results.values() if status.startswith('✓'))
        total_checks = len(self.results)
        print(f"  Passed: {success_count}/{total_checks} checks")
        
        return len(self.issues) == 0
    
    def run_all_checks(self):
        """Run all system checks"""
        print("Running Hadoop readiness checks...\n")
        
        self.check_java()
        self.check_python()
        self.check_ssh()
        self.check_system_resources()
        
        return self.generate_report()

if __name__ == "__main__":
    checker = HadoopReadinessChecker()
    is_ready = checker.run_all_checks()
    
    # Save report to file
    with open('hadoop_readiness_report.txt', 'w') as f:
        f.write("Hadoop System Readiness Report\n")
        f.write("="*40 + "\n\n")
        for component, status in checker.results.items():
            f.write(f"{component}: {status}\n")
        
        if checker.issues:
            f.write("\nIssues to resolve:\n")
            for issue in checker.issues:
                f.write(f"- {issue}\n")
    
    print(f"\n📄 Report saved to: hadoop_readiness_report.txt")
    sys.exit(0 if is_ready else 1)
```

### Project Execution
```bash
# Make script executable
chmod +x hadoop_readiness_check.py

# Run the checker
python3 hadoop_readiness_check.py

# View the generated report
cat hadoop_readiness_report.txt
```

## Expected Project Outcomes
After completing this project, you should have:
1. A comprehensive understanding of your system's readiness for Hadoop
2. A reusable script for checking system prerequisites
3. A documented list of any issues and their solutions
4. Practical experience with Python system administration tasks

## Day 1 Summary
Today you've established the foundation for your Hadoop journey by:
- Understanding Big Data concepts and Hadoop's role
- Setting up your Ubuntu development environment
- Practicing essential Linux commands
- Creating a system readiness assessment tool

## Reference Links
- [Apache Hadoop Official Site](https://hadoop.apache.org/)
- [Hadoop Documentation](https://hadoop.apache.org/docs/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Python for System Administration](https://docs.python.org/3/library/subprocess.html)

## Next Day Preview
Day 2 will cover:
- Hadoop architecture deep dive
- HDFS (Hadoop Distributed File System) concepts
- Installing and configuring Hadoop in pseudo-distributed mode
- Basic HDFS operations with Python

---
**Time Investment**: 2-3 hours
**Difficulty Level**: Beginner
**Hands-on Percentage**: 70%