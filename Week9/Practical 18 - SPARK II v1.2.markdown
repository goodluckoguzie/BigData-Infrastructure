# CMP701 Practical 18: Spark SQL with Parquet Files

## Introduction

This practical builds on **Practical 17: Introduction to Apache Spark**, extending your skills to use **Spark SQL** for querying large datasets stored in **Parquet** format on HDFS. Spark SQL allows you to run SQL-like queries on structured data, simplifying complex aggregations and searches compared to RDD-based operations. You’ll work with a Wikipedia dataset to perform tasks like counting records, aggregating by user, and searching for specific text.

### Objectives
- Start the Hadoop sandbox and access the PySpark shell.
- Upload a set of Parquet files to HDFS.
- Load Parquet files into a Spark DataFrame and register it as a temporary SQL table.
- Write and execute Spark SQL queries to count records, aggregate data, and search for text.
- Handle memory issues in Spark and optimize for single-node operation.
- Understand Spark SQL’s workflow for querying structured data.

### Prerequisites
- **Hadoop Sandbox**: Hortonworks Sandbox running in Docker (`sandbox-hdp`, `sandbox-proxy`).
- **PuTTY**: For SSH access to `sandbox.hortonworks.com:2222`.
- **FileZilla**: For transferring files to/from the sandbox.
- **Dataset**: `Practical 18 - Data.zip` from Blackboard, containing 10 Parquet files.
- **Python Knowledge**: Familiarity with Python and basic SQL syntax.
- **Practical 17 Completion**: Familiarity with PySpark, HDFS, and the sandbox environment.

## Part A – Setup and Data Preparation

### Task A1: Start Hadoop Sandbox
**Objective**: Ensure the Hadoop sandbox is running in Docker.

**Activity**:
1. Open **Docker Desktop** and wait for it to show as running (green status).
2. Start the containers:
   ```bash
   docker start sandbox-hdp
   docker start sandbox-proxy
   ```
3. Verify they’re running:
   ```bash
   docker ps
   ```
   **Expected Output**:
   ```
   CONTAINER ID   IMAGE                     ...   NAMES
   <id>           hortonworks/sandbox-hdp   ...   sandbox-hdp
   <id>           hortonworks/sandbox-proxy ...   sandbox-proxy
   ```

**Why**: The `sandbox-hdp` container hosts Hadoop, HDFS, and Spark, while `sandbox-proxy` exposes ports (e.g., 2222 for SSH). Both must be active for this practical.

