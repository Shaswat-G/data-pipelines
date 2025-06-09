# ETL using shell scripting

This is just to show that ETL pipelines can be built using simple shell scripts and python scripts. They can be scheduled using cron jobs to run at regular intervals.

## Introduction to Linux Commands for ETL

Linux provides powerful command-line tools that are perfect for building ETL pipelines. These tools follow the Unix philosophy of doing one thing well and can be combined using pipes to create complex data processing workflows. For data engineers, mastering these commands is essential for efficient data manipulation, especially when working with text-based data formats like CSV, TSV, or log files.

## Task:

Report hourly average, min and max temp from a remote temp sensor to a dashboard and update every minute.

Use the get_temp_api to get temperature and store it in a log file - with a 60 reading buffer. (1 hour)
Write a script call get_stats.py to read the log file and calculate the average, min and max temp for the last hour.
Load using load_stats.py to load the stats into a database.

Use crontab to schedule the script to run every minute.

## Linux Commands for Data Extraction

### Exercise 1 - Extracting data using 'cut' command

The `cut` command is a powerful tool for data extraction in the ETL process. It allows you to extract specific sections of text from each line of files or from piped data. This is particularly useful for working with delimited data like CSV files or structured log files.

#### Extracting characters

To extract specific characters from each line of text:

```bash
# Extract the first four characters
echo "database" | cut -c1-4
# Output: data

# Extract 5th to 8th characters
echo "database" | cut -c5-8
# Output: base

# Extract non-contiguous characters (1st and 5th)
echo "database" | cut -c1,5
# Output: db
```

#### Extracting fields/columns

For delimited text files, `cut` can extract specific fields:

```bash
# Extract usernames (first field) from /etc/passwd
cut -d":" -f1 /etc/passwd

# Extract multiple fields (username, userid, home directory)
cut -d":" -f1,3,6 /etc/passwd

# Extract a range of fields (userid through home directory)
cut -d":" -f3-6 /etc/passwd
```

In ETL pipelines, `cut` is invaluable for:

- Extracting specific columns from CSV/TSV data sources
- Parsing log files with consistent formatting
- Preprocessing data before transformation steps
- Creating simplified views of complex data for reporting

### Exercise 2 - Transforming data using 'tr' command

The `tr` command (translate) is a powerful utility for data transformation in ETL processes. It allows you to replace or remove specific characters from the input text, which is essential for data cleansing and standardization.

#### Translating character sets

Convert text between different character cases:

```bash
# Convert lowercase to uppercase
echo "Shell Scripting" | tr "[a-z]" "[A-Z]"
# Output: SHELL SCRIPTING

# Using predefined character sets
echo "Shell Scripting" | tr "[:lower:]" "[:upper:]"
# Output: SHELL SCRIPTING

# Convert uppercase to lowercase
echo "Shell Scripting" | tr "[A-Z]" "[a-z]"
# Output: shell scripting
```

#### Squeezing repeated characters

The `-s` option replaces repeated occurrences of characters with a single instance:

```bash
# Replace multiple spaces with a single space
ps | tr -s " "

# Using character class notation
ps | tr -s "[:space:]"
```

This is particularly useful for:

- Normalizing whitespace in text data
- Cleaning up poorly formatted input files
- Preparing data for fixed-width field processing

#### Deleting characters

The `-d` option removes specified characters completely:

```bash
# Remove all digits from text
echo "My login pin is 5634" | tr -d "[:digit:]"
# Output: My login pin is
```

In ETL workflows, the `tr` command is essential for:

- Standardizing case for consistent analysis
- Removing unwanted characters (control chars, non-printable chars)
- Normalizing data from different sources
- Preparing text for further processing by other tools
- Converting between different data formats or encodings

The simplicity and efficiency of `tr` make it ideal for processing large volumes of text data in ETL pipelines, especially when combined with other text processing tools using pipes.

## Linux Commands for Data Transformation

### Exercise 3 - Filtering data using 'grep' command

The `grep` command is essential for filtering data in ETL pipelines. It searches for lines that match a specified pattern and outputs only those lines, making it perfect for extracting relevant data from large files.

#### Basic pattern matching

```bash
# Find all lines containing the word "error" in a log file
grep "error" application.log

# Case-insensitive search
grep -i "error" application.log

# Display line numbers along with the matched lines
grep -n "error" application.log
```

#### Using regular expressions

`grep` becomes more powerful with regular expressions:

