# CMP701 Practical 16: Hadoop Distributed File System (HDFS II) via Ambari Web Interface

## Introduction

This practical builds on **Practical 15 – HDFS I**, where you managed the Hadoop Distributed File System (HDFS) using command-line tools. Here, you’ll perform similar tasks—creating directories, uploading files, viewing file details, downloading files, concatenating files, and copying directories—using the **Ambari web interface**, specifically its **Files View**. Ambari provides a user-friendly, graphical alternative to command-line operations, making HDFS management accessible to users less comfortable with terminal commands. This is common in enterprise environments where web-based tools simplify cluster management.

### Objectives
- Access the Hadoop cluster via Ambari’s web interface.
- Use Ambari’s Files View to create directories and upload files to HDFS.
- View file details (e.g., size, owner, permissions) in HDFS.
- Download files from HDFS to your local computer.
- Concatenate multiple files into a single file using Ambari.
- Copy directories within HDFS for organization or backup.
- Understand how Ambari’s interface simplifies HDFS operations compared to command-line methods.

## Prerequisites
- **Hadoop Cluster**: Hortonworks Sandbox or equivalent, running on a Linux VM (e.g., `sandbox.hortonworks.com`).
- **CSV Files**: The `sf-salaries-2011-2013.csv` and `sf-salaries-2014.csv` files from Practical 15, available on your local computer or Linux VM.
- **Ambari Access**: Browser access to `sandbox.hortonworks.com:8080` with login credentials (default: `maria_dev`/`maria_dev`, or as set in prior labs).
- **FileZilla**: For transferring files between your local computer and the Linux VM, if needed.

## Part A – Setup and Access

### Task A1: Start Hadoop Sandbox in Docker
**Objective**: Ensure the Hadoop sandbox is running in Docker.

**Activity**:
1. Open **Docker Desktop** and wait until it’s fully running.
2. Start both containers:
   ```bash
   docker start sandbox-hdp
   docker start sandbox-proxy
   ```
3. Verify they’re running:
   ```bash
   docker ps
   ```
   **Expected Output**: You should see `sandbox-hdp` and `sandbox-proxy` listed.

**Why**: Docker hosts the Hortonworks Sandbox, and both containers must be running to access Hadoop services and Ambari.

### Task A2: Verify Ports
**Objective**: Confirm the necessary ports are mapped for SSH and Ambari access.

**Activity**:
1. Check port mappings:
   ```bash
   docker port sandbox-proxy
   ```
   **Expected Output**:
   ```
   2222/tcp -> 0.0.0.0:2222
   8080/tcp -> 0.0.0.0:8080
   ```
2. If ports are missing, re-run your deploy script or restart Docker.

**Why**: Port 2222 enables SSH access (PuTTY), and port 8080 allows access to Ambari’s web interface. Verifying ensures connectivity.

### Task A3: Reset Password if Needed
**Objective**: Reset the root password if you can’t log in via PuTTY.

**Activity**:
1. Enter the `sandbox-hdp` container:
   ```bash
   docker exec -it sandbox-hdp bash
   ```
2. Reset the root password:
   ```bash
   passwd root
   ```
   - Enter a secure password (e.g., `hadoop1234`, 8+ characters).
3. Exit the container:
   ```bash
   exit
   ```
4. Try SSH again via PuTTY:
   - **Host**: `sandbox.hortonworks.com`
   - **Port**: `2222`
   - **Username**: `root`
   - **Password**: Your new password (e.g., `hadoop1234`).

**Why**: Password issues can block SSH access. Resetting ensures you can log in to prepare datasets.

### Task A4: Access Ambari
**Objective**: Log into Ambari’s web interface to manage HDFS.

**Activity**:
1. Open a browser (Chrome or Firefox recommended).
2. Navigate to:
   ```
   http://sandbox.hortonworks.com:8080
   ```
3. Log in:
   - **Username**: `maria_dev`
   - **Password**: `maria_dev` (or your updated Ambari admin password).
4. If services show as “Stopped” in the Ambari dashboard:
   - Go to **Services** → **Actions** → **Start All** to ensure all Hadoop services are running.

**Why**: Ambari’s dashboard provides access to HDFS management tools. Starting services prevents errors during file operations.

