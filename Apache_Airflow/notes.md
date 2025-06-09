# Intro

Open-source (great support community) platform for orchestrating complex computational workflows (DAGs). It lets you build and run workflows.

Note: Airflow is not a workflow engine, it is a workflow orchestrator. And, it is not a data streaming solution.

## Architecture

Apache Airflow's architecture consists of several key components that work together to provide robust workflow orchestration:

### Core Components

1. **Scheduler**

   - Monitors all tasks and DAGs
   - Triggers task instances once their dependencies are complete
   - Submits tasks to the executor to run
   - Handles task retries and backfills
   - Runs as a service that determines which tasks need to be run when

2. **Metadata Database**

   - Central repository for Airflow's state
   - Stores DAG runs, task instances, variables, connections, and configurations
   - Supports multiple database backends (PostgreSQL, MySQL, SQLite)
   - Maintains historical execution data for analytics and monitoring
   - Enables the web server to display the current state of all workflows

3. **Workers/Executors**

   - Execute the tasks assigned by the scheduler
   - Various executor types available:
     - SequentialExecutor: Runs one task at a time (default with SQLite)
     - LocalExecutor: Runs tasks using parallel processes on a single machine
     - CeleryExecutor: Distributes tasks across multiple worker nodes using Celery
     - KubernetesExecutor: Dynamically launches pods for each task on Kubernetes
     - DaskExecutor: Uses Dask distributed clusters for task execution

4. **Web Server**

   - Provides a user-friendly UI to monitor and manage workflows
   - Visualizes DAG structure and dependencies
   - Shows historical runs, logs, and task status
   - Allows manual triggering, pausing, and resuming of DAGs
   - Exposes a REST API for programmatic interaction

5. **DAG Directory**
   - Contains Python files defining the workflows
   - Scanned periodically by the scheduler for changes
   - Enables version control and collaboration through standard code practices

### Data Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  DAG Files  │◄────┤  Scheduler  │────►│  Metadata   │
└─────────────┘     └──────┬──────┘     │  Database   │
                           │            └──────┬──────┘
                           ▼                   │
                    ┌─────────────┐            │
                    │  Executor   │            │
                    └──────┬──────┘            │
                           │                   │
                           ▼                   │
                    ┌─────────────┐            │
                    │   Workers   │            │
                    └──────┬──────┘            │
                           │                   │
                           │            ┌──────▼──────┐
                           └───────────►│  Web Server │
                                        └─────────────┘