```bash
# Find lines that start with "2023-"
grep "^2023-" application.log

# Find lines containing email addresses
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" users.txt

# Find lines containing IP addresses
grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" server_access.log
```

#### Inverting matches and context

```bash
# Show lines that do NOT contain "success"
grep -v "success" application.log

# Show 2 lines before and after each match
grep -B 2 -A 2 "critical" application.log
```

In ETL pipelines, `grep` is valuable for:

- Filtering out irrelevant data before processing
- Extracting specific transaction types from log files
- Identifying records that match business criteria
- Finding errors or anomalies in data sets
- Pre-filtering large datasets to reduce processing load

### Exercise 4 - Advanced text processing with 'sed'

The `sed` (Stream Editor) command is powerful for performing text transformations on data streams. It's particularly useful in ETL pipelines for modifying data during the transformation phase.

#### Basic substitution

```bash
# Replace the first occurrence of "old" with "new" on each line
sed 's/old/new/' input.txt

# Replace all occurrences of "old" with "new" on each line
sed 's/old/new/g' input.txt

# Case-insensitive substitution
sed 's/old/new/gi' input.txt
```

#### Deleting and printing specific lines

```bash
# Delete all lines containing "DEBUG"
sed '/DEBUG/d' application.log

# Print only lines matching pattern
sed -n '/ERROR/p' application.log

# Delete the first 10 lines of a file
sed '1,10d' data.csv
```

#### Multiple operations

```bash
# Multiple substitutions using -e
sed -e 's/old/new/g' -e 's/foo/bar/g' input.txt

# Using a sed script file for complex transformations
sed -f transform.sed input.txt
```

In ETL pipelines, `sed` is invaluable for:

- Reformatting data to match target schema requirements
- Cleaning and standardizing values
- Removing or commenting out header lines
- Adding/removing delimiters in data files
- Performing complex string replacements

### Exercise 5 - Data processing with 'awk'

`awk` is a powerful text processing language designed for data extraction and reporting. It's particularly well-suited for ETL pipelines due to its ability to process structured data and perform calculations.

#### Field processing

```bash
# Print specific columns (fields) from a CSV file
awk -F, '{print $1, $3}' data.csv

# Sum values in the third column
awk -F, '{sum+=$3} END {print "Total: " sum}' sales.csv

# Calculate average of the fourth column
awk -F, '{sum+=$4; count++} END {print "Average: " sum/count}' data.csv
```

#### Filtering with conditions

```bash
# Print lines where the second field is greater than 100
awk -F, '$2 > 100 {print $0}' data.csv

# Print customer records where purchase amount exceeds $1000
awk -F, '$3 > 1000 {print $1 " spent $" $3}' transactions.csv

# Multiple conditions with logical operators
awk -F, '$2 == "COMPLETED" && $4 > 500 {print $0}' orders.csv
```

#### Built-in functions and variables

```bash
# Convert values to uppercase
awk -F, '{print toupper($1), $2}' data.csv

# Count occurrences of each value in the second column
awk -F, '{count[$2]++} END {for (val in count) print val, count[val]}' data.csv

# Format output with printf
awk -F, '{printf "%-20s $%.2f\n", $1, $3}' sales.csv
```

In ETL pipelines, `awk` excels at:

- Performing complex calculations on structured data
- Transforming data formats with precise control
- Creating summary statistics and aggregations
- Filtering data based on sophisticated conditions
- Restructuring data with field reordering and manipulation

## Combining Commands for ETL Pipelines

The real power of Linux commands in ETL comes from combining them using pipes (`|`). This allows you to create efficient data processing pipelines where the output of one command becomes the input to the next.

### Example: Complete ETL Pipeline with Linux Commands

Here's an example of processing a server log file to extract, transform, and load error data:

```bash
# Extract errors, transform the format, and load to a summary file
grep "ERROR" server.log |
  cut -d' ' -f1,3,7- |
  sed 's/ERROR://' |
  tr '[:lower:]' '[:upper:]' |
  awk '{count[$1]++} END {for(date in count) print date, count[date] " errors"}' |
  sort > error_summary.txt
```

This pipeline:

1. Extracts only ERROR lines from the log file
2. Cuts out specific fields (date, error code, and message)
3. Removes the "ERROR:" prefix
4. Converts the text to uppercase
5. Counts errors by date
6. Sorts the results
7. Loads the data into a summary file

### Real-World ETL Task Example

