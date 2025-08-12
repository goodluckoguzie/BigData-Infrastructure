# CMP701 Practical 17: Introduction to Apache Spark (PySpark on Hadoop Sandbox)

## Introduction

This practical introduces **Apache Spark**, a powerful big data processing engine that’s faster and more user-friendly than traditional MapReduce. Using **PySpark** (Spark’s Python interface) on your Hadoop sandbox, you’ll learn to process data in parallel, leveraging Spark’s **Resilient Distributed Datasets (RDDs)**. This lab builds on Practical 15 (HDFS I) and Practical 16 (HDFS II), using HDFS to store input and output data.

### Objectives
- Start the Hadoop sandbox and access the PySpark shell.
- Create and manipulate RDDs using a simple dataset (Fibonacci numbers).
- Download a text file (“Ulysses”), upload it to HDFS, and process it with Spark.
- Write and run a PySpark word count program to analyze text data.
- View and retrieve results from HDFS.
- Understand Spark’s transformations and actions for data processing.

## Prerequisites
- **Hadoop Sandbox**: Hortonworks Sandbox running in Docker (`sandbox-hdp`, `sandbox-proxy`).
- **PuTTY**: For SSH access to `sandbox.hortonworks.com:2222`.
- **FileZilla**: For transferring output files to your local machine.
- **Text File**: Access to `http://www.gutenberg.org/files/4300/4300-0.txt` (James Joyce’s “Ulysses”).
- **Python Knowledge**: Basic understanding of Python (lists, functions).

## Part A – Setup and Access

### Task A1: Start Your Hadoop Sandbox
**Objective**: Ensure the Hadoop sandbox is running in Docker.

**Activity**:
1. Open **Docker Desktop** and wait for it to show as running (green status).
2. Check if the `sandbox-hdp` container is running:
   ```bash
   docker ps
   ```
3. If `sandbox-hdp` is not running, start it:
   ```bash
   docker start sandbox-hdp
   ```
4. Connect `sandbox-hdp` to the default bridge network (if not already connected):
   ```bash
   docker network connect bridge sandbox-hdp
   ```
5. Check if the `sandbox-proxy` container exists:
   ```bash
   docker ps -a
   ```
   - If a container named `cool_aryabhata` or another auto-generated name appears instead of `sandbox-proxy`, rename it:
     ```bash
     docker rename cool_aryabhata sandbox-proxy
     ```
   - If a `sandbox-proxy` container exists but is stopped or misconfigured, remove it:
     ```bash
     docker rm sandbox-proxy
     ```
   - If no `sandbox-proxy` container exists or after removal, create it with the correct port mappings:
     ```bash
     docker run -d --name sandbox-proxy --network bridge -p 2222:22 -p 4200:4200 -p 8020:8020 -p 8042:8042 -p 8080:8080 -p 8188:8188 -p 8983:8983 -p 10000:10000 -p 10001:10001 -p 10002:10002 -p 11000:11000 -p 15000:15000 -p 16000:16000 -p 18080:18080 -p 19888:19888 -p 21000:21000 -p 33553:33553 -p 39419:39419 -p 42111:42111 -p 50070:50070 -p 50075:50075 -p 50090:50090 -p 50111:50111 -p 60000:60000 -p 60080:60080 -p 61080:61080 -p 61888:61888 -p 62000:62000 hortonworks/sandbox-proxy:1.0
     ```
6. Verify both containers are running:
   ```bash
   docker ps
   ```
   **Expected Output**:
   ```
   CONTAINER ID   IMAGE                           ...   NAMES
   <id>           hortonworks/sandbox-hdp:2.6.5   ...   sandbox-hdp
   <id>           hortonworks/sandbox-proxy:1.0   ...   sandbox-proxy
   ```

**Why**: The `sandbox-hdp` container hosts Hadoop and Spark, while `sandbox-proxy` exposes ports (e.g., 2222 for SSH) for external access. Both must be active, correctly named, and networked for the lab to proceed.

**Debugging**:
- **Error**: `No such container: sandbox-proxy`
  - **Cause**: The `sandbox-proxy` container does not exist or was created with an auto-generated name (e.g., `cool_aryabhata`).
  - **Fix**: Check for existing containers with `docker ps -a`. If a proxy container exists with a different name, rename it using `docker rename <old-name> sandbox-proxy`. If no proxy container exists, run the `docker run` command above to create it.
- **Error**: `Conflict. The container name "/sandbox-proxy" is already in use`
  - **Cause**: A previous attempt created a `sandbox-proxy` container that is stopped or misconfigured.
  - **Fix**: Remove the existing container:
    ```bash
    docker rm sandbox-proxy
    ```
    Then re-run the `docker run` command above.
