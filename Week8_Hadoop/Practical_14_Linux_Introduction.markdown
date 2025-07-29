# CMP701 Practical 14: A Basic Introduction to Linux

## Introduction

This practical introduces the Linux operating system, which is essential for working with Hadoop. It covers connecting to a Linux host using PuTTY, basic Linux shell commands for file management, and installing/using the Nano text editor. These skills are foundational for managing data in future Hadoop-based labs.

### Objectives
- Connect to a Linux virtual machine (VM) or container using PuTTY.
- Learn basic Linux commands to navigate and manage files and directories.
- Install and use the Nano text editor to create and edit files.
- Understand case sensitivity in Linux file systems.

## Task 0: Connect to Your Hadoop Host

**Objective**: Establish a connection to a Linux-based Hadoop host.

**Activity**:
- **Launch PuTTY**: Open PuTTY, a tool for remotely accessing Linux systems.
- **Enter Connection Details**: In the "Host Name (or IP address)" field, input `sandbox.hortonworks.com` and set the port to `2222`. Click "Open." On first connection, accept the security alert to add the server's key fingerprint to PuTTY's record.
- **Login**:
  - **Username**: `root`
  - **Password**: Use `hadoop` for the first login. You will be prompted to confirm this password and set a new password for the `root` user. Note this new password, as it will be required for future logins.
- **Expected Outcome**: A terminal prompt appears (e.g., `[root@localhost ~]#`), indicating a successful connection.

## Topic 1: Exploring Basic Linux Commands

**Objective**: Learn essential Linux commands to navigate and manage files.

**Activity**:
1. **Check Current Directory**:
   - Command: `pwd` (Print Working Directory)
   - Purpose: Displays the current directory path.
   - Example Output: `/root` (the home directory for the root user).
   - Note: The root user is the supreme administrator in Linux, with home directory at `/root`. Other users’ home directories are typically at `/home/username` (e.g., `/home/john` in CentOS).

2. **Navigate to Root Directory**:
   - Command: `cd /`
   - Purpose: Changes to the root of the file system, where all files and directories originate.
   - Explanation: `/` is the top-level directory in Linux.

3. **List Files and Directories**:
   - Command: `ls`
   - Purpose: Lists files and directories in the current location.
   - Example Output:
     ```
     bin  boot  dev  etc  home  root  usr  var
     ```
   - Note: Use `ls -a` to show hidden files (names starting with a dot, e.g., `.bashrc`).
   - **Color Coding in Linux**:
     - **Green**: Executable files
     - **Blue**: Directories
     - **Cyan**: Symbolic links
     - **Yellow**: Pipes
     - **Magenta**: Sockets or image files (e.g., `.jpg`, `.png`)
     - **Red**: Archives (e.g., `.tar`, `.zip`)
     - **Bold Yellow on Black**: Block or character device drivers
     - **Blinking White on Red**: Orphaned symlinks or filesystems
     - **Blue on Green**: Writable directories without sticky bit
     - **Cyan**: Files writable by others with sticky bit

4. **Understand Case Sensitivity**:
   - Command: `cd /Home`
   - Result: Error (`No such file or directory`) because Linux is case-sensitive.
   - Correct Command: `cd /home`
   - Explanation: `Home` and `home` are treated as distinct names.

5. **Create a File**:
   - Command: `touch /root/perf`
   - Purpose: Creates an empty file named `perf` in `/root`.

6. **Copy a File**:
   - Command: `cp /root/perf /home/perf1`
   - Purpose: Copies `perf` from `/root` to `/home`, renaming it to `perf1`.
   - Syntax: `cp /path/to/source /path/to/destination`

7. **Move or Rename a File**:
   - Command: `mv /root/perf /home/perf`
   - Purpose: Moves `perf` from `/root` to `/home` or renames it if the destination is in the same directory.
   - Syntax: `mv /path/to/source /path/to/destination`

8. **Create a Directory**:
   - Command: `mkdir /home/myNewDirectory`
   - Purpose: Creates a directory named `myNewDirectory` in `/home`.
   - Verification: Use `ls /home` to confirm (output: `myNewDirectory perf perf1`).

9. **Create a File in a Directory**:
   - Command: `touch /home/myNewDirectory/obstacle`
   - Purpose: Creates an empty file named `obstacle` in `/home/myNewDirectory`.

10. **Delete Files and Directories**:
    - Delete a file: `rm /home/myNewDirectory/obstacle`
    - Delete an empty directory: `rmdir /home/myNewDirectory`
    - Delete a non-empty directory: `rm -r -f /home/myNewDirectory`
    - **Warning**: Avoid `rm -rf /` as it deletes the entire system.

11. **Use Shortcuts**:
    - **Up Arrow (↑)**: Recalls previous commands.
    - **Tab Key**: Auto-completes file or directory names to prevent typos.

## Task 2: Install and Use Nano Text Editor

**Objective**: Install the Nano text editor and create/edit a file.

**Activity**:
1. **Fix Repository Issues**:
   - **Background**: The Hadoop VM uses outdated software repositories, causing errors during software installation.
   - Commands (run sequentially):
     ```bash
     yum erase ius-release
     yum-config-manager --save --setopt=HDP-2.6-repo-1.skip_if_unavailable=true
     yum-config-manager --save --setopt=HDP-UTILS-1.1.0.22-repo-1.skip_if_unavailable=true
     yum-config-manager --save --setopt=ambari-2.6.2.0.skip_if_unavailable=true
     ```
   - Purpose: Removes references to unavailable repositories to prevent errors.

2. **Install Nano**:
   - Commands:
     ```bash
     yum clean all
     yum install nano
     ```
   - Purpose: Clears cached packages and installs Nano.
   - Action: When prompted `Is this ok [y/d/N]:`, type `y` and press Enter.

3. **Create and Edit a File**:
   - Command: `nano /root/helloWorld`
   - Action:
     - Nano editor opens.
     - Type: `Hello, this is my first Linux file!`
     - Save: Press `Ctrl + O`, then Enter.
     - Exit: Press `Ctrl + X`.
   - Verify: `cat /root/helloWorld` (displays the file’s content).

## Conclusion

This practical provided foundational Linux skills for Hadoop environments:
- Connected to a Linux host using PuTTY.
- Learned commands (`pwd`, `cd`, `ls`, `cp`, `mv`, `touch`, `mkdir`, `rm`, `rmdir`) to navigate and manage files.
- Understood Linux’s case sensitivity and color-coded file system.
- Installed Nano and created/edited a text file.

These skills prepare you for advanced Hadoop tasks, such as managing files in HDFS. For further practice, try creating additional files or directories and experimenting with commands like `ls -l` (detailed listing) or `man <command>` (view command documentation).