Let's implement our temperature monitoring task using Linux commands:

```bash
#!/bin/bash
# Script: temp_etl.sh

# Extract: Get temperature data from API and append to log
curl -s http://sensor-api.example.com/temp |
  jq '.temperature' >> /var/log/temp_readings.log

# Transform: Calculate statistics for the last 60 readings
tail -n 60 /var/log/temp_readings.log |
  awk 'BEGIN {min=999; max=-999}
       {sum+=$1; if($1<min) min=$1; if($1>max) max=$1}
       END {printf "%.2f,%.2f,%.2f\n", sum/NR, min, max}' > /tmp/temp_stats.csv

# Load: Insert into database (using a simple method)
cat /tmp/temp_stats.csv |
  psql -c "COPY temp_hourly_stats(avg_temp, min_temp, max_temp) FROM STDIN WITH CSV" mydatabase
```

This script:

1. Extracts temperature data from an API and logs it
2. Transforms the last 60 readings to calculate stats
3. Loads the results into a PostgreSQL database

To schedule it as specified:

```bash
# Add to crontab to run every minute
(crontab -l 2>/dev/null; echo "* * * * * /path/to/temp_etl.sh") | crontab -
```

These Linux command-line tools demonstrate how powerful and flexible they can be for building efficient ETL pipelines, especially for text-based data processing.

## Scheduling ETL Pipelines with Cron

Scheduling is a critical aspect of ETL pipelines to ensure data is processed at the right times and intervals. In Linux environments, cron is the standard utility for scheduling tasks.

### Understanding Cron Syntax

The cron schedule format consists of five time fields plus the command to execute:

```
* * * * * command_to_execute
│ │ │ │ │
│ │ │ │ └─ Day of week (0-6, Sunday=0)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

Each field can contain:

- A specific value (e.g., `5`)
- A range (e.g., `1-5`)
- A list of values (e.g., `1,3,5`)
- A step value (e.g., `*/10` means every 10 units)
- An asterisk (`*`) representing all valid values

### Common ETL Scheduling Patterns

Different ETL workflows require different scheduling patterns:

```bash
# Run every minute (real-time data collection)
* * * * * /path/to/realtime_etl.sh

# Run hourly at the beginning of each hour
0 * * * * /path/to/hourly_etl.sh

# Run daily at midnight
0 0 * * * /path/to/daily_etl.sh

# Run weekly on Sunday at 2 AM
0 2 * * 0 /path/to/weekly_etl.sh

# Run monthly on the 1st at 3 AM
0 3 1 * * /path/to/monthly_etl.sh

# Run every 15 minutes
*/15 * * * * /path/to/quarter_hourly_etl.sh

# Run weekdays at 6 AM and 6 PM
0 6,18 * * 1-5 /path/to/business_hours_etl.sh
```

### Managing ETL Job Output

For ETL jobs, capturing output is crucial for monitoring and debugging:

```bash
# Redirect stdout and stderr to a log file
0 * * * * /path/to/etl_script.sh >> /var/log/etl_script.log 2>&1

# Redirect stdout to one file and stderr to another
0 * * * * /path/to/etl_script.sh > /var/log/etl_output.log 2> /var/log/etl_errors.log

# Email the output (if mail server is configured)
0 0 * * * /path/to/etl_script.sh | mail -s "ETL Job Report" admin@example.com
```

### Environment Variables in Cron Jobs

Cron jobs run with a minimal environment, which can cause issues for ETL scripts:

```bash
# Set environment variables directly in crontab
0 * * * * export DATABASE_URL="postgres://user:pass@host/db"; /path/to/etl_script.sh

# Better approach: set variables in the script itself
0 * * * * /path/to/etl_script.sh

# In etl_script.sh:
#!/bin/bash
export PATH=/usr/local/bin:/usr/bin:/bin
export DATABASE_URL="postgres://user:pass@host/db"
# Rest of your ETL code...
```

### ETL Job Dependencies and Sequencing

For complex ETL pipelines with dependent jobs:

```bash
# Simple sequential jobs
0 1 * * * /path/to/etl_step1.sh && /path/to/etl_step2.sh

# Using a lock file to prevent overlapping runs
0 * * * * flock -n /tmp/etl.lock /path/to/long_running_etl.sh

# Using timestamps to track completion
0 * * * * /path/to/check_dependency.sh && /path/to/etl_job.sh