- **Error**: `container <id> not attached to default bridge network`
  - **Cause**: The `sandbox-hdp` container is not on the default bridge network, causing networking issues.
  - **Fix**: Connect `sandbox-hdp` to the bridge network:
    ```bash
    docker network connect bridge sandbox-hdp
    ```
    Then re-run the `docker run` command for `sandbox-proxy` with `--network bridge`.

### Task A2: Verify SSH Port
**Objective**: Confirm SSH access is available on port 2222.

**Activity**:
1. Check port mappings:
   ```bash
   docker port sandbox-proxy
   ```
   **Expected Output**:
   ```
   2222/tcp -> 0.0.0.0:2222
   ...
   ```
2. If the port is missing, ensure the `sandbox-proxy` container was created with the correct port mappings (see Task A1). Remove and recreate the container if needed:
   ```bash
   docker rm sandbox-proxy
   docker run -d --name sandbox-proxy --network bridge -p 2222:22 -p 4200:4200 -p 8020:8020 -p 8042:8042 -p 8080:8080 -p 8188:8188 -p 8983:8983 -p 10000:10000 -p 10001:10001 -p 10002:10002 -p 11000:11000 -p 15000:15000 -p 16000:16000 -p 18080:18080 -p 19888:19888 -p 21000:21000 -p 33553:33553 -p 39419:39419 -p 42111:42111 -p 50070:50070 -p 50075:50075 -p 50090:50090 -p 50111:50111 -p 60000:60000 -p 60080:60080 -p 61080:61080 -p 61888:61888 -p 62000:62000 hortonworks/sandbox-proxy:1.0
   ```

**Why**: Port 2222 enables SSH connections to the sandbox, necessary for accessing the PySpark shell.

**Debugging**:
- **Error**: No output from `docker port sandbox-proxy` or missing 2222 mapping
  - **Cause**: The `sandbox-proxy` container was created without proper port mappings or is not running.
  - **Fix**: Verify the container exists and is running with `docker ps`. If it exists but has no port mappings, remove and recreate it using the `docker run` command from Task A1.

### Task A3: Connect via PuTTY
**Objective**: Log into the sandbox using SSH.

**Activity**:
1. Open **PuTTY** and configure:
   - **Host**: `sandbox.hortonworks.com`
   - **Port**: `2222`
   - **Connection type**: SSH
2. Click **Open**, accept any security alert (key fingerprint), and log in:
   - **Username**: `root`
   - **Password**: Your password (e.g., `hadoop1234`)
3. **If “Access denied”**:
   - Access the container directly:
     ```bash
     docker exec -it sandbox-hdp bash
     ```
   - Reset the password:
     ```bash
     passwd root
     ```
     - Enter a secure password (8+ characters, e.g., `hadoop1234`).
     - **Expected Output**:
       ```
       passwd: password updated successfully
       ```
   - Exit:
     ```bash
     exit
     ```
   - Retry PuTTY login.

**Why**: SSH access allows you to interact with the sandbox’s Linux environment and run Spark commands.

### Task A4: Open PySpark Shell
**Objective**: Launch the PySpark interactive shell for Spark operations.

