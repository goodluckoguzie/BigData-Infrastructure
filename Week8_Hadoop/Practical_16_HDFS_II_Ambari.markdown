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
- **Ambari Access**: Browser access to `sandbox.hortonworks.com:8080` with login credentials (default: `ambari`/`ambari`, or as set in prior labs).
- **FileZilla**: For transferring files between your local computer and the Linux VM, if needed.

## Task 0: Start Hadoop and Access Ambari

**Objective**: Connect to the Ambari web interface to manage HDFS.

**Activity**:
1. **Start the Hadoop VM**: Ensure your Hortonworks Sandbox or Hadoop cluster is running. If services are stopped, start them using the Ambari dashboard or command-line tools (e.g., `start_ambari.sh` if provided in your setup).
2. **Open a Browser**: Use Chrome, Firefox, or another modern browser.
3. **Access Ambari**: Navigate to:
   ```
   http://sandbox.hortonworks.com:8080
   ```
   - **Port 8080** is the default for Ambari’s web interface.
4. **Log In**:
   - **Username**: `ambari`
   - **Password**: Use the password set in previous labs (default: `ambari` if unchanged).
5. **Verify**: The Ambari dashboard loads, showing cluster status and services (e.g., HDFS, YARN).

**Why**: Ambari is a management tool for Hadoop clusters, providing a graphical interface to monitor services and manage HDFS. Accessing it ensures you can perform file operations visually.

## Task 1: Prepare CSV Files (If Needed)

**Objective**: Ensure the required CSV files are available on your local computer for uploading via Ambari.

**Activity**:
- **If you completed Practical 15**: The files `sf-salaries-2011-2013.csv` and `sf-salaries-2014.csv` are already on your Linux VM in `/localDatasets`. Skip to **Task 2**.
- **If not**:
  1. Connect to the Linux VM using PuTTY:
     - Host: `sandbox.hortonworks.com`
     - Port: `2222`
     - Username: `root`
     - Password: As set in prior labs.
  2. Create a directory and download the files:
     ```bash
     mkdir /localDatasets
     cd /localDatasets
     wget https://raw.githubusercontent.com/funkimunk/hadoop-datasets/main/sf-salaries-2011-2013.csv
     wget https://raw.githubusercontent.com/funkimunk/hadoop-datasets/main/sf-salaries-2014.csv
     ```
  3. Transfer files to your local computer using FileZilla:
     - Open FileZilla and connect using:
       - Host: `sandbox.hortonworks.com`
       - Protocol: SFTP
       - Port: `2222`
       - Username: `root`
       - Password: As set.
     - Navigate to `/localDatasets` (remote side).
     - Select a local folder (e.g., `Downloads`).
     - Drag `sf-salaries-2011-2013.csv` and `sf-salaries-2014.csv` to your local folder.

**Why**: Ambari’s Files View uploads files directly from your local computer to HDFS, so you need the CSV files on your machine (e.g., Windows) rather than the Linux VM.

## Task 2: Open Ambari Files View

**Objective**: Access the HDFS file manager in Ambari to perform file operations.

**Activity**:
1. In the Ambari dashboard, locate the **menu selector** (top-right corner, typically a grid or dropdown icon).
2. Select **Files View**.
3. Verify the interface loads, showing HDFS directories like `/user`, `/tmp`, etc.

**Why**: Files View is Ambari’s graphical interface for HDFS, similar to a file explorer. It allows you to browse, upload, download, and manage files without using `hdfs dfs` commands.

## Task 3: Create Required Folders in HDFS

**Objective**: Set up the same HDFS directory structure as in Practical 15 for organizing datasets.

**Activity**:
1. In Files View, navigate to the `/user` directory by clicking it.
2. Click **New Folder**:
   - Name: `hadoop`
   - Click **Create**.
3. Enter the `/user/hadoop` directory.
4. Create three subfolders:
   - Click **New Folder** → Name: `sf-salaries-2011-2013` → Create.
   - Click **New Folder** → Name: `sf-salaries-2014` → Create.
   - Click **New Folder** → Name: `sf-salaries` → Create.
5. Verify the structure by navigating back to `/user/hadoop`. You should see:
   ```
   sf-salaries-2011-2013
   sf-salaries-2014
   sf-salaries
   ```

**Why**: Organizing data in HDFS directories is essential for Hadoop workflows. This structure mirrors Practical 15, ensuring consistency for data storage and future processing.

## Task 4: Upload Files to HDFS via Ambari

**Objective**: Upload the two CSV files from your local computer to their respective HDFS directories.

**Activity**:
1. Navigate to `/user/hadoop/sf-salaries-2011-2013` in Files View.
2. Click the **Upload File** button (cloud with an upward arrow).
3. In the file selection window, browse to your local `Downloads` folder (or wherever you stored the files).
4. Select `sf-salaries-2011-2013.csv` and click **Open**.
5. Repeat for the second file:
   - Navigate to `/user/hadoop/sf-salaries-2014`.
   - Click **Upload File**, select `sf-salaries-2014.csv`, and click **Open**.