## Part B – Preparing Datasets

### Task B1: SSH into Sandbox (PuTTY)
**Objective**: Access the Linux VM to download datasets if not already present.

**Activity**:
1. Open PuTTY and connect:
   - **Host**: `sandbox.hortonworks.com`
   - **Port**: `2222`
   - **Username**: `root`
   - **Password**: Your password (e.g., `hadoop1234`).
2. Create a staging directory (if not already present):
   ```bash
   mkdir /localDatasets
   cd /localDatasets
   ```
3. Download datasets (skip if already present from Practical 15):
   ```bash
   wget https://raw.githubusercontent.com/funkimunk/Hadoop-DataSets/main/sf-salaries-2011-2013.csv
   wget https://raw.githubusercontent.com/funkimunk/Hadoop-DataSets/main/sf-salaries-2014.csv
   ```
4. Verify:
   ```bash
   ls
   ```
   **Expected Output**:
   ```
   sf-salaries-2011-2013.csv  sf-salaries-2014.csv
   ```

**Why**: The datasets must be available on the VM to transfer to your local machine for Ambari uploads.

### Task B2: Transfer CSV Files to Local Machine
**Objective**: Move the CSV files to your local computer for uploading via Ambari.

**Activity**:
1. Open **FileZilla** (installed on your lab machine).
2. Click **Site Manager** → **New Site**:
   - **Protocol**: SFTP
   - **Host**: `sandbox.hortonworks.com`
   - **Port**: `2222`
   - **Username**: `root`
   - **Password**: Your password.
3. Connect and navigate to `/localDatasets` on the remote side.
4. Select a local folder (e.g., `Desktop` or `Downloads`).
5. Drag `sf-salaries-2011-2013.csv` and `sf-salaries-2014.csv` to your local folder.
6. Verify the files are on your local machine (e.g., check `Desktop`).

**Why**: Ambari’s Files View uploads files from your local computer, not the VM, so you need the CSVs locally.

## Part C – Using Ambari Files View

### Task C1: Open Files View
**Objective**: Access Ambari’s HDFS file manager.

**Activity**:
1. In the Ambari dashboard, click the **selector icon** (top-right, three horizontal bars).
2. Choose **Files View**.
3. Verify the interface loads, showing HDFS directories like `/user`, `/tmp`, etc.

**Why**: Files View is Ambari’s graphical interface for HDFS, allowing you to manage files without `hdfs dfs` commands.

### Task C2: Create Folders
**Objective**: Set up the HDFS directory structure from Practical 15.

**Activity**:
1. Navigate to `/user` in Files View.
2. Click **New Folder**:
   - Name: `hadoop`
   - Click **Create**.
3. Enter `/user/hadoop`.
4. Create three subfolders:
   - Click **New Folder** → Name: `sf-salaries-2011-2013` → Create.
   - Click **New Folder** → Name: `sf-salaries-2014` → Create.
   - Click **New Folder** → Name: `sf-salaries` → Create.
5. Navigate back to `/user/hadoop` and verify:
   ```
   sf-salaries-2011-2013
   sf-salaries-2014
   sf-salaries
   ```

**Why**: This structure organizes datasets consistently with Practical 15, preparing HDFS for file uploads and processing.

### Task C3: Upload Files to HDFS
**Objective**: Upload the CSV files to their respective HDFS directories.

**Activity**:
1. Navigate to `/user/hadoop/sf-salaries-2011-2013`.
2. Click **Upload** (cloud with upward arrow).
3. Browse to your local folder (e.g., `Downloads`), select `sf-salaries-2011-2013.csv`, and click **Open**.
4. Repeat for the second file:
   - Navigate to `/user/hadoop/sf-salaries-2014`.
   - Click **Upload**, select `sf-salaries-2014.csv`, and click **Open**.
5. Verify:
   - In `/user/hadoop/sf-salaries-2011-2013`, see `sf-salaries-2011-2013.csv`.
   - In `/user/hadoop/sf-salaries-2014`, see `sf-salaries-2014.csv`.

**Why**: Uploading via Ambari is equivalent to `hdfs dfs -put`, making data available for Hadoop jobs.

