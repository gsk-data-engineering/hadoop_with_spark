# Complete Hadoop Uninstallation Guide

A comprehensive guide to completely remove Apache Hadoop and all related components from your laptop running JDK11.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Stop Hadoop Services](#stop-hadoop-services)
3. [Remove Installation Directory](#remove-installation-directory)
4. [Clean Environment Variables](#clean-environment-variables)
5. [Remove Data Directories](#remove-data-directories)
6. [Remove Configuration Files](#remove-configuration-files)
7. [Clean Temporary Files](#clean-temporary-files)
8. [Remove Hadoop User](#remove-hadoop-user)
9. [Package Manager Cleanup](#package-manager-cleanup)
10. [SSH Configuration Cleanup](#ssh-configuration-cleanup)
11. [Verification Steps](#verification-steps)
12. [Additional Cleanup](#additional-cleanup)
13. [JDK11 Considerations](#jdk11-considerations)

---

## Prerequisites

Before starting the uninstallation process:
- Ensure you have administrative privileges (sudo access)
- Back up any important data stored in HDFS
- Note down your current Hadoop installation paths
- Close any applications that might be using Hadoop

---

## Stop Hadoop Services

First, stop all running Hadoop services to prevent file locks and ensure clean removal.

```bash
# Stop all Hadoop services at once
$HADOOP_HOME/sbin/stop-all.sh

# Alternative: Stop services individually
$HADOOP_HOME/sbin/stop-dfs.sh
$HADOOP_HOME/sbin/stop-yarn.sh

# Stop JobHistory Server if running
$HADOOP_HOME/sbin/mr-jobhistory-daemon.sh stop historyserver

# Verify no Hadoop processes are running
jps
```

**Note:** If `$HADOOP_HOME` is not set, navigate to your Hadoop installation directory and run the stop scripts.

---

## Remove Installation Directory

Remove the main Hadoop installation directory and any related directories.

```bash
# Common installation locations - remove the one that applies to your setup

# If installed in /opt
sudo rm -rf /opt/hadoop*

# If installed in /usr/local
sudo rm -rf /usr/local/hadoop*

# If installed in home directory
rm -rf ~/hadoop*

# If installed in custom location, replace with your path
sudo rm -rf /path/to/your/hadoop/installation
```

---

## Clean Environment Variables

Remove Hadoop-related environment variables from your shell configuration files.

### Step 1: Identify Your Shell Configuration File
```bash
# Check your current shell
echo $SHELL

# Common configuration files:
# - Bash: ~/.bashrc, ~/.bash_profile, ~/.profile
# - Zsh: ~/.zshrc
# - Fish: ~/.config/fish/config.fish
```

### Step 2: Edit Configuration File
```bash
# For Bash users
nano ~/.bashrc

# For Zsh users
nano ~/.zshrc

# For system-wide changes
sudo nano /etc/environment
```

### Step 3: Remove These Environment Variables
Look for and remove or comment out these lines:

```bash
# Hadoop Environment Variables (REMOVE THESE)
export HADOOP_HOME=/path/to/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_STREAMING=$HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-*.jar
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

# Java settings for Hadoop (if specific to Hadoop)
export JAVA_HOME=/path/to/java
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib/native"
```

### Step 4: Reload Configuration
```bash
# Reload your shell configuration
source ~/.bashrc
# or
source ~/.zshrc

# Or restart your terminal
```

---

## Remove Data Directories

Remove all Hadoop data directories, including HDFS data, logs, and temporary files.

```bash
# Remove HDFS data directories (common locations)
sudo rm -rf /tmp/hadoop*
sudo rm -rf /var/lib/hadoop*
sudo rm -rf ~/hadoop_data
sudo rm -rf /hadoop

# Remove Hadoop logs
sudo rm -rf /var/log/hadoop*
sudo rm -rf /usr/local/hadoop/logs
sudo rm -rf /opt/hadoop/logs

# Remove namenode and datanode directories (check your hdfs-site.xml for custom paths)
sudo rm -rf /var/lib/hadoop-hdfs
sudo rm -rf /data/hadoop
sudo rm -rf /hdfs

# Remove YARN logs and data
sudo rm -rf /var/log/yarn*
sudo rm -rf /tmp/yarn*
```

**Important:** Check your `hdfs-site.xml` and `core-site.xml` files for custom data directory paths before removal.

---

## Remove Configuration Files

Remove Hadoop configuration files from system directories.

```bash
# Remove system configuration directories
sudo rm -rf /etc/hadoop*
sudo rm -rf ~/.hadoop*

# Remove any custom configuration directories
sudo rm -rf /usr/local/etc/hadoop
sudo rm -rf /opt/hadoop/etc

# Remove Hadoop-related files from /etc
sudo find /etc -name "*hadoop*" -type f -delete
```

---

## Clean Temporary Files

Remove temporary files created by Hadoop during operation.

```bash
# Remove Hadoop temporary files
sudo rm -rf /tmp/hsperfdata_*
sudo rm -rf /tmp/Jetty_*
sudo rm -rf /tmp/hadoop-*

# Remove YARN temporary files
sudo rm -rf /tmp/yarn-*
sudo rm -rf /tmp/usercache

# Clean system temporary directories
sudo find /tmp -name "*hadoop*" -exec rm -rf {} +
sudo find /var/tmp -name "*hadoop*" -exec rm -rf {} +
```

---

## Remove Hadoop User

If you created a dedicated user account for Hadoop, remove it.

```bash
# Check if hadoop user exists
id hadoop

# Remove hadoop user and home directory
sudo userdel -r hadoop

# Remove hadoop group
sudo groupdel hadoop

# Remove from sudoers if added
sudo visudo
# Remove any lines containing 'hadoop'
```

---

## Package Manager Cleanup

If Hadoop was installed using a package manager, use the appropriate removal commands.

### Ubuntu/Debian
```bash
# Remove Hadoop packages
sudo apt remove hadoop*
sudo apt remove --purge hadoop*

# Remove dependencies no longer needed
sudo apt autoremove
sudo apt autoclean

# Update package database
sudo apt update
```

### CentOS/RHEL/Fedora
```bash
# Remove Hadoop packages
sudo yum remove hadoop*
# or for newer versions
sudo dnf remove hadoop*

# Clean package cache
sudo yum clean all
# or
sudo dnf clean all
```

### macOS (Homebrew)
```bash
# Remove Hadoop
brew uninstall hadoop

# Remove dependencies if no longer needed
brew autoremove

# Clean cache
brew cleanup
```

---

## SSH Configuration Cleanup

If you configured SSH specifically for Hadoop (passwordless SSH), clean up the configuration.

```bash
# Remove Hadoop-specific SSH keys
rm ~/.ssh/id_rsa_hadoop*
rm ~/.ssh/id_dsa_hadoop*

# Edit SSH config file
nano ~/.ssh/config
# Remove any Hadoop-specific host configurations

# Remove authorized keys entries (if added specifically for Hadoop)
nano ~/.ssh/authorized_keys
# Remove any Hadoop-related public keys
```

---

## Verification Steps

Verify that Hadoop has been completely removed from your system.

### Check Command Availability
```bash
# These commands should return "command not found"
hadoop version
hdfs version
yarn version
mapred version
```

### Check for Running Processes
```bash
# Check for any running Hadoop processes
ps aux | grep hadoop
ps aux | grep hdfs
ps aux | grep yarn

# Check Java processes (should not show Hadoop-related processes)
jps
```

### Search for Remaining Files
```bash
# Search for any remaining Hadoop files
find / -name "*hadoop*" 2>/dev/null | head -20
find /home -name "*hadoop*" 2>/dev/null
find /opt -name "*hadoop*" 2>/dev/null
find /usr -name "*hadoop*" 2>/dev/null

# Check for Hadoop in PATH
which hadoop
which hdfs
which yarn
```

### Verify Environment Variables
```bash
# Check if Hadoop environment variables are still set
env | grep -i hadoop
echo $HADOOP_HOME
echo $PATH | grep hadoop
```

---

## Additional Cleanup

### Remove Aliases and Functions
Edit your shell configuration files to remove Hadoop-related aliases:

```bash
# Remove lines like these from ~/.bashrc or ~/.zshrc
alias start-hadoop='$HADOOP_HOME/sbin/start-all.sh'
alias stop-hadoop='$HADOOP_HOME/sbin/stop-all.sh'
alias hadoop-logs='tail -f $HADOOP_HOME/logs/*.log'
```

### Clean History Files
```bash
# Remove Hadoop commands from shell history
history | grep hadoop
# Manually edit ~/.bash_history or ~/.zsh_history if needed
```

### Remove Desktop Shortcuts
```bash
# Remove any Hadoop desktop shortcuts
rm ~/Desktop/*hadoop*
rm ~/.local/share/applications/*hadoop*
```

---

## JDK11 Considerations

### Should You Keep JDK11?

**Keep JDK11 if:**
- You have other Java applications that require it
- You're developing Java applications
- You use other big data tools that require Java
- You plan to reinstall Hadoop or similar tools later

**Remove JDK11 only if:**
- It was installed exclusively for Hadoop
- You have no other Java applications
- You want to free up disk space and are certain you won't need Java

### Verifying JDK11 Usage
```bash
# Check what processes are using Java
ps aux | grep java

# Check if other applications depend on Java
# Ubuntu/Debian
apt rdepends openjdk-11-jdk

# Check Java installation
java -version
javac -version
```

---

## Final Steps

### 1. Restart System
```bash
# Restart your system to ensure all changes take effect
sudo reboot
```

### 2. Post-Restart Verification
After restart, verify the cleanup:

```bash
# Check commands
hadoop version  # Should return "command not found"
hdfs version    # Should return "command not found"

# Check processes
ps aux | grep hadoop  # Should return nothing

# Check environment
env | grep -i hadoop  # Should return nothing
```

### 3. Disk Space Recovery
Check how much disk space was recovered:

```bash
# Check available disk space
df -h
```

---

## Troubleshooting

### Permission Denied Errors
If you encounter permission errors:
```bash
# Use sudo for system directories
sudo rm -rf /path/to/hadoop

# For files in home directory
rm -rf ~/hadoop*
```

### Files in Use Errors
If files are still in use:
```bash
# Find processes using Hadoop files
sudo lsof | grep hadoop

# Kill processes if necessary
sudo pkill -f hadoop
```

### Stubborn Files
For files that won't delete:
```bash
# Force removal
sudo rm -rf --no-preserve-root /path/to/hadoop

# Check for immutable attributes
lsattr /path/to/file
sudo chattr -i /path/to/file  # Remove immutable attribute
```

---

## Conclusion

Following this guide will completely remove Hadoop and all its related components from your laptop. The process involves stopping services, removing files, cleaning configurations, and verifying the removal.

Remember to:
- Back up any important data before starting
- Keep JDK11 if other applications need it
- Restart your system after completion
- Verify the removal was successful

If you plan to reinstall Hadoop in the future, consider documenting your current configuration before removal.

---

**Document Version:** 1.0  
**Last Updated:** June 2025  
**Compatible With:** All major Linux distributions, macOS, Windows (with appropriate command modifications)