**Troubleshooting**:
- If containers don’t appear, ensure Docker Desktop is running and re-run the start commands.
- If errors persist, restart Docker or redeploy the sandbox using your deployment script.

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
   ```
2. If the port is missing, restart the `sandbox-proxy` container:
   ```bash
   docker restart sandbox-proxy
   ```

**Why**: Port 2222 enables SSH connections to the sandbox, necessary for accessing the PySpark shell and managing HDFS.

**Troubleshooting**:
- If the port is not mapped, verify the sandbox deployment script or check Docker Desktop’s network settings.

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
     - Enter a secure password (e.g., `hadoop1234`, 8+ characters).
     - **Expected Output**:
       ```
       passwd: password updated successfully
       ```
   - Exit:
     ```bash
     exit
     ```
   - Retry PuTTY login.

**Why**: SSH access allows you to interact with the sandbox’s Linux environment, run Spark commands, and manage HDFS.

**Troubleshooting**:
- Ensure Docker containers are running (`docker ps`).
- Verify the correct host (`sandbox.hortonworks.com`) and port (`2222`).

### Task A4: Download and Extract Dataset
**Objective**: Obtain and prepare the Wikipedia dataset for upload to HDFS.

**Activity**:
1. On your **local machine**, download `Practical 18 - Data.zip` from the Blackboard Datasets folder.
2. Extract the zip file to a local directory (e.g., `Downloads/Practical_18_Data`).
3. Verify that the extracted folder contains **10 Parquet files** (e.g., `part-00000.parquet`, `part-00001.parquet`, etc.).

**Why**: The dataset contains Wikipedia page data in Parquet format, a columnar storage format optimized for Spark SQL queries. Extracting locally prepares the files for upload to HDFS.

**Troubleshooting**:
- If the zip file is missing, check Blackboard’s Datasets folder or contact your instructor.
- Ensure your extraction tool (e.g., WinZip, 7-Zip) is working and the files are not corrupted.

### Task A5: Upload Parquet Files to HDFS
**Objective**: Store the Parquet files in HDFS for Spark access.

**Activity**:
1. In PuTTY, set HDFS permissions to avoid access issues:
   ```bash
   su hdfs
   hdfs dfs -chmod 777 /user
   exit
   ```
2. Create an HDFS directory for the dataset:
   ```bash
   hdfs dfs -mkdir -p /user/hadoop/data/wiki_parquet
   ```
3. Transfer the Parquet files to the sandbox using **FileZilla**:
   - Open **FileZilla** and configure:
     - **Protocol**: SFTP
     - **Host**: `sandbox.hortonworks.com`
     - **Port**: `2222`
     - **Username**: `root`
     - **Password**: Your password (e.g., `hadoop1234`)
   - On the **local side**, navigate to the folder containing the extracted Parquet files.
   - On the **remote side**, navigate to `/tmp`.
   - Drag all 10 Parquet files to `/tmp`.
4. Move the files from `/tmp` to HDFS:
   ```bash
   hdfs dfs -put /tmp/part-*.parquet /user/hadoop/data/wiki_parquet/
   ```
5. Verify the upload:
   ```bash
   hdfs dfs -ls /user/hadoop/data/wiki_parquet
   ```
   **Expected Output**:
   ```
   -rw-r--r--   3 root hadoop   <size> 2025-08-05 00:20 /user/hadoop/data/wiki_parquet/part-00000.parquet
   -rw-r--r--   3 root hadoop   <size> 2025-08-05 00:20 /user/hadoop/data/wiki_parquet/part-00001.parquet
   ...
   ```

**Why**: HDFS is Spark’s primary storage for large datasets. The Parquet files are uploaded to `/user/hadoop/data/wiki_parquet` for efficient querying with Spark SQL. Permissions ensure accessibility.

**Troubleshooting**:
- **FileZilla connection fails**: Verify SSH credentials, port 2222, and that `sandbox-proxy` is running.
- **Permission denied in HDFS**: Re-run the `chmod 777` command or ensure you’re using the `root` user.
- **Files not found**: Confirm the Parquet files are in `/tmp` before the `hdfs dfs -put` command.

## Part B – Spark SQL Setup and Data Loading

### Task B1: Open PySpark Shell
**Objective**: Launch the PySpark interactive shell for Spark SQL operations.

**Activity**:
1. In PuTTY, start PySpark with increased driver memory:
   ```bash
   pyspark --driver-memory 1G
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
3. Initialize the SQLContext:
   ```python
   from pyspark.sql import SQLContext
   sqlContext = SQLContext(sc)
   ```

**Why**: The PySpark shell provides an interactive Python environment with a pre-initialized `SparkContext` (`sc`). The `--driver-memory 1G` flag allocates extra memory to prevent `java.lang.OutOfMemoryError`. The `SQLContext` enables Spark SQL functionality.

**Troubleshooting**:
- **PySpark fails to start**: Ensure `sandbox-hdp` is running (`docker ps`) and the sandbox has enough memory allocated in Docker Desktop.
- **Memory errors**: Increase driver memory (e.g., `--driver-memory 2G`) if errors persist.

### Task B2: Load Parquet Files into a DataFrame
**Objective**: Load the Wikipedia Parquet dataset into a Spark DataFrame.

**Activity**:
1. In the PySpark shell, load the Parquet files:
   ```python
   wikiData = sqlContext.read.parquet("/user/hadoop/data/wiki_parquet")
   ```
2. Verify the data by counting records:
   ```python
   wikiData.count()
   ```
   **Expected Output**:
   ```
   39365
   ```

**Why**: The `read.parquet()` method loads Parquet files into a DataFrame, which includes schema information (e.g., columns like `username` and `text`). The `count()` action verifies the data was loaded correctly, showing the total number of Wikipedia pages (filtered for "berkeley" in this dataset).