### Task C4: View File Properties
**Objective**: Inspect file and directory details in HDFS.

**Activity**:
1. Navigate to `/user` in Files View.
2. Observe the directory contents:
   - **Name**: Folders (e.g., `hadoop`).
   - **Size**: Not shown for directories.
   - **Last Modified**: Date/time of creation or modification.
   - **Owner**: Likely `hdfs` or `maria_dev`.
   - **Group**: Likely `hadoop`.
   - **Permissions**: E.g., `rwxr-xr-x`.
3. Navigate to `/user/hadoop/sf-salaries-2011-2013`.
4. Check details for `sf-salaries-2011-2013.csv`:
   - **Name**: `sf-salaries-2011-2013.csv`
   - **Size**: Approximately 11.7 MB.
   - **Last Modified**: E.g., `2025-08-04 14:30`.
   - **Owner**: Likely `hdfs` or `maria_dev`.
   - **Group**: Likely `hadoop`.
   - **Permissions**: Likely `rw-r--r--`.
5. Repeat for `sf-salaries-2014.csv` in `/user/hadoop/sf-salaries-2014`.

**Why**: Viewing properties verifies uploads and ensures correct permissions. Unlike `hdfs dfs -du`, Files View shows file sizes but not directory sizes.

### Task C5: Download Files from HDFS
**Objective**: Retrieve a file from HDFS to your local computer.

**Activity**:
1. Navigate to `/user/hadoop/sf-salaries-2011-2013`.
2. Click the row for `sf-salaries-2011-2013.csv` (it highlights blue).
3. Click **Download** (downward arrow).
4. Verify the file is in your local `Downloads` folder (open in Excel or a text editor).

**Why**: Downloading allows analysis outside Hadoop, equivalent to `hdfs dfs -get`.

### Task C6: Merge Files (Concatenate)
**Objective**: Combine the two CSV files into one.

**Activity**:
1. **Copy File**:
   - Navigate to `/user/hadoop/sf-salaries-2011-2013`.
   - Select `sf-salaries-2011-2013.csv` (row highlights blue).
   - Click **Copy**, navigate to `/user/hadoop/sf-salaries-2014`, and click **Copy**.
   - Verify both files are in `/user/hadoop/sf-salaries-2014`:
     ```
     sf-salaries-2011-2013.csv
     sf-salaries-2014.csv
     ```
2. **Concatenate**:
   - In `/user/hadoop/sf-salaries-2014`, hold **Shift** and select both files.
   - Click **Concatenate**.
   - The merged file downloads as `concatenated.txt` to your `Downloads` folder.
3. **Verify**:
   - Rename the file to `merged.csv`.
   - Open in Excel or run `wc -l merged.csv` (on a Linux system) to confirm ~158,000 rows.

**Why**: Concatenation merges datasets for analysis, equivalent to `hdfs dfs -getmerge`. Files must be in the same directory for Ambari’s merge feature.

### Task C7: Copy Folder Recursively
**Objective**: Copy the `hadoop` directory to another HDFS location.

**Activity**:
1. Navigate to `/user`.
2. Select the `hadoop` folder (row highlights blue).
3. Click **Copy**.
4. Navigate to `/tmp` and click **Copy**.
5. Verify:
   - Navigate to `/tmp/hadoop`.
   - Confirm subfolders (`sf-salaries-2011-2013`, `sf-salaries-2014`, `sf-salaries`) and files are copied.

**Why**: Recursive copying organizes or backs up data, equivalent to `hdfs dfs -cp`.

## Conclusion
This practical demonstrated HDFS management using Ambari’s Files View, a graphical alternative to Practical 15’s command-line tasks:
- Started the Hadoop sandbox and accessed Ambari.
- Prepared and transferred datasets.
- Created directories, uploaded files, viewed properties, downloaded files, merged files, and copied directories in HDFS.

**Key Takeaways**:
- Ambari simplifies HDFS operations for non-technical users.
- Tasks mirror Practical 15 but use a point-and-click interface.
- Combining command-line and web-based skills prepares you for diverse Hadoop environments.

For further exploration, try Ambari’s service monitoring features or compare with `hdfs dfs -help` commands.
