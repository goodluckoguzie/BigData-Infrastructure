# CMP701 Practical 15: Hadoop Distributed File System (HDFS I)

## Introduction

This practical introduces the **Hadoop Distributed File System (HDFS)**, a distributed file system designed for storing large datasets across multiple servers in a Hadoop cluster. Unlike a local Linux file system, HDFS provides fault tolerance and parallel data processing, making it essential for Hadoop-based data analysis. You will learn to manage files on HDFS using command-line tools, download datasets from the internet, transfer them to HDFS, retrieve files, merge datasets, and organize directories within HDFS.

### Objectives
- Connect to a Linux-based Hadoop host using PuTTY.
- Download datasets to the local Linux file system and upload them to HDFS.
- Use HDFS commands to manage files and directories (`-put`, `-get`, `-du`, `-getmerge`, `-cp`).
- Transfer files from the Linux VM to your local computer using FileZilla.
- Understand the workflow of moving data between local storage, HDFS, and your personal machine.

## Task 0: Connect to Your Hadoop Host

**Objective**: Establish a connection to the Linux-based Hadoop host.

**Activity**:
- **Launch PuTTY**: Open PuTTY, a tool for remotely accessing Linux systems.
- **Enter Connection Details**: In the "Host Name (or IP address)" field, input `sandbox.hortonworks.com` and set the port to `2222`. Click "Open." On first connection, accept the security alert to add the server's key fingerprint to PuTTY's record.
- **Login**:
  - **Username**: `root`
  - **Password**: Use `hadoop` for the first login. You will be prompted to confirm this password and set a new password for the `root` user. Use this new password for subsequent logins.
- **Expected Outcome**: A terminal prompt appears (e.g., `[root@localhost ~]#`), indicating a successful connection.

**Why**: This establishes access to the Linux virtual machine (VM) hosting the Hadoop environment, allowing you to interact with both the local file system and HDFS.

## Task 1: Create a Local Staging Directory

**Objective**: Set up a local directory to store downloaded datasets before uploading to HDFS.

**Activity**:
- Create the directory:
  ```bash
  mkdir /localDatasets
  ```
- Navigate to it:
  ```bash
  cd /localDatasets
  ```

**Why**: The `/localDatasets` directory acts as a staging area on the Linux file system, similar to a "Downloads" folder, where datasets are temporarily stored before being transferred to HDFS.

## Task 2: Download Datasets from the Web

**Objective**: Download two San Francisco salary datasets to the local file system.

**Activity**:
- Download the datasets using `wget`:
  ```bash
  wget https://raw.githubusercontent.com/funkimunk/hadoop-datasets/main/sf-salaries-2011-2013.csv
  wget https://raw.githubusercontent.com/funkimunk/hadoop-datasets/main/sf-salaries-2014.csv
  ```
- Verify the files are downloaded:
  ```bash
  ls
  ```
  - Expected Output:
    ```
    sf-salaries-2011-2013.csv  sf-salaries-2014.csv
    ```
- **Note**: You can paste URLs into PuTTY by right-clicking in the terminal.

**Why**: These datasets (CSV files containing salary data) will be used for Hadoop processing. Downloading them to the local file system is the first step before uploading to HDFS, where Hadoop can access them.

## Task 3: Prepare HDFS and Upload Files

**Objective**: Switch to the HDFS user to configure permissions, create directories in HDFS, and upload files from the local file system.

**Activity**:
1. **Switch to the HDFS User**:
   - Command:
     ```bash
     su hdfs
     cd
     ```
   - **Why**: The `hdfs` user has administrative privileges for managing HDFS. Switching to this user ensures proper permissions for configuration tasks.
2. **Set Permissions on /user Directory**:
   - Command:
     ```bash
     hdfs dfs -chmod 777 /user
     ```
   - **Why**: This grants read/write/execute permissions to all users on the `/user` directory in HDFS, avoiding permission issues during file operations (not recommended for production environments due to security risks).
3. **Return to Root User**:
   - Command:
     ```bash
     exit
     ```
   - **Why**: Most tasks are performed as the `root` user for simplicity in this lab.
4. **Create Directories in HDFS**:
   - Commands:
     ```bash
     hdfs dfs -mkdir /user/hadoop
     hdfs dfs -mkdir /user/hadoop/sf-salaries-2011-2013
     hdfs dfs -mkdir /user/hadoop/sf-salaries-2014
     ```
   - Verify:
     ```bash
     hdfs dfs -ls /user/hadoop
     ```
     - Expected Output:
       ```
       drwxrwxrwx   - root hadoop          0 2025-07-29 13:42 /user/hadoop/sf-salaries-2011-2013
       drwxrwxrwx   - root hadoop          0 2025-07-29 13:42 /user/hadoop/sf-salaries-2014
       ```
   - **Note**: Using `ls /user/hadoop` (without `hdfs dfs`) will result in an error because it targets the local file system, not HDFS.
   - **Why**: HDFS requires a structured directory hierarchy to organize data. These directories will store the salary datasets.