```

## How Airflow Works

Airflow's operation follows a straightforward yet powerful workflow model:

1. **DAG Definition**

   - Users define DAGs (Directed Acyclic Graphs) in Python
   - Each DAG represents a collection of tasks with their dependencies
   - Example:
     ```python
     with DAG('example_dag', start_date=datetime(2023, 1, 1),
              schedule_interval='@daily') as dag:
         task_1 = PythonOperator(task_id='extract', python_callable=extract_data)
         task_2 = PythonOperator(task_id='transform', python_callable=transform_data)
         task_3 = PythonOperator(task_id='load', python_callable=load_data)

         # Define dependencies
         task_1 >> task_2 >> task_3
     ```

2. **DAG Parsing**

   - Scheduler parses DAG files to create DAG objects
   - DAGs are stored in the metadata database
   - The scheduler determines which DAGs need to be run based on their schedule

3. **Task Scheduling**

   - When a DAG run is created, task instances are created for each task
   - The scheduler monitors task dependencies
   - When all upstream dependencies are met, tasks are queued for execution

4. **Task Execution**

   - Executor picks up queued tasks and distributes them to workers
   - Workers execute the tasks and report status back to the metadata database
   - Tasks can run in parallel if dependencies allow

5. **Monitoring and Management**
   - Web server provides real-time visibility into task/DAG status
   - Users can monitor progress, view logs, and troubleshoot issues
   - REST API enables programmatic control and integration

## Task Lifecycle

Tasks in Airflow follow a defined lifecycle with multiple possible states:

1. **No Status**

   - Initial state before the scheduler processes the task
   - No record exists in the metadata database yet

2. **Scheduled**

   - Task has been scheduled for execution
   - Waiting for the executor to pick it up

3. **Queued**

   - Task has been sent to the executor
   - Waiting for a worker to execute it

4. **Running**

   - Task is currently being executed by a worker
   - Logs are being generated and stored

5. **Success**

   - Task completed successfully
   - Downstream tasks can now be triggered

6. **Failed**

   - Task execution ended with an error
   - Can trigger failure callbacks or notifications
   - May prevent downstream tasks from running

7. **Retrying**

   - Task failed but has remaining retry attempts
   - Will be rescheduled based on retry policy

8. **Skipped**

   - Task was intentionally skipped (e.g., by a conditional operator)
   - Not considered failed or successful for dependency purposes

9. **Up for Reschedule**

   - Special state for sensors that didn't meet their criteria
   - Task will be rescheduled to check again later

10. **Upstream Failed**

    - Task wasn't run because an upstream dependency failed
    - Won't be attempted until the upstream task succeeds

11. **Removed**
    - Task exists in the database but is no longer in the DAG
    - Usually happens when DAG definitions change

## Advantages of Airflow

Airflow offers numerous advantages that have made it the industry standard for workflow orchestration:

### Scalability

- Handles workflows with thousands of tasks
- Distributes workloads across multiple workers
- Supports horizontal scaling through Celery or Kubernetes
- Processes millions of task instances efficiently
- Manages complex dependency trees without performance degradation

### Extensibility

- **Custom Operators**: Create specialized task types for specific use cases
- **Custom Hooks**: Build reusable connections to external systems
- **Custom Executors**: Implement custom task distribution logic
- **Plugins**: Extend the web UI and add new features
- **Provider Packages**: Community-maintained integrations with external systems

### Flexibility

- Define workflows using Python code, not restrictive YAML or XML
- Implement complex business logic and conditional branching
- Use dynamic task generation based on runtime parameters
- Create task dependencies using multiple approaches:

  ```python
  # Method 1: Bitshift operators
  task_1 >> task_2 >> task_3

  # Method 2: set_upstream/set_downstream
  task_3.set_upstream(task_2)
  task_2.set_upstream(task_1)

  # Method 3: List syntax with chain
  from airflow.models.baseoperator import chain
  chain(task_1, task_2, task_3)
  ```

### Rich User Interface

- Visualize DAG structure and dependencies in an intuitive graph view
- Monitor task execution in real-time with color-coded status indicators
- View detailed logs directly in the browser
- Track historical performance and execution times
- Manage connections, variables, and XComs through the UI
- Trigger, pause, or clear DAG runs with a few clicks

Since DAGs and workflows are defined in Python, you can use all the power of Python to define your workflows. You can use any Python library, and you can even call external APIs or services. Plus, since its code, you can maintain, version, collaborate and test your workflows just like any other code!

## Python-Based Workflow Definition

Airflow's Python-centric approach provides powerful capabilities that distinguish it from other workflow tools:

### Leveraging Python's Ecosystem

- **Library Integration**: Seamlessly incorporate thousands of Python packages

  ```python
  # Using pandas for data transformation
  def transform_data(**context):
      import pandas as pd
      df = pd.read_csv('/tmp/raw_data.csv')
      # Apply transformations
      df['processed'] = df['raw_value'] * 1.5
      df.to_csv('/tmp/processed_data.csv')
  ```

- **API Interaction**: Connect with external services using Python clients

  ```python
  def fetch_weather_data(**context):
      import requests
      response = requests.get(
          'https://api.weather.gov/points/39.7456,-97.0892',
          headers={'Accept': 'application/json'}
      )
      return response.json()
  ```

- **Advanced Data Processing**: Utilize scientific and ML libraries
  ```python
  def train_model(**context):
      from sklearn.ensemble import RandomForestClassifier
      from sklearn.datasets import make_classification
      X, y = make_classification()
      clf = RandomForestClassifier()
      clf.fit(X, y)
      # Save model
      import joblib
      joblib.dump(clf, '/tmp/model.joblib')
  ```

### Software Development Best Practices

- **Version Control**: Store DAGs in Git repositories

  - Track changes over time
  - Revert to previous versions if needed
  - Collaborate with team members through PRs
  - Enforce code reviews before deployment

- **Testing**: Create unit and integration tests for workflows

  ```python
  # test_dag.py
  import unittest
  from airflow.models import DagBag

  class TestMyDAG(unittest.TestCase):
      def test_dag_loaded(self):
          dagbag = DagBag()
          dag = dagbag.get_dag('my_dag')
          self.assertIsNotNone(dag)
          self.assertEqual(len(dag.tasks), 5)
          self.assertEqual(dagbag.import_errors, {})
  ```

- **CI/CD Integration**: Automate deployment of DAGs

  - Validate DAGs syntax before deployment
  - Run tests automatically on commit
  - Deploy to staging then production environments
  - Monitor DAG performance after changes

- **Modularization**: Create reusable components

  ```python
  # In a shared module: common_tasks.py
  def create_data_quality_check(table_name, sql_check):
      return SQLCheckOperator(
          task_id=f'check_{table_name}',
          sql=sql_check,
          conn_id='my_database'
      )

  # In your DAG file
  from common_tasks import create_data_quality_check

  check_orders = create_data_quality_check(
      'orders',
      'SELECT COUNT(*) FROM orders WHERE order_date IS NULL'
  )
  ```

### Dynamic Workflow Generation

- **Parameterized DAGs**: Create templates for similar workflows

  ```python
  for database in ['customers', 'orders', 'products']:
      extract_task = PythonOperator(
          task_id=f'extract_{database}',
          python_callable=extract_data,
          op_kwargs={'database': database}
      )
      transform_task = PythonOperator(
          task_id=f'transform_{database}',
          python_callable=transform_data,
          op_kwargs={'database': database}
      )
      extract_task >> transform_task >> load_task
  ```

- **Configuration as Code**: Store workflow configurations in Python

  ```python
  # config.py
  ETL_CONFIG = {
      'customers': {
          'source': 'mysql',
          'destination': 'bigquery',
          'transformations': ['normalize', 'deduplicate']
      },
      'orders': {
          'source': 'api',
          'destination': 'bigquery',
          'transformations': ['validate', 'aggregate']
      }
  }

  # dag.py
  from config import ETL_CONFIG

  for entity, config in ETL_CONFIG.items():
      # Create dynamic tasks based on configuration
  ```

- **Runtime Task Generation**: Create tasks based on previous task outputs
  ```python
  def create_processing_tasks(**context):
      tables = context['ti'].xcom_pull(task_ids='discover_tables')
      for table in tables:
          process_task = PythonOperator(
              task_id=f'process_{table}',
              python_callable=process_table,
              op_kwargs={'table': table},
              dag=context['dag']
          )
          # Set dependencies
          discover_task >> process_task >> complete_task
  ```

## Platform to programmatically author, schedule and monitor workflows

Airflow provides a comprehensive platform for the entire workflow lifecycle:

### Authoring Workflows

- **Code-First Approach**: Define workflows in Python for maximum flexibility
- **Web UI DAG Creation**: Some deployments support UI-based DAG creation
- **Template System**: Use Jinja templating for dynamic task parameters

  ```python
  templated_command = """
  {% for i in range(5) %}
      echo "{{ ds }}"
      echo "{{ macros.ds_add(ds, 7) }}"
      echo "{{ params.my_param }}"
  {% endfor %}
  """

  t1 = BashOperator(
      task_id='templated',
      bash_command=templated_command,
      params={'my_param': 'Parameter I passed in'},
  )
  ```

- **Variable System**: Store and retrieve configuration values

  ```python
  from airflow.models import Variable

  api_key = Variable.get("api_key", deserialize_json=True)
  ```

- **Connection Management**: Securely store and use connection credentials

  ```python
  from airflow.hooks.base_hook import BaseHook

  conn = BaseHook.get_connection('my_postgres_conn')
  ```

### Scheduling Capabilities

- **Cron-like Scheduling**: Familiar syntax for time-based scheduling
  ```python
  # Run at 10:00 PM every day
  dag = DAG('example', schedule_interval='0 22 * * *')
  ```
- **Preset Intervals**: Human-readable interval definitions
  ```python
  # Options include: @once, @hourly, @daily, @weekly, @monthly, @yearly
  dag = DAG('example', schedule_interval='@daily')
  ```
- **Custom Intervals**: Define intervals using datetime objects

  ```python
  from datetime import timedelta

  dag = DAG('example', schedule_interval=timedelta(hours=6))
  ```

- **Event-Triggered**: Trigger DAGs based on external events (with sensors)
- **Backfilling**: Automatically run missed intervals when catching up
  ```python
  # Will backfill all missed runs since January 1, 2023
  dag = DAG(
      'example',
      start_date=datetime(2023, 1, 1),
      schedule_interval='@daily',
      catchup=True
  )
  ```

### Monitoring Capabilities

- **DAG Visualization**: Interactive graph representation of workflow structure
- **Status Tracking**: Real-time monitoring of task and DAG status
- **Historical Views**: Access to all previous runs and their outcomes
- **Log Inspection**: View logs directly in the UI
  - Stream logs in real-time during execution
  - Download logs for offline analysis
- **Metrics and Analytics**: Track performance metrics over time
  - Task duration
  - Success/failure rates
  - Resource utilization
- **Alerting**: Configure notifications for failures or SLA misses
  ```python
  default_args = {
      'email': ['team@example.com'],
      'email_on_failure': True,
      'email_on_retry': False,
      'sla': timedelta(hours=2),
      'sla_miss_callback': notify_sla_miss
  }
  ```

### Management Features

- **Manual Triggers**: Run DAGs on-demand regardless of schedule
- **Task Reruns**: Retry specific failed tasks without rerunning the entire DAG
- **DAG Pausing**: Temporarily disable DAGs without deleting them
- **Clear Task Instances**: Reset task state to rerun them
- **Mark Success/Failure**: Manually override task status
- **Access Control**: Role-based access control for different user types
  - Viewer: Can only view DAGs and their status
  - User: Can trigger DAGs but not modify them
  - Op: Can perform administrative functions
  - Admin: Full control over all Airflow features

## Main Principles and Features

Apache Airflow is built around several core principles and features that define its functionality and appeal:

### Core Principles

1. **DAGs (Directed Acyclic Graphs)**

   - Fundamental structure representing workflows
   - Directed: Tasks have a clear direction of flow
   - Acyclic: No cycles allowed (prevents infinite loops)
   - Graph: Network of tasks with dependencies

2. **Idempotence**

   - Tasks should produce the same result regardless of how many times they run
   - Critical for reliability and recovery from failures
   - Enables retries and backfilling

3. **Configuration as Code**

   - Workflows defined in Python rather than UI clicks or XML/YAML
   - Enables version control, testing, and CI/CD integration
   - Allows programmatic generation of complex workflows

4. **Extensibility**

   - Plugin architecture for adding custom functionality
   - Ability to create custom operators, hooks, and interfaces
   - Provider packages for third-party integrations

5. **Operability**
   - Rich UI for monitoring and troubleshooting
   - Comprehensive logging and alerting
   - Metrics for performance monitoring

### DAG Structure and Definition

DAGs are Python scripts that follow a specific structure:

```python
# 1. Library Imports
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