**Troubleshooting**:
- **Path not found**: Confirm the HDFS path (`/user/hadoop/data/wiki_parquet`) and that all Parquet files were uploaded (Task A5).
- **Slow count**: Parquet files are large; the count may take a few seconds due to distributed processing.

### Task B3: Register DataFrame as a Temporary Table
**Objective**: Enable SQL queries by registering the DataFrame as a table.

**Activity**:
1. Register the DataFrame as a temporary table named `wikiData`:
   ```python
   wikiData.registerTempTable("wikiData")
   ```

**Why**: The `registerTempTable` method allows Spark SQL to treat the DataFrame as a relational table, enabling SQL queries on the data. The table name `wikiData` is used in subsequent queries.

**Troubleshooting**:
- **Table not found**: Ensure the `registerTempTable` command ran successfully and the DataFrame (`wikiData`) was created in Task B2.

## Part C – Running Spark SQL Queries

### Task C1: Count Total Records
**Objective**: Use Spark SQL to count the total number of Wikipedia pages.

**Activity**:
1. Run the SQL query:
   ```python
   result = sqlContext.sql("SELECT COUNT(*) AS pageCount FROM wikiData").collect()
   ```
2. Access the count from the result:
   ```python
   print(result[0].pageCount)
   ```
   **Expected Output**:
   ```
   39365
   ```

**Why**: The `SELECT COUNT(*)` query counts all rows in the `wikiData` table. The `.collect()` action returns a list of `Row` objects, and `result[0].pageCount` extracts the count value. This confirms the dataset size.

**Troubleshooting**:
- **Syntax error**: Ensure the SQL query syntax is correct and the table name is `wikiData`.
- **OutOfMemoryError**: Restart PySpark with increased memory:
   ```bash
   pyspark --driver-memory 2G
   ```

### Task C2: Find Top 10 Users by Page Creation
**Objective**: Query the top 10 usernames by the number of Wikipedia pages they created.

**Activity**:
1. Run the SQL query:
   ```python
   top_users = sqlContext.sql("""
       SELECT username, COUNT(*) AS cnt
       FROM wikiData
       WHERE username <> ''
       GROUP BY username
       ORDER BY cnt DESC
       LIMIT 10
   """).collect()
   ```
2. Display the results:
   ```python
   for row in top_users:
       print(f"Username: {row.username}, Pages Created: {row.cnt}")
   ```
   **Example Output**:
   ```
   Username: User1, Pages Created: 150
   Username: User2, Pages Created: 120
   ...
   ```

**Why**: This query:
- Filters out empty usernames (`WHERE username <> ''`).
- Groups records by `username` and counts occurrences (`COUNT(*)`).
- Orders by count in descending order (`ORDER BY cnt DESC`).
- Limits to the top 10 (`LIMIT 10`).
The `collect()` action retrieves the results as a list of `Row` objects, which we iterate to print usernames and counts.

**Troubleshooting**:
- **No results**: Ensure the `username` column exists in the Parquet schema (use `wikiData.printSchema()` to check).
- **OutOfMemoryError**: Increase driver memory (e.g., `--driver-memory 2G`) and restart PySpark.
- **Slow query**: Large datasets may take time; ensure the Parquet files are correctly loaded.

### Task C3: Count Articles Containing "california"
**Objective**: Write a Spark SQL query to count Wikipedia articles containing the word "california" in the `text` column.

**Activity**:
1. Set the Parquet reader to non-vectorized mode for single-node operation:
   ```python
   spark.conf.set("spark.sql.parquet.enableVectorizedReader", "false")
   ```
2. Run the SQL query:
   ```python
   cal_count = sqlContext.sql("""
       SELECT COUNT(*) AS californiaCount
       FROM wikiData
       WHERE text LIKE '%california%'
   """).collect()
   ```
3. Display the result:
   ```python
   print(cal_count[0].californiaCount)
   ```
   **Example Output**:
   ```
   <number>
   ```

**Why**: 
- The `spark.conf.set` command disables vectorized reading, which is optimized for distributed clusters but can cause issues in local mode (single-node sandbox).
- The `LIKE '%california%'` clause searches for the word "california" (case-insensitive) in the `text` column.
- The `COUNT(*)` aggregates the number of matching rows, and `collect()` retrieves the result.