**Activity**:
1. In PuTTY, start PySpark:
   ```bash
   pyspark
   ```
   **Expected Output**:
   ```
   Welcome to
         ____              __
        / __/__  ___ _____/ /__
       _\ \/ _ \/ _ `/ __/  '_/
      /___/ .__/\_,_/_/ /_/\_\   version x.x.x
         /_/
   ```
2. Reduce log verbosity:
   ```python
   sc.setLogLevel("WARN")
   ```

**Why**: The PySpark shell provides an interactive Python environment with a pre-initialized `SparkContext` (`sc`), ready for RDD operations.

## Part B – Working with Spark RDDs

### Task B1: Create a Parallelized Collection (Fibonacci Numbers)
**Objective**: Create an RDD from a small dataset to understand Spark’s parallelism.

**Activity**:
1. In the PySpark shell, create a Fibonacci list:
   ```python
   fib = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
   rdd = sc.parallelize(fib)
   ```
2. Retrieve all elements:
   ```python
   rdd.collect()
   ```
   **Expected Output**:
   ```
   [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
   ```

**Why**: `sc.parallelize()` distributes a Python list across the cluster as an RDD, Spark’s core data structure. `collect()` retrieves the data for verification.

### Task B2: View RDD Partitions
**Objective**: Understand how Spark splits data across partitions.

**Activity**:
1. Check partition layout:
   ```python
   rdd.glom().collect()
   ```
   **Example Output**:
   ```
   [[0, 1, 1, 2, 3], [5, 8, 13, 21, 34]]
   ```

**Why**: `glom()` shows how Spark divides the RDD into partitions, which are processed in parallel across the cluster.

### Task B3: Re-parallelize with Custom Partitions
**Objective**: Control the number of partitions for better parallelism.

**Activity**:
1. Create an RDD with 2 partitions:
   ```python
   rdd2 = sc.parallelize(fib, 2)
   rdd2.glom().collect()
   ```
   **Example Output**:
   ```
   [[0, 1, 1, 2, 3], [5, 8, 13, 21, 34]]
   ```

**Why**: Specifying partitions (e.g., 2) allows you to optimize data distribution for performance, especially on larger clusters.

## Part C – Using Real Data from HDFS

### Task C1: Download the Book “Ulysses”
**Objective**: Download a text file to the sandbox’s local file system.

**Activity**:
1. In PuTTY, create a directory:
   ```bash
   mkdir /localDatasets
   cd /localDatasets
   ```
2. Download “Ulysses”:
   ```bash
   wget http://www.gutenberg.org/files/4300/4300-0.txt -O ulysses.txt
   ```
3. Verify:
   ```bash
   ls
   ```
   **Expected Output**:
   ```
   ulysses.txt
   ```
   **Troubleshooting**: If `wget` fails, check your internet connection or URL (`http://www.gutenberg.org/files/4300/4300-0.txt`).

**Why**: The text file will be used as input for Spark processing, staged locally before HDFS upload.

### Task C2: Upload to HDFS
**Objective**: Store the text file in HDFS for Spark access.

**Activity**:
1. Set permissions to avoid access issues:
   ```bash
   su hdfs
   hdfs dfs -chmod 777 /user
   exit
   ```
2. Create an HDFS directory:
   ```bash
   hdfs dfs -mkdir -p /user/hadoop/ulysses
   ```
3. Upload the file:
   ```bash
   hdfs dfs -put ulysses.txt /user/hadoop/ulysses/
   ```
4. Verify:
   ```bash
   hdfs dfs -ls /user/hadoop/ulysses
   ```
   **Expected Output**:
   ```
   -rw-r--r--   3 root hadoop   <size> 2025-08-05 00:10 /user/hadoop/ulysses/ulysses.txt
   ```

**Why**: HDFS is Spark’s primary storage for large datasets. Permissions ensure accessibility, and `hdfs dfs -put` transfers the file.

### Task C3: Create an RDD from HDFS File
**Objective**: Load the text file into a Spark RDD and explore it.

**Activity**:
1. In the PySpark shell, create an RDD:
   ```python
   uly_rdd = sc.textFile("/user/hadoop/ulysses/ulysses.txt")
   ```
2. Count lines:
   ```python
   uly_rdd.count()
   ```
   **Example Output**: ~26,000 (exact number depends on the file).
3. Check partitions:
   ```python
   uly_rdd.glom().collect()
   ```
   **Example Output**: Lists of text lines grouped by partition.

**Why**: `textFile()` reads an HDFS file into an RDD, with each line as an element. `count()` and `glom()` verify data loading and distribution.

## Part D – Simple Word Count Program

### Task D1: Study Given Code
**Objective**: Understand a sample Spark word count program.

**Activity**:
1. Review this example code (provided in the lab):
   ```python
   from pyspark import SparkContext

   def main():
       sc = SparkContext(appName='newApp')
       input_file = sc.textFile('/user/hduser/input/input.txt')
       x = input_file.flatMap(lambda line: line.split()) \
           .map(lambda word: (word, 1)) \
           .reduceByKey(lambda a, b: a + b)
       x.saveAsTextFile('/user/hduser/output')
       sc.stop()

   if __name__ == '__main__':
       main()
   ```
2. **Key Components**:
   - **Transformations**:
     - `flatMap(lambda line: line.split())`: Splits lines into words.
     - `map(lambda word: (word, 1))`: Creates key-value pairs (word, 1).
     - `reduceByKey(lambda a, b: a + b)`: Sums counts for each word.
   - **Action**:
     - `saveAsTextFile()`: Writes results to HDFS.
   - **Note**: In the PySpark shell, `sc` is pre-initialized. This script creates a new `SparkContext` for standalone execution via `spark-submit`.

**Why**: Understanding transformations (lazy) and actions (executed) is key to Spark programming. This code counts word frequencies in a text file.

### Task D2: Create Word Count Program in Terminal
**Objective**: Create a word count program for “Ulysses” directly on the sandbox.

**Activity**:
1. In PuTTY, create `wordcount.py` using `nano`:
   ```bash
   cd /root
   nano wordcount.py
   ```
2. Type or paste the following code:
   ```python
   from pyspark import SparkContext

   def main():
       sc = SparkContext(appName='wordcount')
       input_file = sc.textFile('/user/hadoop/ulysses/ulysses.txt')
       counts = input_file.flatMap(lambda line: line.split()) \
           .map(lambda word: (word, 1)) \
           .reduceByKey(lambda a, b: a + b)
       counts.saveAsTextFile('/user/hadoop/output/wordcount')
       sc.stop()

   if __name__ == '__main__':
       main()
   ```
3. Save and exit:
   - Press `Ctrl+O`, then `Enter` to save.
   - Press `Ctrl+X` to exit.
4. Verify the file exists:
   ```bash
   ls /root
   ```
   **Expected Output**:
   ```
   wordcount.py
   ```
   **Troubleshooting**:
   - If `nano` is unfamiliar, use arrow keys to navigate, type the code, and follow save/exit steps.
   - If `nano` is not installed, use `vi` (type `i` to insert, paste code, press `Esc`, then `:wq` to save and quit).

**Why**: Creating the script directly on the sandbox avoids FileZilla for this step, aligning with terminal-based workflows. The script processes `ulysses.txt` and outputs word counts to HDFS.

### Task D3: Run Script with Spark
**Objective**: Execute the word count program.

**Activity**:
1. In PuTTY, run:
   ```bash
   spark-submit --master local wordcount.py
   ```
2. **Troubleshooting**:
   - **Error**: “Path exists” → Delete the output directory:
     ```bash
     hdfs dfs -rm -r /user/hadoop/output/wordcount
     ```
   - **Error**: “Permission denied” → Re-run permissions (Task C2) or check HDFS path (`/user/hadoop/ulysses/ulysses.txt`).
   - **Error**: “Syntax error” → Re-open `wordcount.py` in `nano` and check indentation or typos.

**Why**: `spark-submit` runs the script in Spark’s local mode, processing `ulysses.txt` and saving results to HDFS.

### Task D4: View and Retrieve Output
**Objective**: Check the word count results and transfer to your local machine.

**Activity**:
1. List output files:
   ```bash
   hdfs dfs -ls /user/hadoop/output/wordcount
   ```
   **Expected Output**:
   ```
   -rw-r--r--   3 root hadoop   <size> 2025-08-05 00:15 /user/hadoop/output/wordcount/part-00000
   ...
   ```
2. View the first 20 lines:
   ```bash
   hdfs dfs -cat /user/hadoop/output/wordcount/part-00000 | head -20
   ```
   **Example Output**:
   ```
   ('the', 15000)
   ('and', 8000)
   ('a', 7500)
   ...
   ```
3. Transfer output to your local machine:
   - Open **FileZilla**:
     - **Protocol**: SFTP
     - **Host**: `sandbox.hortonworks.com`
     - **Port**: `2222`
     - **Username**: `root`
     - **Password**: Your password.
   - Navigate to `/user/hadoop/output/wordcount` on the remote side.
   - Drag `part-00000` to your local `Downloads` folder.
   - Open in a text editor to verify word counts.
4. **Troubleshooting**:
   - If no output, check Spark job logs in PuTTY or verify `ulysses.txt` exists:
     ```bash
     hdfs dfs -ls /user/hadoop/ulysses
     ```
   - If FileZilla fails, check SSH credentials or port 2222.

**Why**: Results are stored in HDFS as `part-` files (one per partition). Transferring to your local machine allows further analysis (e.g., in Excel or a text editor).

### Task D5: Explore Spark Documentation
**Objective**: Learn about Spark’s transformations and actions.

**Activity**:
1. Open a browser and visit:
   - Transformations: `https://spark.apache.org/docs/latest/programming-guide.html#transformations`
   - Actions: `https://spark.apache.org/docs/latest/programming-guide.html#actions`
2. Review key concepts:
   - **Transformations**: Create new RDDs (e.g., `map`, `flatMap`, `reduceByKey`). Lazy, not executed until an action is called.
   - **Actions**: Trigger computation and return results (e.g., `collect`, `saveAsTextFile`).

**Why**: Understanding Spark’s programming model is essential for writing efficient data processing jobs.

## Conclusion
This practical introduced Apache Spark and PySpark:
- Started the Hadoop sandbox and accessed the PySpark shell.
- Created RDDs from a small list and explored partitioning.
- Processed a real text file (“Ulysses”) from HDFS.
- Wrote a word count program in the terminal, ran it, and retrieved results.
- Learned about Spark’s transformations and actions.

**Key Takeaways**:
- Spark’s RDDs enable parallel data processing.
- Transformations (`flatMap`, `map`, `reduceByKey`) are lazy; actions (`collect`, `saveAsTextFile`) trigger execution.
- Spark integrates with HDFS for scalable storage.

For further learning, explore Spark’s DataFrame API or try processing larger datasets.