# 2. DAG Arguments
default_args = {
    'owner': 'data_team',
    'depends_on_past': False,
    'email_on_failure': True,
    'email': ['alerts@example.com'],
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

# 3. DAG Definition
with DAG(
    'example_etl_workflow',
    default_args=default_args,
    description='An example ETL workflow',
    schedule_interval='0 0 * * *',
    start_date=datetime(2023, 1, 1),
    catchup=False,
    tags=['example', 'etl'],
) as dag:

    # 4. Task Definitions
    extract = BashOperator(
        task_id='extract',
        bash_command='python /scripts/extract.py',
    )

    transform = PythonOperator(
        task_id='transform',
        python_callable=transform_function,
    )

    load = BashOperator(
        task_id='load',
        bash_command='python /scripts/load.py',
    )

    # 5. Task Pipeline (Dependencies)
    extract >> transform >> load
```

### Task Types and Operators

Airflow provides a wide variety of operators for different task types:

1. **Action Operators**

   - **BashOperator**: Executes bash commands
     ```python
     task = BashOperator(
         task_id='print_date',
         bash_command='date',
     )
     ```
   - **PythonOperator**: Calls Python functions

     ```python
     def my_function(**context):
         return "Hello World"

     task = PythonOperator(
         task_id='python_task',
         python_callable=my_function,
     )
     ```

   - **SQLOperator**: Executes SQL queries
     ```python
     task = SQLExecuteQueryOperator(
         task_id='create_table',
         sql="CREATE TABLE IF NOT EXISTS users (id INT, name VARCHAR(50))",
         conn_id='postgres_default',
     )
     ```
   - **DockerOperator**: Runs a command inside a Docker container
   - **KubernetesOperator**: Creates Kubernetes pods
   - **EMROperator**: Manages AWS EMR clusters
   - Many more specific to various services and platforms

2. **Transfer Operators**

   - Move data between systems
   - Examples: `S3ToRedshiftOperator`, `MySQLToHiveTransfer`, etc.
   - Handle format conversion and connection management

3. **Sensors**
   - Wait for conditions to be met before proceeding
   - **FileSensor**: Waits for a file to land in a location
     ```python
     task = FileSensor(
         task_id='wait_for_file',
         filepath='/data/events.json',
         poke_interval=60,  # seconds
     )
     ```
   - **S3KeySensor**: Waits for an object in S3
   - **ExternalTaskSensor**: Waits for a task in another DAG
   - **HttpSensor**: Waits for an HTTP endpoint to return a specific result
   - **SqlSensor**: Waits for a SQL query to return results

### Task Dependencies

Tasks in Airflow can be linked to define the execution order:

1. **Bitshift Operators** (most common)

   ```python
   # Linear dependency
   task_1 >> task_2 >> task_3

   # Fan-out
   task_1 >> [task_2, task_3, task_4]

   # Fan-in
   [task_2, task_3, task_4] >> task_5

   # Complex dependencies
   task_1 >> [task_2, task_3] >> task_4
   task_1 >> task_5 >> task_4
   ```

2. **Set Methods**

   ```python
   # Equivalent to task_1 >> task_2
   task_2.set_upstream(task_1)
   task_1.set_downstream(task_2)
   ```

3. **Chain Function**

   ```python
   from airflow.models.baseoperator import chain

   # Equivalent to task_1 >> task_2 >> task_3
   chain(task_1, task_2, task_3)
   ```

4. **Cross-Downstream Function**

   ```python
   from airflow.models.baseoperator import cross_downstream

   # Creates dependencies: t1 >> [t2, t3], t4 >> [t2, t3]
   cross_downstream([t1, t4], [t2, t3])
   ```

### XComs (Cross-Communication)

XComs allow tasks to exchange data:

```python
# Task that pushes data
def push_data(**context):
    value = {"key": "value"}
    context['ti'].xcom_push(key='my_data', value=value)

# Task that pulls data
def pull_data(**context):
    value = context['ti'].xcom_pull(task_ids='push_task', key='my_data')
    print(f"Retrieved value: {value}")

push_task = PythonOperator(
    task_id='push_task',
    python_callable=push_data,
)

pull_task = PythonOperator(
    task_id='pull_task',
    python_callable=pull_data,
)

push_task >> pull_task
```

## List common use cases

Airflow excels in a variety of data orchestration scenarios across industries. Here are the most common use cases:

### ETL/ELT Data Pipelines

Airflow's most common application is orchestrating data movement and transformation:

1. **Traditional ETL Workflows**

   - Extract data from multiple sources
   - Transform data using various processing frameworks
   - Load data into data warehouses or data lakes
   - Example:

     ```python
     extract_task = PythonOperator(
         task_id='extract_from_mysql',
         python_callable=extract_mysql_data,
     )

     transform_task = SparkSubmitOperator(
         task_id='transform_with_spark',
         application='/jobs/transformation.py',
         conn_id='spark',
     )

     load_task = S3ToRedshiftOperator(
         task_id='load_to_redshift',
         schema='public',
         table='customers',
         s3_bucket='data-lake',
         s3_key='transformed/customers.csv',
         copy_options=['CSV'],
     )

     extract_task >> transform_task >> load_task
     ```

2. **Modern ELT Pipelines**

   - Load raw data into cloud data warehouses
   - Transform data using SQL within the warehouse
   - Create data models for analytics and reporting
   - Example tools: Snowflake, BigQuery, Redshift, dbt

3. **Data Lake Management**
   - Ingest raw data into low-cost storage
   - Catalog and organize data for discoverability
   - Process data using distributed frameworks
   - Serve processed data to downstream consumers

### Machine Learning Pipelines

Airflow orchestrates the entire ML lifecycle:

1. **Data Preparation and Feature Engineering**

   - Data collection and cleaning
   - Feature extraction and transformation
   - Feature selection
   - Dataset splitting and validation

2. **Model Training and Evaluation**

   - Hyperparameter tuning
   - Cross-validation
   - Model training on distributed infrastructure
   - Model evaluation and metrics collection
   - Example:

     ```python
     preprocess_task = PythonOperator(
         task_id='preprocess_data',
         python_callable=preprocess_function,
     )

     train_task = KubernetesPodOperator(
         task_id='train_model',
         namespace='ml-training',
         image='ml-training:latest',
         cmds=['python', 'train.py'],
         arguments=['--data-path', '/data/processed.csv'],
     )

     evaluate_task = PythonOperator(
         task_id='evaluate_model',
         python_callable=evaluate_model,
     )

     preprocess_task >> train_task >> evaluate_task
     ```

3. **Model Deployment and Monitoring**
   - Model versioning and artifact management
   - A/B testing
   - Scheduled model retraining
   - Model performance monitoring

### Data Warehouse Maintenance

Airflow helps maintain healthy and efficient data warehouses:

1. **Table Optimization**

   - Vacuum operations
   - Table statistics updates
   - Partition management
   - Example:
     ```python
     vacuum_task = PostgresOperator(
         task_id='vacuum_analyze',
         sql="VACUUM ANALYZE sales_table;",
         postgres_conn_id='redshift',
     )
     ```

2. **Incremental Loads**

   - Delta detection
   - Change data capture (CDC)
   - Incremental processing

3. **Data Quality Checks**
   - Schema validation
   - Data completeness checks
   - Business rule validation
   - Example:
     ```python
     check_task = SQLCheckOperator(
         task_id='check_no_nulls',
         sql="SELECT COUNT(*) FROM users WHERE email IS NULL",
         conn_id='warehouse',
         tolerance=0,  # Fail if any nulls found
     )
     ```

### Report Generation and Data Products

Airflow schedules and orchestrates reporting and analytics:

1. **Scheduled Business Reports**

   - Regular financial reports
   - KPI dashboards
   - Customer analytics
   - Example:

     ```python
     generate_report = PythonOperator(
         task_id='generate_monthly_report',
         python_callable=generate_monthly_financial_report,
         op_kwargs={'month': '{{ macros.ds_format(ds, "%Y-%m-%d", "%Y-%m") }}'},
     )

     email_report = EmailOperator(
         task_id='email_report',
         to=['finance@example.com'],
         subject='Monthly Financial Report - {{ macros.ds_format(ds, "%Y-%m-%d", "%B %Y") }}',
         html_content='Please find attached the monthly report.',
         files=['/reports/finance_{{ macros.ds_format(ds, "%Y-%m-%d", "%Y_%m") }}.pdf'],
     )

     generate_report >> email_report
     ```

2. **Data API Maintenance**

   - Refresh cached data
   - Update materialized views
   - Rebuild search indices

3. **Business Intelligence Updates**
   - Populate OLAP cubes
   - Update visualization platforms
   - Refresh dashboard data

### Cloud Resource Management

Airflow orchestrates cloud infrastructure:

1. **Ephemeral Compute Resources**

   - Start/stop compute clusters (Spark, Hadoop)
   - Scale resources based on workload
   - Terminate resources after use
   - Example:

     ```python
     create_cluster = EmrCreateJobFlowOperator(
         task_id='create_emr_cluster',
         job_flow_overrides={
             'Name': 'Data Processing Cluster',
             'ReleaseLabel': 'emr-6.3.0',
             'Instances': {
                 'InstanceGroups': [
                     {
                         'Name': 'Master nodes',
                         'Market': 'ON_DEMAND',
                         'InstanceRole': 'MASTER',
                         'InstanceType': 'm5.xlarge',
                         'InstanceCount': 1,
                     }
                 ],
                 'KeepJobFlowAliveWhenNoSteps': True,
                 'TerminationProtected': False,
             },
         },
         aws_conn_id='aws_default',
     )

     process_data = EmrAddStepsOperator(
         task_id='add_steps',
         job_flow_id="{{ task_instance.xcom_pull(task_ids='create_emr_cluster', key='return_value') }}",
         steps=[{
             'Name': 'Process Data',
             'ActionOnFailure': 'CONTINUE',
             'HadoopJarStep': {
                 'Jar': 'command-runner.jar',
                 'Args': ['spark-submit', '--deploy-mode', 'cluster', 's3://bucket/script.py'],
             },
         }],
     )

     terminate_cluster = EmrTerminateJobFlowOperator(
         task_id='terminate_emr_cluster',
         job_flow_id="{{ task_instance.xcom_pull(task_ids='create_emr_cluster', key='return_value') }}",
     )

     create_cluster >> process_data >> terminate_cluster
     ```

2. **Infrastructure as Code**
   - Provision and deprovision resources
   - Update configurations
   - Manage environment scaling

### Data Migration and Integration

Airflow facilitates complex data movement between systems:

1. **System Migrations**

   - Legacy system to cloud migrations
   - Database platform migrations
   - Application data migrations

2. **API Integrations**

   - Extract data from third-party APIs
   - Sync data between SaaS applications
   - Manage API rate limits and pagination
   - Example:

     ```python
     def fetch_from_api(endpoint, **context):
         import requests
         response = requests.get(
             f"https://api.example.com/{endpoint}",
             headers={"Authorization": "Bearer " + Variable.get("api_token")}
         )
         return response.json()

     fetch_users = PythonOperator(
         task_id='fetch_users',
         python_callable=fetch_from_api,
         op_kwargs={'endpoint': 'users'},
     )

     fetch_orders = PythonOperator(
         task_id='fetch_orders',
         python_callable=fetch_from_api,
         op_kwargs={'endpoint': 'orders'},
     )

     store_data = PythonOperator(
         task_id='store_data',
         python_callable=store_api_data,
     )

     [fetch_users, fetch_orders] >> store_data
     ```

3. **Multi-system Data Synchronization**
   - Keeping data consistent across platforms
   - Handling different data formats and schemas
   - Managing data versioning and history

### Real-World Example: E-commerce Data Pipeline

A typical e-commerce workflow might include:

```python
with DAG(
    'ecommerce_daily_analytics',
    schedule_interval='0 1 * * *',  # Run at 1 AM daily
    start_date=datetime(2023, 1, 1),
    catchup=False,
) as dag:

    # Wait for transaction data to be available
    check_transaction_data = FileSensor(
        task_id='check_transaction_data',
        filepath='/data/transactions/{{ ds }}.csv',
        poke_interval=300,  # Check every 5 minutes
        timeout=60 * 60 * 2,  # Timeout after 2 hours
    )

    # Load transactions into staging
    load_transactions = PythonOperator(
        task_id='load_transactions',
        python_callable=load_transactions_to_staging,
        op_kwargs={'date': '{{ ds }}'},
    )

    # Load customer data from API
    fetch_customer_data = HttpSensor(
        task_id='check_api_availability',
        http_conn_id='customer_api',
        endpoint='health',
        response_check=lambda response: response.status_code == 200,
        poke_interval=60,
        timeout=60 * 30,
    ) >> PythonOperator(
        task_id='fetch_customer_data',
        python_callable=fetch_customers,
    )

    # Transform and aggregate data
    transform_data = SparkSubmitOperator(
        task_id='transform_data',
        application='/jobs/transform_ecommerce.py',
        application_args=['--date', '{{ ds }}'],
        conn_id='spark',
    )

    # Load to data warehouse
    load_to_warehouse = PostgresOperator(
        task_id='load_to_warehouse',
        sql="""
        INSERT INTO sales_facts
        SELECT * FROM staging.processed_sales
        WHERE date = '{{ ds }}';
        """,
        postgres_conn_id='data_warehouse',
    )

    # Update dashboards
    refresh_dashboards = HttpOperator(
        task_id='refresh_dashboards',
        http_conn_id='tableau_api',
        endpoint='refreshes',
        method='POST',
        data=json.dumps({
            "dashboard_id": "sales-overview",
            "refresh_type": "full"
        }),
        headers={"Content-Type": "application/json"},
    )

    # Send report to stakeholders
    send_report = EmailOperator(
        task_id='send_daily_report',
        to=['sales@example.com', 'marketing@example.com'],
        subject='Daily Sales Report - {{ ds }}',
        html_content='<h3>Daily Sales Report</h3><p>Attached is the daily sales report.</p>',
        files=['/reports/daily_{{ ds }}.pdf'],
    )

    # Define the workflow
    check_transaction_data >> load_transactions
    [load_transactions, fetch_customer_data] >> transform_data >> load_to_warehouse
    load_to_warehouse >> [refresh_dashboards, send_report]
```

This workflow demonstrates how Airflow can coordinate complex multi-system data orchestration, incorporating sensors, various operators, error handling, and parallel task execution.

## Additional Features and Best Practices

Completing our overview of Apache Airflow, let's explore some advanced features and recommended practices:

### Advanced Features

1. **Dynamic DAG Generation**

   - Generate DAGs programmatically based on configuration
   - Use factory patterns to create DAGs from templates
   - Example:

     ```python
     # In a file that will be imported by Airflow
     def create_data_pipeline_dag(source_system, tables):
         dag_id = f"etl_{source_system}"

         with DAG(dag_id, schedule_interval='@daily', ...) as dag:
             for table in tables:
                 extract = PythonOperator(
                     task_id=f"extract_{table}",
                     python_callable=extract_table,
                     op_kwargs={'system': source_system, 'table': table},
                 )
                 # ... more tasks

         return dag

     # Create multiple DAGs
     for source in config['sources']:
         globals()[f"etl_{source['name']}"] = create_data_pipeline_dag(
             source['name'],
             source['tables']
         )
     ```

2. **SubDAGs and TaskGroups**

   - Organize complex workflows into logical groups
   - Improve UI visualization for large DAGs
   - Example:

     ```python
     from airflow.utils.task_group import TaskGroup

     with DAG('example_with_groups', ...) as dag:

         start = DummyOperator(task_id='start')

         with TaskGroup(group_id='extract') as extract_group:
             extract_task1 = PythonOperator(...)
             extract_task2 = PythonOperator(...)

         with TaskGroup(group_id='transform') as transform_group:
             transform_task1 = PythonOperator(...)
             transform_task2 = PythonOperator(...)

         end = DummyOperator(task_id='end')

         start >> extract_group >> transform_group >> end
     ```

3. **Smart Sensors**
   - Reduce Airflow database load by consolidating sensor checks
   - More efficient for workflows with many sensors
   - Example:

     ```python
     from airflow.sensors.smart_sensor import SmartSensorOperator

     # Enable smart sensor in airflow.cfg
     # smart_sensor_directory=/tmp/smart_sensor

     file_sensor = FileSensor(
         task_id='wait_for_file',
         filepath='/data/file.csv',
         mode='reschedule',  # Use reschedule mode for smart sensors
     )
     ```

### Deployment Best Practices

1. **Infrastructure Scaling**

   - Separate scheduler, web server, and worker nodes
   - Scale workers horizontally based on workload
   - Use appropriate executor for your scale:
     - LocalExecutor: Small deployments
     - CeleryExecutor: Medium to large
     - KubernetesExecutor: Dynamic scaling needs

2. **Security Considerations**

   - Use Airflow's RBAC (Role-Based Access Control)
   - Store sensitive information in Secret Backends
   - Implement network security for component communication
   - Example:

     ```python
     # Using secrets backend
     from airflow.hooks.base_hook import BaseHook

     conn = BaseHook.get_connection('my_conn')
     # Credentials retrieved from Secret Manager/Vault
     ```

3. **Monitoring and Observability**
   - Integrate with monitoring systems (Prometheus, Grafana)
   - Set up alerts for DAG failures
   - Track task duration for SLA management
   - Monitor resource utilization

### Development Best Practices

1. **DAG Design Principles**

   - Keep tasks atomic and idempotent
   - Design for failure recovery
   - Use meaningful task IDs
   - Create logical task groups
   - Document DAG purpose and ownership

2. **Testing Strategies**

   - Unit test task functions
   - Integration test DAG structure
   - Use Airflow's testing utilities
   - Example:

     ```python
     # test_dag.py
     import unittest
     from airflow.models import DagBag

     class TestMyDAG(unittest.TestCase):
         def setUp(self):
             self.dagbag = DagBag()

         def test_dag_loaded(self):
             dag = self.dagbag.get_dag('my_dag_id')
             self.assertIsNotNone(dag)
             self.assertEqual(len(dag.tasks), 5)

         def test_dependencies(self):
             dag = self.dagbag.get_dag('my_dag_id')
             task1 = dag.get_task('task1')
             task2 = dag.get_task('task2')
             self.assertIn(task2, task1.downstream_list)
     ```

3. **CI/CD Pipeline Integration**
   - Run tests on commit
   - Lint DAG files for errors
   - Deploy to staging before production
   - Version control all DAGs
