# Hadoop Mastery Week 1: Foundation & Environment Setup

## Overview
This week focuses on understanding Hadoop fundamentals and setting up your development environment. You'll get hands-on experience with basic Hadoop operations and Python integration.

---

## Day 1 (Monday) - Hadoop Fundamentals & Installation
**Time**: 2-3 hours  
**Goal**: Understand Hadoop ecosystem and set up Single Node cluster

### Learning Objectives:
- Understand what Hadoop is and why it's used
- Learn about HDFS architecture and MapReduce paradigm
- Install Hadoop on Ubuntu

### Study Materials:
- Hadoop Official Documentation: Architecture Overview
- "Hadoop: The Definitive Guide" - Chapter 1-2
- YouTube: "Hadoop Tutorial for Beginners" (any reputable channel)

### Hands-on Exercises:
1. **Environment Setup**:
   - Install Java 8/11 on Ubuntu
   - Download and install Hadoop 3.x
   - Configure SSH for localhost
   - Set up environment variables (JAVA_HOME, HADOOP_HOME)

2. **Verification Exercise**:
   - Start Hadoop services (NameNode, DataNode, ResourceManager, NodeManager)
   - Access Hadoop web interfaces (port 9870 for NameNode, 8088 for ResourceManager)
   - Verify all services are running using `jps` command

**Expected Outcomes**:
- Hadoop cluster running successfully on your Ubuntu machine
- All web interfaces accessible
- Understanding of Hadoop's distributed architecture

---

## Day 2 (Tuesday) - HDFS Deep Dive
**Time**: 2-3 hours  
**Goal**: Master HDFS operations and understand distributed file system concepts

### Learning Objectives:
- Understand HDFS architecture (NameNode, DataNode, blocks)
- Learn HDFS commands and file operations
- Understand replication and fault tolerance

### Study Materials:
- Hadoop Documentation: HDFS User Guide
- Focus on block storage, replication factor, and metadata management

### Hands-on Exercises:
1. **Basic HDFS Operations**:
   - Create directories in HDFS
   - Upload files from local filesystem to HDFS
   - List, copy, move files within HDFS
   - Download files from HDFS to local system
   - Check file permissions and ownership

2. **Advanced HDFS Operations**:
   - Set replication factor for files
   - Check file block information using `fsck` command
   - Monitor HDFS storage using web UI
   - Create and manage different directory structures

**Expected Outcomes**:
- Comfortable with all basic HDFS commands
- Understanding of how files are stored as blocks
- Ability to monitor HDFS health and storage utilization

---

## Day 3 (Wednesday) - Python Environment for Hadoop
**Time**: 2-3 hours  
**Goal**: Set up Python development environment for Hadoop integration

### Learning Objectives:
- Install and configure Python libraries for Hadoop
- Understand different ways to interact with Hadoop using Python
- Set up development tools and IDE

### Study Materials:
- PyDoop documentation
- Snakebite library documentation
- HDFS3 library documentation

### Hands-on Exercises:
1. **Python Environment Setup**:
   - Install Anaconda/Miniconda
   - Create virtual environment for Hadoop projects
   - Install key libraries: pydoop, snakebite, hdfs3, mrjob

2. **Python-HDFS Integration Test**:
   - Write Python script to connect to HDFS
   - Perform basic file operations (create, read, write, delete) using Python
   - List directories and file information programmatically
   - Handle connection errors and exceptions

**Expected Outcomes**:
- Python environment ready for Hadoop development
- Successfully connecting to HDFS using Python
- Basic HDFS operations working through Python scripts

---

## Day 4 (Thursday) - MapReduce Fundamentals
**Time**: 2-3 hours  
**Goal**: Understand MapReduce programming model and write first MapReduce program

### Learning Objectives:
- Understand Map and Reduce phases
- Learn about Combiners, Partitioners, and job workflow
- Write your first MapReduce program in Python

### Study Materials:
- MapReduce paper by Google (original paper)
- Hadoop Documentation: MapReduce Tutorial
- MRJob documentation for Python

### Hands-on Exercises:
1. **Classic Word Count Program**:
   - Write mapper function to tokenize text and emit word counts
   - Write reducer function to aggregate counts
   - Test locally first, then run on Hadoop cluster
   - Analyze input/output and job statistics

2. **Custom MapReduce Program**:
   - Create a program to find maximum temperature from weather data
   - Implement proper error handling and data validation
   - Run job with different input sizes and analyze performance

**Expected Outcomes**:
- Understanding of MapReduce execution flow
- Successfully running MapReduce jobs on your cluster
- Ability to debug and monitor job execution

---

## Day 5 (Friday) - YARN and Resource Management
**Time**: 2-3 hours  
**Goal**: Understand YARN architecture and resource management in Hadoop

### Learning Objectives:
- Learn YARN components (ResourceManager, NodeManager, ApplicationMaster)
- Understand job scheduling and resource allocation
- Monitor and manage running applications

### Study Materials:
- YARN Documentation: Architecture and concepts
- Resource scheduling algorithms (FIFO, Fair Scheduler, Capacity Scheduler)

### Hands-on Exercises:
1. **YARN Configuration**:
   - Configure memory and CPU settings for containers
   - Set up different queues for job management
   - Modify scheduler configuration

2. **Application Monitoring**:
   - Submit multiple MapReduce jobs simultaneously
   - Monitor resource utilization through YARN web UI
   - Kill and restart applications
   - Analyze application logs and troubleshoot failures

**Expected Outcomes**:
- Understanding of how YARN manages cluster resources
- Ability to configure and tune YARN settings
- Proficiency in monitoring and troubleshooting applications

---

## Weekend (Saturday-Sunday) - Integration Project
**Time**: 8-10 hours total  
**Goal**: Build end-to-end data processing pipeline

### Project: Web Log Analysis System

### Saturday (4-5 hours):
**Setup and Data Preparation**:
1. Download sample web server log files (Apache/Nginx logs)
2. Create HDFS directory structure for the project
3. Upload log files to HDFS
4. Write Python scripts to validate and clean data

**Expected Outcomes**:
- Project environment set up with real-world data
- Data quality assessment completed
- Clean datasets ready for processing

### Sunday (4-5 hours):
**Analysis Pipeline**:
1. **Top Pages Analysis**:
   - MapReduce job to find most visited pages
   - Calculate hit counts and response codes
   - Generate summary statistics

2. **User Behavior Analysis**:
   - Analyze user agents and referrer information
   - Calculate session durations and page sequences
   - Identify peak traffic hours

3. **Error Analysis**:
   - Find most common error codes
   - Identify problematic URLs
   - Generate error rate trends

**Expected Outcomes**:
- Complete data processing pipeline working
- Multiple insights extracted from web logs
- Understanding of real-world Hadoop application

---

## Week 1 Assessment Checklist:
- [ ] Hadoop cluster running smoothly
- [ ] Comfortable with HDFS operations via command line and Python
- [ ] Successfully written and executed MapReduce programs
- [ ] Understanding of YARN resource management
- [ ] Completed web log analysis project
- [ ] Can troubleshoot common Hadoop issues

## Next Week Preview:
Week 2 will focus on Hadoop ecosystem tools including Hive for SQL-like queries, Pig for data flow scripting, and HBase for NoSQL database operations.

---

**Note**: Adjust timing based on your learning pace. If you need more time on any topic, it's better to master the fundamentals before moving forward.