# In check_dependency.sh:
#!/bin/bash
if [[ -f /var/run/etl_upstream.timestamp ]]; then
  LAST_RUN=$(cat /var/run/etl_upstream.timestamp)
  NOW=$(date +%s)
  # Only proceed if upstream job ran in the last hour
  if (( NOW - LAST_RUN < 3600 )); then
    exit 0  # Success, dependency is satisfied
  fi
fi
exit 1  # Dependency not satisfied
```

### Monitoring Cron Jobs

Effective monitoring is essential for ETL reliability:

```bash
# Create a timestamp file on successful completion
0 * * * * /path/to/etl_job.sh && date +%s > /var/run/etl_job.timestamp

# Send alert on failure
0 * * * * /path/to/etl_job.sh || curl -X POST https://alerts.example.com/webhook -d "ETL job failed"

# Log execution time for performance monitoring
0 * * * * start=$(date +%s); /path/to/etl_job.sh; end=$(date +%s); echo "Execution time: $((end-start)) seconds" >> /var/log/etl_performance.log
```

### Best Practices for Cron ETL Jobs

1. **Make scripts idempotent**: Scripts should be safe to run multiple times without causing data duplication or corruption.

2. **Include error handling**: ETL scripts should gracefully handle errors and report issues.

3. **Use absolute paths**: Always use full paths to scripts and files to avoid path-related issues.

4. **Set appropriate timeouts**: Prevent jobs from running indefinitely with tools like `timeout`.

5. **Document schedules**: Maintain documentation of all scheduled jobs, their dependencies, and expected runtimes.

6. **Implement proper logging**: Structured logging helps with debugging and monitoring.

7. **Consider time zones**: Be aware of system time zone settings, especially for systems spanning multiple regions.

8. **Test thoroughly**: Test cron jobs with realistic data volumes and under failure conditions.

### Advanced ETL Scheduling Solutions

For enterprise ETL pipelines, specialized scheduling tools offer more features than basic cron:

- **Apache Airflow**: Python-based platform for programmatically authoring, scheduling, and monitoring workflows
- **Luigi**: Python module for building complex pipelines of batch jobs with dependency resolution
- **Jenkins**: Automation server that can be used for scheduling and monitoring ETL jobs
- **Control-M**: Enterprise job scheduler with advanced workload automation features

These tools provide benefits like:

- Visual workflow representation
- Dependency management
- Retry logic
- SLA monitoring
- Centralized logging
- Web-based interfaces

For our temperature monitoring example, a robust implementation using advanced features might look like:

```bash
#!/bin/bash
# Script: temp_etl_robust.sh

# Set up logging
LOG_FILE="/var/log/temp_etl.log"
exec > >(tee -a ${LOG_FILE}) 2>&1
echo "$(date): Starting temperature ETL process"

# Create lock file to prevent overlapping executions
LOCK_FILE="/var/lock/temp_etl.lock"
if ! flock -n 200 ; then
    echo "$(date): Another instance is running. Exiting."
    exit 1
fi

# Error handling function
handle_error() {
    echo "$(date): ERROR - $1"
    # Send alert
    curl -s -X POST https://alerts.example.com/webhook -d "Temperature ETL error: $1"
    exit 1
}