6. Verify:
   - In `/user/hadoop/sf-salaries-2011-2013`, you should see `sf-salaries-2011-2013.csv`.
   - In `/user/hadoop/sf-salaries-2014`, you should see `sf-salaries-2014.csv`.

**Why**: Uploading files to HDFS via Ambari is equivalent to the `hdfs dfs -put` command in Practical 15. It places data in HDFS, making it accessible for Hadoop jobs, but with a point-and-click interface.

## Task 5: View File Details

**Objective**: Inspect file details (e.g., size, owner, permissions) in HDFS using Files View.

**Activity**:
1. Navigate to `/user/hadoop/sf-salaries-2011-2013` in Files View.
2. Observe the file details for `sf-salaries-2011-2013.csv`. The columns display:
   - **Name**: `sf-salaries-2011-2013.csv`
   - **Size**: Approximately 11.2 MB (as noted in the document).
   - **Last Modified**: Date and time the file was uploaded (e.g., `2025-07-29 13:42`).
   - **Owner**: Likely `ambari` or `hadoop` (depends on the user uploading).
   - **Group**: Likely `hadoop`.
   - **Permissions**: Likely `rw-r--r--` (read/write for owner, read-only for others).
3. Repeat for `sf-salaries-2014.csv` in `/user/hadoop/sf-salaries-2014`.

**Why**: File details help verify successful uploads and troubleshoot issues (e.g., incorrect permissions preventing access). Unlike the `hdfs dfs -du` command, Files View shows file sizes directly but not directory sizes.

## Task 6: Download Files from HDFS to Your Computer

**Objective**: Retrieve a file from HDFS to your local computer using Ambari.

**Activity**:
1. Navigate to `/user/hadoop/sf-salaries-2011-2013` in Files View.
2. Click the row for `sf-salaries-2011-2013.csv` (it highlights blue).
3. Click the **Download** button (appears in the file operations toolbar).
4. The file downloads to your local computer’s **Downloads** folder.
5. Verify the file is accessible (e.g., open in Excel or a text editor).

**Why**: Downloading files from HDFS allows you to analyze or share data outside the Hadoop environment, similar to the `hdfs dfs -get` command in Practical 15.

## Task 7: Concatenate Files in HDFS

**Objective**: Merge the two CSV files into a single file using Ambari’s concatenate feature.

**Activity**:
1. **Copy Files to the Same Folder**:
   - Navigate to `/user/hadoop/sf-salaries-2011-2013`.
   - Click the row for `sf-salaries-2011-2013.csv` (it highlights blue).
   - Click **Copy** in the toolbar.
   - In the copy window, navigate to `/user/hadoop/sf-salaries-2014` and click **Copy**.
   - Verify both files are now in `/user/hadoop/sf-salaries-2014`:
     ```
     sf-salaries-2011-2013.csv
     sf-salaries-2014.csv
     ```
2. **Concatenate Files**:
   - In `/user/hadoop/sf-salaries-2014`, hold **Shift** and click both files to select them.
   - Click the **Concatenate** button in the toolbar.
   - A merged file (e.g., `concatenated.txt`) downloads to your local computer’s **Downloads** folder.
3. **Verify**:
   - Rename the downloaded file to `merged.csv` (if needed).
   - Open in Excel or use a command-line tool (e.g., `wc -l merged.csv` on a Linux system) to confirm it contains approximately 158,000 rows (combined data from both files).

**Why**: Concatenation combines multiple files into one, useful for consolidating datasets for analysis. This is equivalent to the `hdfs dfs -getmerge` command in Practical 15 but done visually. The files must be in the same HDFS directory for Ambari to merge them.

## Task 8: Copy Directories in HDFS

**Objective**: Copy the entire `hadoop` directory to another location in HDFS for organization or backup.

**Activity**:
1. Navigate to `/user` in Files View.
2. Click the row for the `hadoop` folder (it highlights blue).
3. Click the **Copy** button in the toolbar.
4. In the copy window, navigate to `/tmp` and click **Copy**.
5. Verify:
   - Navigate to `/tmp/hadoop`.
   - Confirm the subfolders (`sf-salaries-2011-2013`, `sf-salaries-2014`, `sf-salaries`) and their files are copied.

**Why**: Copying directories in HDFS organizes data or creates backups, similar to the `hdfs dfs -cp` command in Practical 15. Ambari’s recursive copy ensures all contents are transferred.

## Conclusion

This practical introduced managing HDFS using **Ambari’s Files View**, a graphical alternative to the command-line tools used in Practical 15:
- Accessed Ambari and navigated to Files View.
- Created directories and uploaded files to HDFS visually.
- Viewed file details (size, owner, permissions).
- Downloaded files to your local computer.
- Concatenated multiple files into a single file.
- Copied entire directories within HDFS.

**Key Takeaways**:
- Ambari simplifies HDFS operations for users unfamiliar with command-line interfaces, making it ideal for enterprise settings.
- The tasks mirror Practical 15 but use a point-and-click interface, demonstrating flexibility in Hadoop management.
- Understanding both command-line and web-based methods prepares you for diverse Hadoop environments.

For further exploration, try other Ambari features (e.g., monitoring HDFS health) or compare with command-line equivalents using `hdfs dfs -help`.