5. **Upload Files to HDFS**:
   - Navigate to the local directory:
     ```bash
     cd /localDatasets
     ```
   - Upload files:
     ```bash
     hdfs dfs -put sf-salaries-2011-2013.csv /user/hadoop/sf-salaries-2011-2013/
     hdfs dfs -put sf-salaries-2014.csv /user/hadoop/sf-salaries-2014/
     ```
   - Verify:
     ```bash
     hdfs dfs -ls /user/hadoop/sf-salaries-2011-2013
     hdfs dfs -ls /user/hadoop/sf-salaries-2014
     ```
     - Expected Output:
       ```
       -rw-r--r--   3 root hadoop   <size> 2025-07-29 13:42 /user/hadoop/sf-salaries-2011-2013/sf-salaries-2011-2013.csv
       -rw-r--r--   3 root hadoop   <size> 2025-07-29 13:42 /user/hadoop/sf-salaries-2014/sf-salaries-2014.csv
       ```
   - **Why**: The `hdfs dfs -put` command transfers files from the local file system to HDFS, making them accessible for Hadoop processing.

## Task 4: Check File Sizes in HDFS

**Objective**: Determine the storage usage of files and directories in HDFS.

**Activity**:
- Check the size of the `/user/hadoop` directory:
  ```bash
  hdfs dfs -du /user/hadoop
  ```
- Check the size of the 2014 dataset:
  ```bash
  hdfs dfs -du /user/hadoop/sf-salaries-2014/sf-salaries-2014.csv
  ```
- **Output**: Displays sizes in bytes (e.g., `123456789 /user/hadoop/sf-salaries-2014/sf-salaries-2014.csv`).

**Why**: The `-du` command helps monitor storage usage, which is critical for managing resources in a distributed system like HDFS.

## Task 5: Retrieve Files from HDFS to Local File System

**Objective**: Copy a file from HDFS back to the local Linux file system.

**Activity**:
- Create a local directory for retrieved files:
  ```bash
  mkdir /returnedDatasets
  ```
- Retrieve the 2011-2013 dataset:
  ```bash
  hdfs dfs -get /user/hadoop/sf-salaries-2011-2013/sf-salaries-2011-2013.csv /returnedDatasets/
  ```
- Verify:
  ```bash
  ls /returnedDatasets
  ```
  - Expected Output:
    ```
    sf-salaries-2011-2013.csv
    ```

**Why**: After processing data in Hadoop, you may need to retrieve results to the local file system for further analysis or transfer to your personal computer.

## Task 6: Transfer Files to Your Local Computer

**Objective**: Use FileZilla to transfer a file from the Linux VM to your personal computer.

**Activity**:
- Open **FileZilla** on your local machine (available in the lab’s Start Menu).
- Configure the connection:
  - Go to **File → Site Manager → New Site**.
  - Set:
    - **Host**: `sandbox.hortonworks.com`
    - **Protocol**: SFTP
    - **Port**: `2222`
    - **Logon Type**: Normal
    - **Username**: `root`
    - **Password**: Your `root` password (set in Practical 13 or 14).
  - Click **Connect**.
- Navigate to `/returnedDatasets` in the remote file tree (right side).
- Select a local folder on your computer (left side).
- Drag `sf-salaries-2011-2013.csv` from the remote to the local folder to download.
- Verify the file is accessible on your computer (e.g., open in Excel).

**Why**: Transferring files to your local machine allows you to analyze data using tools like Excel or share results outside the Hadoop environment.

## Task 7: Merge Files from HDFS

**Objective**: Combine multiple HDFS files into a single local file.

**Activity**:
- Merge the two datasets:
  ```bash
  hdfs dfs -getmerge /user/hadoop/sf-salaries-2011-2013/* /user/hadoop/sf-salaries-2014/* /returnedDatasets/testMerge.csv
  ```
- Verify the merged file’s row count:
  ```bash
  wc -l /returnedDatasets/testMerge.csv
  ```
  - Expected Output: Approximately 158,000 rows (combined data from both datasets).
- Transfer `testMerge.csv` to your local computer using FileZilla (follow Task 6 steps).
- Verify the row count in Excel on your local machine.

**Why**: Hadoop jobs often produce multiple output files. The `-getmerge` command consolidates them into a single file for easier analysis.

## Task 8: Copy Directories in HDFS

**Objective**: Copy directories within HDFS to organize data.

**Activity**:
- Copy the salary directories to a new HDFS directory:
  ```bash
  hdfs dfs -cp /user/hadoop/sf-salaries-2011-2013 /user/hadoop/sf-salaries-2014 /user/hadoop/salaries
  ```
- Verify:
  ```bash
  hdfs dfs -ls /user/hadoop/salaries
  ```
  - Expected Output:
    ```
    drwxr-xr-x   - root hadoop          0 2025-07-29 13:42 /user/hadoop/salaries/sf-salaries-2011-2013
    drwxr-xr-x   - root hadoop          0 2025-07-29 13:42 /user/hadoop/salaries/sf-salaries-2014
    ```

**Why**: The `-cp` command allows you to reorganize data within HDFS, which is useful for structuring datasets for Hadoop jobs.

## Conclusion

This practical introduced you to managing files on HDFS, a critical component of Hadoop:
- Connected to the Hadoop host and set up a local staging directory.
- Downloaded datasets and uploaded them to HDFS using `hdfs dfs -put`.
- Managed HDFS files with commands like `-ls`, `-du`, `-get`, `-getmerge`, and `-cp`.
- Transferred files to your local computer using FileZilla.
- Learned the workflow: **Download → Stage Locally → Upload to HDFS → Process → Retrieve**.

These skills are foundational for Hadoop data processing, as all analysis jobs require data to be stored in HDFS. For further practice, explore additional HDFS commands (e.g., `hdfs dfs -cat` to view file contents) or refer to the Hadoop documentation (`hdfs dfs -help`).