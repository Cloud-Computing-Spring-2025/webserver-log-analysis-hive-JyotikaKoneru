# Web Server Log Analysis with Apache Hive

## **Project Overview**  
This project analyzes web server logs using Apache Hive to extract meaningful insights such as request counts, status code distributions, most visited pages, traffic sources, suspicious activities, and traffic trends over time.

## **Learning Outcomes**  

1. **Apache Hive for Big Data Analysis** – Writing and executing HiveQL queries to analyze large-scale log data.

2. **Query Optimization** – Implementing partitioning to improve query performance.

3. **Traffic Analysis** – Extracting insights from web server logs to understand user behavior and system performance.

4. **Containerized Environment** – Setting up and running Hive on Docker, and using Hue for executing queries.



## **Setup and Execution**

### **1. Start the Service**
Run the following command to start the services:
```bash
docker compose up -d
```
HiveQL queries are executed in **Hue**, accessible at **port 8888**.

### **2. Upload CSV File to Hive**
Upload the web server log CSV file from your local machine to Hive using Hue.

### **3. Execute HiveQL Queries**

Run the following HiveQL queries to perform analysis:

- **Count Total Web Requests**:
  ```sql
  SELECT COUNT(*) AS total_requests FROM hue__tmp_web_server_logs;
  ```

- **Analyze Status Codes**:
  ```sql
  SELECT status, COUNT(*) AS count FROM hue__tmp_web_server_logs GROUP BY status;
  ```

- **Identify Most Visited Pages**:
  ```sql
  SELECT url, COUNT(*) AS visits 
  FROM hue__tmp_web_server_logs 
  GROUP BY url 
  ORDER BY visits DESC 
  LIMIT 3;
  ```

- **Traffic Source Analysis**:
  ```sql
  SELECT user_agent, COUNT(*) AS count 
  FROM hue__tmp_web_server_logs 
  GROUP BY user_agent 
  ORDER BY count DESC;
  ```

- **Detect Suspicious Activity**:
  ```sql
  SELECT ip, COUNT(*) AS failed_requests 
  FROM hue__tmp_web_server_logs  
  WHERE status IN (404, 500) 
  GROUP BY ip 
  HAVING COUNT(*) > 3;
  ```

- **Analyze Traffic Trends**:
  ```sql
  SELECT SUBSTR(`timestamp`, 1, 16) AS minute, COUNT(*) AS total_requests
  FROM hue__tmp_web_server_logs
  GROUP BY SUBSTR(`timestamp`, 1, 16)
  ORDER BY minute;
  ```

### **4. Implement Partitioning**

- **Create Partitioned Table**:
  ```sql
  DROP TABLE IF EXISTS web_server_logs_partitioned;
  
  CREATE TABLE web_server_logs_partitioned (
      ip STRING,
      `timestamp` STRING,
      url STRING,
      user_agent STRING
  )
  PARTITIONED BY (status INT)
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE;
  ```

- **Load Data into Partitioned Table**:
  ```sql
  SET hive.exec.dynamic.partition = true;
  SET hive.exec.dynamic.partition.mode = nonstrict;
  ```
  ```

  INSERT OVERWRITE TABLE web_server_logs_partitioned PARTITION (status)
  SELECT ip, `timestamp`, url, user_agent, status FROM hue__tmp_web_server_logs;
  ```

### **5. Export Analysis Results**

- **Total Web Requests**:
  ```sql
  INSERT OVERWRITE DIRECTORY '/user/hive/output/total_web_requests'
  SELECT COUNT(*) FROM hue__tmp_web_server_logs;
  ```

- **Status Code Analysis**:
  ```sql
  INSERT OVERWRITE DIRECTORY '/user/hive/output/status_code_analysis'
  SELECT status, COUNT(*) AS count FROM hue__tmp_web_server_logs GROUP BY status;
  ```

- **Most Visited Pages**:
  ```sql
  INSERT OVERWRITE DIRECTORY '/user/hive/output/most_visited_pages'
  SELECT url, COUNT(*) AS visits 
  FROM hue__tmp_web_server_logs 
  GROUP BY url 
  ORDER BY visits DESC 
  LIMIT 3;
  ```