# Extract: Get temperature data with retry logic
MAX_RETRIES=3
retry=0
while [ $retry -lt $MAX_RETRIES ]; do
    echo "$(date): Extracting temperature data (attempt $((retry+1)))"
    temperature_data=$(curl -s --max-time 5 http://sensor-api.example.com/temp)

    if [ $? -eq 0 ] && [ ! -z "$temperature_data" ]; then
        break
    fi

    retry=$((retry+1))
    if [ $retry -lt $MAX_RETRIES ]; then
        echo "$(date): Extraction failed, retrying in 10 seconds"
        sleep 10
    else
        handle_error "Failed to extract temperature data after $MAX_RETRIES attempts"
    fi
done

# Parse temperature value
temp_value=$(echo $temperature_data | jq -r '.temperature')
if [ $? -ne 0 ] || [ -z "$temp_value" ]; then
    handle_error "Failed to parse temperature data"
fi

echo "$(date): Recorded temperature: $temp_value"
echo "$temp_value" >> /var/log/temp_readings.log

# Transform: Calculate statistics
echo "$(date): Calculating temperature statistics"
if [ $(wc -l < /var/log/temp_readings.log) -lt 60 ]; then
    echo "$(date): Warning - Less than 60 readings available"
fi

stats=$(tail -n 60 /var/log/temp_readings.log |
    awk 'BEGIN {min=999; max=-999}
    {sum+=$1; if($1<min) min=$1; if($1>max) max=$1}
    END {printf "%.2f,%.2f,%.2f\n", sum/NR, min, max}')

echo "$(date): Statistics calculated: $stats"
echo "$stats" > /tmp/temp_stats.csv

# Load: Insert into database with error handling
echo "$(date): Loading data into database"
if ! cat /tmp/temp_stats.csv |
    psql -v ON_ERROR_STOP=1 -c "COPY temp_hourly_stats(avg_temp, min_temp, max_temp) FROM STDIN WITH CSV" mydatabase; then
    handle_error "Failed to load data into database"
fi

# Record successful completion
echo "$(date): Temperature ETL process completed successfully"
date +%s > /var/run/temp_etl.timestamp

# Release lock
} 200>${LOCK_FILE}
```

This enhanced script includes error handling, logging, retry logic, and locks - all critical components for reliable scheduled ETL pipelines.

## Database Connections in ETL Pipelines

Loading data into databases is a critical final step in ETL pipelines. Properly handling database connections, credentials, and data loading requires special attention to security and efficiency.

### Managing Database Credentials

There are several approaches to handling database credentials in ETL scripts:

#### Using Environment Variables

Environment variables are the preferred method for storing sensitive connection information:

```bash
# Set environment variables in your script
export PGHOST="database.example.com"
export PGPORT="5432"
export PGDATABASE="analytics"
export PGUSER="etl_user"
export PGPASSWORD="secure_password"

# Use the environment variables with psql
psql -c "SELECT count(*) FROM events"

# Clear sensitive variables when done
unset PGPASSWORD
```

#### Using Password Files

For PostgreSQL, you can use a .pgpass file to avoid storing passwords in scripts:

```bash
# Create a .pgpass file in the user's home directory
echo "database.example.com:5432:analytics:etl_user:secure_password" > ~/.pgpass
chmod 600 ~/.pgpass  # Restrict permissions

# Now you can connect without specifying password
psql -h database.example.com -U etl_user -d analytics
```

#### Using Connection Strings

Connection strings provide a compact way to specify all connection parameters:

```bash
# Full connection string with all parameters
CONNECTION_STRING="postgresql://etl_user:secure_password@database.example.com:5432/analytics"

# Use with tools that support connection strings
psql "${CONNECTION_STRING}" -c "SELECT count(*) FROM events"
```

### Loading Data into PostgreSQL

PostgreSQL offers several efficient methods for loading data:

#### Using COPY Command

The COPY command is the fastest way to load data into PostgreSQL:

```bash
# Load CSV file directly from filesystem (requires server access)
psql -c "COPY target_table FROM '/path/to/data.csv' WITH CSV HEADER" mydatabase

# Load CSV from stdin through a pipe
cat data.csv | psql -c "COPY target_table FROM STDIN WITH CSV HEADER" mydatabase

# With field delimiter and NULL handling
cat data.csv | psql -c "COPY target_table FROM STDIN WITH CSV HEADER DELIMITER ',' NULL 'NULL'" mydatabase
```

#### Using \copy Meta-Command

When you don't have direct server filesystem access, use the `\copy` meta-command:

```bash
# Create a SQL script with \copy command
echo "\copy target_table FROM 'data.csv' WITH CSV HEADER" > load_data.sql

# Execute the script
psql -f load_data.sql mydatabase
```

#### Handling Errors During Load

For robust ETL, handle errors during data loading:

```bash
# Stop on first error
psql -v ON_ERROR_STOP=1 -c "COPY target_table FROM STDIN WITH CSV" mydatabase

# Log rejected rows to a file
psql -c "COPY target_table FROM STDIN WITH CSV LOG ERRORS INTO error_table REJECT LIMIT 10" mydatabase
```

### Complete PostgreSQL ETL Example

Here's a complete example of extracting data, transforming it, and loading it into PostgreSQL:

```bash
#!/bin/bash
# extract_transform_load.sh

# Set up database connection securely
export PGHOST="db.example.com"
export PGPORT="5432"
export PGDATABASE="analytics"
export PGUSER="etl_user"

# Use password from a separate secured file
export PGPASSWORD=$(cat ~/.secure/db_password)

# Extract data from API
echo "Extracting data from API..."
curl -s "https://api.example.com/data" | jq '.' > raw_data.json

# Transform data - convert JSON to CSV
echo "Transforming data..."
jq -r '.records[] | [.id, .timestamp, .value] | @csv' raw_data.json > transformed_data.csv

# Create temporary table for staging
echo "Creating staging table..."
psql -c "
  CREATE TEMP TABLE staging_data (
    id VARCHAR(50),
    timestamp TIMESTAMP,
    value NUMERIC
  );
"

# Load data into staging table
echo "Loading data into staging table..."
cat transformed_data.csv | psql -c "
  COPY staging_data FROM STDIN WITH CSV
"

# Merge data into target table (upsert pattern)
echo "Merging data into target table..."
psql -c "
  INSERT INTO production_data (id, timestamp, value)
  SELECT id, timestamp, value FROM staging_data
  ON CONFLICT (id) DO UPDATE
  SET timestamp = EXCLUDED.timestamp,
      value = EXCLUDED.value
"

# Verify row count
row_count=$(psql -t -c "SELECT COUNT(*) FROM staging_data")
echo "Loaded ${row_count} rows"

# Clean up
rm raw_data.json transformed_data.csv
unset PGPASSWORD

echo "ETL process completed successfully"
```

## Essential Linux Utilities for ETL

Beyond the core text processing commands, many other Linux utilities are invaluable for ETL pipelines.

### File Download and Transfer

#### wget and curl

```bash
# Download a file with wget
wget https://example.com/data/dataset.csv

# Download with authentication
wget --user=username --password=password https://example.com/data/secure_dataset.csv

# Recursive download with limits
wget --recursive --level=1 --accept=csv https://example.com/data/

# Using curl for downloads
curl -o dataset.csv https://example.com/data/dataset.csv

# With authentication and headers
curl -u username:password -H "Accept: application/json" https://api.example.com/data
```

#### scp and rsync

```bash
# Copy file from remote server
scp user@remote-server:/path/to/data.csv local_data.csv

# Copy entire directory
scp -r user@remote-server:/path/to/data_dir/ local_data_dir/

# Using rsync for efficient transfers
rsync -avz user@remote-server:/path/to/data_dir/ local_data_dir/

# Rsync with deletion (mirror)
rsync -avz --delete user@remote-server:/path/to/data_dir/ local_data_dir/
```

### File Compression and Archiving

#### gzip, gunzip, and zcat

```bash
# Compress a file
gzip large_file.csv  # Creates large_file.csv.gz

# Decompress a file
gunzip large_file.csv.gz

# View compressed file without decompressing
zcat large_file.csv.gz | head

# Process compressed file directly
zcat large_file.csv.gz | grep "pattern" > filtered_data.csv
```

#### tar

```bash
# Create a compressed archive
tar -czf archive.tar.gz directory/

# Extract a compressed archive
tar -xzf archive.tar.gz

# List contents of an archive
tar -tzf archive.tar.gz

# Extract specific files
tar -xzf archive.tar.gz directory/specific_file.csv
```

### Date and Time Handling

Date manipulation is essential for ETL scheduling and incremental loads:

```bash
# Get current date in YYYY-MM-DD format
today=$(date +%Y-%m-%d)

# Get yesterday's date
yesterday=$(date -d "yesterday" +%Y-%m-%d)
# Or on BSD/macOS:
yesterday=$(date -v-1d +%Y-%m-%d)

# Format timestamp for filenames
timestamp=$(date +%Y%m%d_%H%M%S)
output_file="data_extract_${timestamp}.csv"

# Calculate date ranges
start_date=$(date -d "7 days ago" +%Y-%m-%d)
end_date=$(date +%Y-%m-%d)

# Use in SQL queries
psql -c "SELECT * FROM events WHERE date BETWEEN '${start_date}' AND '${end_date}'"
```

### Data Validation and Integrity

#### md5sum and sha256sum

```bash
# Generate checksum for a file
md5sum data.csv > data.csv.md5

# Verify file integrity
md5sum -c data.csv.md5

# More secure SHA-256 checksum
sha256sum data.csv > data.csv.sha256

# Verify using SHA-256
sha256sum -c data.csv.sha256
```

#### wc (word count)

```bash
# Count lines in a file
wc -l data.csv

# Verify expected row count
expected_rows=1000
actual_rows=$(wc -l < data.csv)
if [ "$actual_rows" -ne "$expected_rows" ]; then
  echo "Error: Expected $expected_rows rows, got $actual_rows"
  exit 1
fi
```

### Network Utilities

#### netcat (nc)

```bash
# Test database connectivity
nc -zv database.example.com 5432

# Simple data transfer
cat data.csv | nc receiver.example.com 9876
```

#### curl for API interaction

```bash
# POST data to an API
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com/endpoint

# POST form data
curl -X POST -F "file=@data.csv" https://api.example.com/upload

# Basic authentication
curl -u username:password https://api.example.com/secure-endpoint
```

### Advanced Shell Features

#### Process Substitution

```bash
# Compare output of two commands
diff <(grep "ERROR" log1.txt) <(grep "ERROR" log2.txt)

# Join files with different delimiters
join <(cut -d, -f1,2 file1.csv) <(cut -d\| -f1,3 file2.txt)
```

#### Parallel Execution

```bash
# Using GNU Parallel to process multiple files
parallel --jobs 4 "process_file.sh {}" ::: file1.csv file2.csv file3.csv file4.csv

# Process chunks of a large file
split -l 100000 large_file.csv chunk_
parallel "process_chunk.sh {}" ::: chunk_*
```

### Complete ETL Utility Example

Here's an example ETL script combining various utilities:

```bash
#!/bin/bash
# comprehensive_etl.sh

set -e  # Exit on any error

# Setup
today=$(date +%Y-%m-%d)
log_file="etl_${today}.log"
exec > >(tee -a "$log_file") 2>&1  # Log all output

echo "=== ETL Process Started at $(date) ==="

# Download data securely
echo "Downloading data..."
curl -s -u "$API_USER:$API_PASSWORD" \
     "https://api.example.com/data?date=${today}" \
     -o "raw_data_${today}.json"

# Validate download
if [[ ! -s "raw_data_${today}.json" ]]; then
    echo "Error: Downloaded file is empty"
    exit 1
fi

# Transform JSON to CSV
echo "Transforming data..."
jq -r '.data[] | [.id, .timestamp, .value] | @csv' "raw_data_${today}.json" > "data_${today}.csv"

# Compress previous day's data
yesterday=$(date -d "yesterday" +%Y-%m-%d)
if [[ -f "data_${yesterday}.csv" ]]; then
    echo "Archiving previous data..."
    gzip -f "data_${yesterday}.csv"
fi

# Count records
record_count=$(wc -l < "data_${today}.csv")
echo "Processing ${record_count} records"

# Data validation
echo "Validating data..."
if grep -q ",NULL," "data_${today}.csv"; then
    echo "Warning: NULL values detected"
fi

# Split large files for parallel processing
if [[ "$record_count" -gt 1000000 ]]; then
    echo "Splitting file for parallel processing..."
    mkdir -p chunks
    split -l 100000 "data_${today}.csv" chunks/chunk_

    echo "Processing chunks in parallel..."
    find chunks -name "chunk_*" | parallel -j 4 "./process_chunk.sh {}"

    echo "Combining results..."
    cat chunks/processed_* > "processed_${today}.csv"
    rm -rf chunks
else
    echo "Processing file..."
    ./process_chunk.sh "data_${today}.csv" > "processed_${today}.csv"
fi

# Load into database
echo "Loading data into database..."
export PGPASSWORD=$(cat ~/.pgpass | grep analytics | cut -d: -f5)
psql -h db.example.com -U etl_user -d analytics << EOF
BEGIN;
CREATE TEMP TABLE temp_import (LIKE target_table);
\copy temp_import FROM 'processed_${today}.csv' WITH CSV HEADER
INSERT INTO target_table
SELECT * FROM temp_import
ON CONFLICT (id) DO UPDATE
SET value = EXCLUDED.value,
    updated_at = NOW();
COMMIT;
EOF
unset PGPASSWORD

# Verify load
load_count=$(psql -h db.example.com -U etl_user -d analytics -t -c "SELECT COUNT(*) FROM target_table WHERE date_trunc('day', created_at) = '${today}'")
echo "Loaded ${load_count} records into database"

# Generate checksums for audit
md5sum "data_${today}.csv" "processed_${today}.csv" > "checksums_${today}.md5"

echo "=== ETL Process Completed at $(date) ==="
```

These utilities and examples demonstrate the power and flexibility of Linux command-line tools for building robust ETL pipelines, from secure database connections to efficient file processing and comprehensive logging.