**Troubleshooting**:
- **Zero results**: Check the `text` column exists (`wikiData.printSchema()`) and that "california" appears in the data. Try a case-sensitive search (`LIKE '%California%'`) if needed.
- **Performance issues**: The `LIKE` operation is computationally expensive; ensure sufficient driver memory (Task B1).
- **Configuration error**: Verify the `spark.conf.set` command was executed before the query.

## Part D – Saving and Retrieving Results

### Task D1: Save Query Results to HDFS
**Objective**: Save the "california" query results to HDFS for later analysis.

**Activity**:
1. Modify the query to save results:
   ```python
   cal_count.saveAsTextFile("/user/hadoop/output/california_count")
   ```
2. Verify the output:
   ```bash
   hdfs dfs -ls /user/hadoop/output/california_count
   ```
   **Expected Output**:
   ```
   -rw-r--r--   3 root hadoop   <size> 2025-08-05 00:30 /user/hadoop/output/california_count/part-00000
   ...
   ```
3. View the results:
   ```bash
   hdfs dfs -cat /user/hadoop/output/california_count/part-00000
   ```
   **Example Output**:
   ```
   [Row(californiaCount=<number>)]
   ```

**Why**: The `saveAsTextFile` method writes the query results to HDFS as text files, split into `part-` files based on partitions. This allows you to store and retrieve results for further analysis.

**Troubleshooting**:
- **Path exists error**: Delete the existing output directory:
   ```bash
   hdfs dfs -rm -r /user/hadoop/output/california_count
   ```
- **Permission denied**: Re-run the `chmod 777 /user` command from Task A5.

### Task D2: Retrieve Results to Local Machine
**Objective**: Transfer the query results to your local machine for inspection.

**Activity**:
1. Open **FileZilla**:
   - **Protocol**: SFTP
   - **Host**: `sandbox.hortonworks.com`
   - **Port**: `2222`
   - **Username**: `root`
   - **Password**: Your password (e.g., `hadoop1234`)
2. Navigate to `/user/hadoop/output/california_count` on the remote side.
3. Drag the `part-00000` file to your local `Downloads` folder.
4. Open the file in a text editor to verify the count.

**Why**: Transferring results to your local machine allows you to analyze them in tools like Excel or a text editor, especially for small outputs like counts.

**Troubleshooting**:
- **FileZilla connection fails**: Check SSH credentials, port 2222, and that `sandbox-proxy` is running.
- **Empty output**: Verify the `saveAsTextFile` command ran successfully and the HDFS path exists.

## Part E – Understanding Spark SQL

### Task E1: Explore Spark SQL Documentation
**Objective**: Learn about Spark SQL and its capabilities.

**Activity**:
1. Open a browser and visit:
   - Spark SQL Guide: `https://spark.apache.org/docs/latest/sql-programming-guide.html`
2. Review key concepts:
   - **DataFrames**: Structured datasets with schema, similar to relational tables.
   - **SQLContext**: The entry point for Spark SQL, used to load data and run queries.
   - **Temporary Tables**: Created with `registerTempTable` for SQL querying.
   - **Parquet Format**: A columnar storage format optimized for big data queries.

**Why**: Understanding Spark SQL’s programming model is essential for efficient querying of large datasets. It simplifies data processing compared to RDDs by leveraging SQL syntax.

## Conclusion

This practical introduced **Spark SQL** for querying structured data in Apache Spark:
- Started the Hadoop sandbox and verified SSH access.
- Uploaded 10 Parquet files to HDFS (`/user/hadoop/data/wiki_parquet`).
- Loaded Parquet data into a Spark DataFrame and registered it as a temporary table.
- Ran SQL queries to count records, find top users, and search for "california" in article text.
- Saved and retrieved query results from HDFS.
- Learned about Spark SQL’s workflow and Parquet format.

**Key Takeaways**:
- Spark SQL enables SQL-like queries on big data, simplifying aggregations and searches.
- Parquet files provide efficient, schema-aware storage for structured data.
- Driver memory and configuration settings (e.g., vectorized reader) are critical for local mode performance.

For further learning, explore Spark SQL’s DataFrame API or try more complex queries on larger datasets.