- **Traffic Source Analysis**:
  ```sql
  INSERT OVERWRITE DIRECTORY '/user/hive/output/traffic_source_analysis'
  SELECT user_agent, COUNT(*) AS count 
  FROM hue__tmp_web_server_logs 
  GROUP BY user_agent 
  ORDER BY count DESC;
  ```

- **Suspicious IP Addresses**:
  ```sql
  INSERT OVERWRITE DIRECTORY '/user/hive/output/suspicious_ip_addresses'
  SELECT ip, COUNT(*) AS failed_requests 
  FROM hue__tmp_web_server_logs  
  WHERE status IN (404, 500) 
  GROUP BY ip 
  HAVING COUNT(*) > 3;
  ```

- **Traffic Trend Over Time**:
  ```sql
  INSERT OVERWRITE DIRECTORY '/user/hive/output/traffic_trend_over_time'
  SELECT SUBSTR(`timestamp`, 1, 16) AS minute, COUNT(*) AS total_requests
  FROM hue__tmp_web_server_logs
  GROUP BY SUBSTR(`timestamp`, 1, 16)
  ORDER BY minute;
  ```

- **Verify Partitioning**:
  ```sql
  SHOW PARTITIONS web_server_logs_partitioned;
  ```

### **6. Extract Output Data to Local System**
After forming the output folder in Hive, use the following commands in VS Code to extract the data:
```bash
docker exec -it hive-server /bin/bash
hdfs dfs -get /user/hive/output /tmp/output

ls -l /tmp/output
exit

pwd

docker cp hive-server:/tmp/output /workspaces/webserver-log-analysis-hive-JyotikaKoneru/
```

## **Challenges Faced and Resolutions**
1. **Hue Server Downtime** – The Hue server repeatedly went down, causing interruptions.
   - **Resolution**: Restarting the Hue server using `docker restart hue`, monitoring logs with `docker logs hue -f`, and ensuring sufficient memory allocation for Docker.

2. **Repeated Folder Formation** – Unintended multiple folder generations while handling output directories.
   - **Resolution**: Carefully structuring Hive queries and verifying paths before running export commands to prevent duplicate folder creation.

3. **File Path Setting Issues** – Difficulty in properly setting up file paths for data extraction and storage.
   - **Resolution**: Using absolute paths, verifying HDFS directory structure with `hdfs dfs -ls /user/hive/output/`, and ensuring correct permissions before executing copy commands.
  

## **Sample Input and Output**
**Sample Input (Web Server Log Entry):**
```
192.168.1.1,2025-02-25 12:34:56,/index.html,Chrome/91,200
192.168.1.2,2025-02-25 12:35:10,/about.html,Firefox/85,404
```

**Expected Output (Status Code Analysis):**
1. Total Web Requests: <img width="468" alt="image" src="https://github.com/user-attachments/assets/34974ebd-6ead-4345-9ae9-2f79fbc0a00f" /> 
2. Analyze Status Codes: <img width="468" alt="image" src="https://github.com/user-attachments/assets/1667a978-803e-4754-ba72-0c57950d8842" /> 
4. Traffic Source Analysis: <img width="468" alt="image" src="https://github.com/user-attachments/assets/b3dec9db-0054-4102-b8e7-8f2a4ad25ed5" /> 
5. Detect Suspicious Activity: <img width="468" alt="image" src="https://github.com/user-attachments/assets/ee4c3f61-5921-4d6d-9840-e1d6aab36125" /> 
6. Analyze Traffic Trends: <img width="468" alt="image" src="https://github.com/user-attachments/assets/c9e9e7e2-6ff6-40fc-adce-06c14a5b6346" /> 
7. Implement Partitioning:
    1. <img width="468" alt="image" src="https://github.com/user-attachments/assets/d7c1465b-dad3-4cdc-8e32-cfce53b9906b" />
	2. <img width="468" alt="image" src="https://github.com/user-attachments/assets/789e5f15-0337-41a4-9180-e42675b2eed7" /> 
8. into external table:
	   <img width="468" alt="image" src="https://github.com/user-attachments/assets/a713db89-b38f-4ae6-afa9-9694b9535956" /> 


















