# Structure and Implementation of Airflow DAGs

## Directed Acyclic Graphs (DAGs) Explained

### What is a DAG?

A Directed Acyclic Graph (DAG) is the core concept in Apache Airflow that represents a workflow as a collection of tasks with directional dependencies between them:

- **Directed**: Relationships between tasks have a specific direction (Task A → Task B means Task B depends on Task A).
- **Acyclic**: No cycles are allowed in the graph (Task A → Task B → Task A would create an infinite loop).
- **Graph**: A network structure connecting tasks as nodes with dependencies as edges.

### DAG Structure Characteristics

1. **Start and End Points**

   - Most DAGs have clear starting tasks (no upstream dependencies) and ending tasks (no downstream dependencies)
   - Complex DAGs may have multiple entry and exit points

2. **Task Flow Patterns**

   - **Linear Sequences**: Tasks that must execute in a specific order
     ```
     A → B → C → D
     ```
   - **Fan-Out (Parallelization)**: One task triggers multiple parallel tasks
     ```
           ┌→ B →┐
     A →   │     │ → E
           └→ C →┘
             ↓
             D
     ```
   - **Fan-In (Aggregation)**: Multiple tasks converge to a single downstream task
     ```
     A →┐
        │
     B →├→ D → E
        │
     C →┘
     ```
   - **Branching**: Conditional execution paths
     ```
           ┌→ B →┐
     A →   │     │
           └→ C →┘
     ```

3. **Task Independence**
   - Tasks should be atomic and function independently
   - A task should not rely on the implementation details of other tasks
   - Tasks communicate through XComs when data exchange is necessary

### DAG Execution Properties

1. **Execution Date**

   - Each DAG run is associated with a logical execution date (data interval)
   - Available in templates as `{{ ds }}`, `{{ execution_date }}`, etc.

2. **Scheduling**

   - Triggered based on `schedule_interval` and `start_date`
   - Multiple concurrent DAG runs can exist for different execution dates

3. **Catchup**

   - When `catchup=True`, Airflow will run all missed intervals since `start_date`
   - When `catchup=False`, only the latest interval will be scheduled

4. **Backfilling**
   - Process of running a DAG for historical dates
   - Useful for reprocessing data after code changes or fixing errors

### DAG Definition Best Practices

1. **DAG Organization**

   - One DAG per file (easier to manage and debug)
   - Group related DAGs in subdirectories
   - Use consistent naming conventions

2. **Code Maintainability**

   - Keep DAG files clean and readable
   - Extract complex logic into separate modules
   - Document the purpose and behavior of the DAG

3. **Design Considerations**
   - Design for idempotence (same outcome regardless of retries)
   - Plan for failure recovery
   - Consider performance implications (task granularity)
   - Document dependencies between DAGs

### Visual Example of a DAG

```
                                    ┌─────────────────┐
                                    │ validate_dataset│
                                    └────────┬────────┘
                                             │
                                             ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│extract_data  │───►transform_data│───►load_to_dw    │───►send_report   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                             │
                                             ▼
                                    ┌─────────────────┐
                                    │update_metadata  │
                                    └─────────────────┘
```

## Operator Categories in Airflow

Operators are the building blocks of Airflow DAGs, representing the actual work to be executed. Airflow provides a rich ecosystem of operators that can be categorized as follows:

### 1. Action Operators

These operators perform specific actions or computations.

#### Core Action Operators

- **PythonOperator**: Executes Python functions

  ```python
  from airflow.operators.python import PythonOperator

  def my_python_function(**context):
      # Your Python code here
      return "some_result"

  python_task = PythonOperator(
      task_id='execute_python',
      python_callable=my_python_function,
      op_kwargs={'param1': 'value1'},
      provide_context=True,
  )
  ```

- **BashOperator**: Executes bash commands

  ```python
  from airflow.operators.bash import BashOperator

  bash_task = BashOperator(
      task_id='execute_bash',
      bash_command='echo "Current date: $(date)"',
      env={'ENV_VAR': 'value'},
  )
  ```

- **SQLExecuteQueryOperator**: Runs SQL queries

  ```python
  from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator

  sql_task = SQLExecuteQueryOperator(
      task_id='execute_sql',
      sql="SELECT * FROM users WHERE created_date = '{{ ds }}'",
      conn_id='my_database_connection',
  )
  ```

#### Cloud Provider Operators

- **AWS Operators**

  - `EmrCreateJobFlowOperator`: Creates an EMR cluster
  - `S3CreateBucketOperator`: Creates an S3 bucket
  - `LambdaInvokeOperator`: Invokes an AWS Lambda function

- **GCP Operators**

  - `BigQueryExecuteQueryOperator`: Executes BigQuery SQL
  - `DataprocSubmitJobOperator`: Submits jobs to Dataproc
  - `GCSToGCSOperator`: Copies objects between GCS buckets

- **Azure Operators**
  - `AzureDataFactoryRunPipelineOperator`: Runs ADF pipelines
  - `AzureBlobStorageToGCSOperator`: Transfers data from Azure to GCS

#### Container/Compute Operators

- **DockerOperator**: Runs a command inside a Docker container

  ```python
  from airflow.providers.docker.operators.docker import DockerOperator

  docker_task = DockerOperator(
      task_id='docker_command',
      image='python:3.9-slim',
      command='python -c "import time; print(time.time())"',
      docker_url='unix://var/run/docker.sock',
      network_mode='bridge',
  )
  ```

- **KubernetesPodOperator**: Creates a Kubernetes pod to run a container

  ```python
  from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

  k8s_task = KubernetesPodOperator(
      task_id='kubernetes_pod_example',
      namespace='default',
      image='python:3.9',
      cmds=["python", "-c"],
      arguments=["print('hello world')"],
      name="airflow-test-pod",
  )
  ```

### 2. Transfer Operators

These operators move data between different systems.

- **S3ToRedshiftOperator**: Loads data from S3 to Redshift

  ```python
  from airflow.providers.amazon.aws.transfers.s3_to_redshift import S3ToRedshiftOperator

  s3_to_redshift = S3ToRedshiftOperator(
      task_id='transfer_s3_to_redshift',
      schema='public',
      table='my_table',
      s3_bucket='my-bucket',
      s3_key='path/to/file.csv',
      redshift_conn_id='redshift_conn',
      aws_conn_id='aws_conn',
      copy_options=['CSV', 'IGNOREHEADER 1'],
  )
  ```

- **MySQLToHiveTransfer**: Transfers data from MySQL to Hive
- **GCSToS3Operator**: Transfers files from Google Cloud Storage to Amazon S3
- **FTPToS3Operator**: Transfers files from an FTP server to Amazon S3
- **HttpToGcsOperator**: Downloads data from an HTTP endpoint to Google Cloud Storage

### 3. Sensor Operators

These operators wait for a specific condition to be true before proceeding.

- **FileSensor**: Waits for a file to land at a specific location

  ```python
  from airflow.sensors.filesystem import FileSensor

  file_sensor = FileSensor(
      task_id='wait_for_file',
      filepath='/data/my_file.csv',
      poke_interval=60,  # check every 60 seconds
      timeout=60 * 60 * 5,  # timeout after 5 hours
      mode='poke',  # alternative: 'reschedule'
  )
  ```

- **S3KeySensor**: Waits for a key (file) to be present in an S3 bucket

  ```python
  from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

  s3_sensor = S3KeySensor(
      task_id='wait_for_s3_key',
      bucket_key='data/{{ ds }}/processed.csv',
      bucket_name='my-bucket',
      aws_conn_id='aws_conn',
      wildcard_match=False,
      timeout=60 * 60 * 12,  # 12 hours
  )
  ```

- **ExternalTaskSensor**: Waits for a task in another DAG to complete

  ```python
  from airflow.sensors.external_task import ExternalTaskSensor

  external_task_sensor = ExternalTaskSensor(
      task_id='wait_for_other_dag_task',
      external_dag_id='other_dag',
      external_task_id='final_task',
      execution_delta=timedelta(hours=1),  # look for executions 1 hour earlier
      timeout=60 * 60,
  )
  ```

- **HttpSensor**: Waits for an HTTP endpoint to return a successful response
- **SqlSensor**: Waits for a SQL query to return results

### 4. Operator Subclasses

Operators can also be categorized by their inheritance structure:

- **BaseOperator**: The parent class for all operators
  - Provides common functionality like task IDs, dependencies, and retry behavior
- **TriggerDagRunOperator**: Triggers another DAG

  ```python
  from airflow.operators.trigger_dagrun import TriggerDagRunOperator

  trigger_dag = TriggerDagRunOperator(
      task_id='trigger_another_dag',
      trigger_dag_id='dag_to_trigger',
      conf={'param': 'value'},  # passed to the triggered DAG
      wait_for_completion=True,  # wait until the triggered DAG finishes
      poke_interval=60,  # check every 60 seconds when waiting
  )
  ```

- **BranchPythonOperator**: Determines which path of execution to follow

  ```python
  from airflow.operators.python import BranchPythonOperator

  def branch_func(**context):
      if context['execution_date'].day % 2 == 0:
          return 'even_day_task'
      else:
          return 'odd_day_task'

  branch_op = BranchPythonOperator(
      task_id='branch_task',
      python_callable=branch_func,
  )

  even_day_task = DummyOperator(task_id='even_day_task')
  odd_day_task = DummyOperator(task_id='odd_day_task')

  branch_op >> [even_day_task, odd_day_task]
  ```

### 5. Creating Custom Operators

When existing operators don't meet your needs, you can create custom operators:

```python
from airflow.models.baseoperator import BaseOperator
from airflow.utils.decorators import apply_defaults

class MyCustomOperator(BaseOperator):
    @apply_defaults
    def __init__(self, my_parameter, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.my_parameter = my_parameter

    def execute(self, context):
        # Custom logic here
        self.log.info(f"Executing with parameter: {self.my_parameter}")
        # Return value is pushed to XCom automatically
        return f"Completed execution with {self.my_parameter}"
```

## DAG Arguments Explained

DAG arguments configure the behavior of your DAG. They control scheduling, error handling, and other important aspects of workflow execution.

### Required DAG Arguments

1. **`dag_id`**

   - Unique identifier for the DAG
   - Used in the UI, database, and API
   - Best practice: Use lowercase, underscores, and descriptive names

   ```python
   dag = DAG('marketing_data_pipeline')
   ```

2. **`start_date`**

   - The date from which the DAG should start running
   - First scheduling happens after this date + schedule_interval
   - Usually defined with datetime objects

   ```python
   from datetime import datetime

   dag = DAG(
       'example_dag',
       start_date=datetime(2023, 6, 1)
   )
   ```

### Common Optional DAG Arguments

1. **`schedule_interval`**

   - Defines how often the DAG runs
   - Can be a cron expression, datetime.timedelta, or preset ('`@daily`', '`@hourly`', etc.)

   ```python
   # Run daily at midnight
   dag = DAG('example_dag', schedule_interval='0 0 * * *')

   # Run every 4 hours
   from datetime import timedelta
   dag = DAG('example_dag', schedule_interval=timedelta(hours=4))

   # Preset options
   dag = DAG('example_dag', schedule_interval='@daily')
   ```

   Preset options include:

   - `@once`: Run once and only once
   - `@hourly`: Run once an hour at the beginning of the hour
   - `@daily`: Run once a day at midnight
   - `@weekly`: Run once a week at midnight on Sunday
   - `@monthly`: Run once a month at midnight on the first day
   - `@yearly`: Run once a year at midnight on January 1
   - `None`: Only run when triggered manually

2. **`catchup`**

   - Controls whether Airflow should catch up on missed DAG runs
   - When set to `True`, Airflow will schedule all missed runs since the start_date
   - When set to `False`, only the latest interval will be scheduled

   ```python
   dag = DAG(
       'example_dag',
       start_date=datetime(2023, 1, 1),
       schedule_interval='@daily',
       catchup=False  # Don't process historical runs
   )
   ```

3. **`default_args`**

   - Dictionary of default parameters to apply to all tasks in the DAG
   - Reduces repetition when multiple tasks share the same configuration

   ```python
   default_args = {
       'owner': 'data_engineering',
       'depends_on_past': False,
       'retries': 3,
       'retry_delay': timedelta(minutes=5),
       'email': ['alerts@example.com'],
       'email_on_failure': True,
       'email_on_retry': False,
   }

   dag = DAG(
       'example_dag',
       default_args=default_args,
       start_date=datetime(2023, 1, 1),
       schedule_interval='@daily',
   )
   ```

### Error Handling Arguments

1. **`retries`**

   - Number of times to retry a failed task
   - Can be defined in default_args or per-task

   ```python
   default_args = {
       'retries': 3,  # Retry failed tasks 3 times
   }
   ```

2. **`retry_delay`**

   - Time to wait between retries
   - Can be a fixed or exponential delay

   ```python
   from datetime import timedelta

   default_args = {
       'retry_delay': timedelta(minutes=5),  # Wait 5 minutes between retries
   }

   # Or with exponential backoff
   from airflow.utils.decorators import apply_defaults

   def exponential_backoff_retry_delay(attempt):
       """Return exponentially increasing delay between retries."""
       return timedelta(seconds=2**attempt)

   # Can be applied to a task with custom operator
   ```

3. **`retry_exponential_backoff`**
   - Whether to use exponential backoff for retries
   - Increases the time between retries progressively
   ```python
   default_args = {
       'retry_exponential_backoff': True,
       'max_retry_delay': timedelta(minutes=60),  # Cap the retry delay
   }
   ```

### Dependency Arguments

1. **`depends_on_past`**

   - If True, task instances will run sequentially, waiting for the previous execution to succeed
   - Useful for incremental data processing that depends on the previous run

   ```python
   default_args = {
       'depends_on_past': True,  # Each run depends on the previous one succeeding
   }
   ```

2. **`wait_for_downstream`**
   - If True, a task will wait for tasks immediately downstream to complete before marking itself as complete
   - Useful for ensuring a task's impact is fully processed before proceeding
   ```python
   default_args = {
       'wait_for_downstream': True,
   }
   ```

### Additional Useful Arguments

1. **`tags`**

   - List of tags for organizing and filtering DAGs in the UI
   - Helps with DAG organization and discovery

   ```python
   dag = DAG(
       'example_dag',
       tags=['production', 'marketing', 'hourly'],
   )
   ```

2. **`description`**

   - Text description of the DAG's purpose
   - Displayed in the UI for documentation

   ```python
   dag = DAG(
       'example_dag',
       description='This DAG processes marketing data from various sources and loads it into the data warehouse',
   )
   ```

3. **`max_active_runs`**

   - Maximum number of active DAG runs allowed to run concurrently
   - Prevents overloading resources when catchup is enabled

   ```python
   dag = DAG(
       'example_dag',
       max_active_runs=1,  # Only one DAG run can be active at a time
   )
   ```

4. **`concurrency`**

   - Maximum number of task instances allowed to run concurrently within the DAG
   - Controls parallelism within a single DAG run

   ```python
   dag = DAG(
       'example_dag',
       concurrency=4,  # Only 4 tasks can run in parallel within this DAG
   )
   ```

5. **`orientation`**
   - Default view orientation in the UI
   - Options: 'TB' (top to bottom), 'LR' (left to right), 'RL', 'BT'
   ```python
   dag = DAG(
       'example_dag',
       orientation='LR',  # Display DAG from left to right in the UI
   )
   ```

## Creating Tasks for a DAG

Tasks are the fundamental units of execution in Airflow. Each task is an instance of an operator and represents a single unit of work within your workflow.

### Basic Task Creation

Tasks are created by instantiating an operator class with the required parameters:

```python
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

# Create a bash task
bash_task = BashOperator(
    task_id='hello_world',
    bash_command='echo "Hello, World!"',
    dag=dag,
)

# Create a Python task
def my_python_function(ds, **kwargs):
    print(f"Execution date is: {ds}")
    return "Function executed successfully"

python_task = PythonOperator(
    task_id='python_example',
    python_callable=my_python_function,
    dag=dag,
)
```

### Common Task Parameters

All tasks (regardless of operator type) accept these parameters:

1. **`task_id`**

   - Unique identifier for the task within the DAG
   - Used in logs, UI, and when referencing tasks
   - Should be descriptive and follow a consistent naming convention

2. **`dag`**

   - Reference to the DAG the task belongs to
   - Can be omitted when using the context manager pattern

   ```python
   with DAG('example_dag', ...) as dag:
       # No need to specify dag=dag here
       task = BashOperator(task_id='example', bash_command='echo "Hello"')
   ```

3. **`trigger_rule`**

   - Defines the rule by which the task gets triggered
   - Default is `all_success` (all upstream tasks succeeded)
   - Options include:
     - `all_success`: All upstream tasks succeeded
     - `all_failed`: All upstream tasks failed
     - `all_done`: All upstream tasks are done (regardless of status)
     - `one_success`: At least one upstream task succeeded
     - `one_failed`: At least one upstream task failed
     - `none_failed`: All upstream tasks have not failed (succeeded or skipped)
     - `none_skipped`: No upstream task is in a skipped state

   ```python
   # This task will run if at least one upstream task succeeds
   clean_up_task = BashOperator(
       task_id='clean_up',
       bash_command='echo "Cleaning up..."',
       trigger_rule='one_success',
   )
   ```

4. **`retries`** and **`retry_delay`**

   - Override the DAG-level retry configuration
   - Useful for tasks with different reliability profiles

   ```python
   # This task will retry more times than the DAG default
   critical_task = PythonOperator(
       task_id='critical_operation',
       python_callable=critical_function,
       retries=10,  # More retries for this critical task
       retry_delay=timedelta(minutes=2),
   )
   ```

5. **`priority_weight`**

   - Gives priority to tasks when executor has limited parallelism
   - Tasks with higher priority_weight run first

   ```python
   high_priority_task = BashOperator(
       task_id='high_priority',
       bash_command='echo "Important!"',
       priority_weight=10,  # Higher than default of 1
   )
   ```

6. **`pool`**

   - Assign task to a named pool of workers
   - Controls concurrency for resource-intensive tasks

   ```python
   # Limit concurrent execution of resource-intensive tasks
   heavy_task = PythonOperator(
       task_id='resource_intensive_task',
       python_callable=heavy_processing,
       pool='limited_resources_pool',  # Pool must be defined in Airflow UI/config
   )
   ```

7. **`execution_timeout`**
   - Maximum time a task is allowed to run
   - Task will fail if it exceeds this time
   ```python
   # Task will fail if it runs longer than 30 minutes
   task_with_timeout = BashOperator(
       task_id='bounded_task',
       bash_command='sleep 1800',  # 30 minutes
       execution_timeout=timedelta(minutes=30),
   )
   ```

### Operator-Specific Parameters

Each operator type has its own specific parameters:

1. **PythonOperator**

   ```python
   python_task = PythonOperator(
       task_id='python_task',
       python_callable=my_function,  # Function to execute
       op_kwargs={'param1': 'value1'},  # Keyword arguments for the function
       op_args=['positional1', 'positional2'],  # Positional arguments
       provide_context=True,  # Pass Airflow context to the function (deprecated in newer versions)
   )
   ```

2. **BashOperator**

   ```python
   bash_task = BashOperator(
       task_id='bash_task',
       bash_command='echo "Current date: $(date)"',
       env={'ENV_VAR': 'value'},  # Environment variables
       append_env=True,  # Whether to append rather than overwrite env
       output_encoding='utf-8',  # Encoding of command output
   )
   ```

3. **HTTPOperator**

   ```python
   from airflow.providers.http.operators.http import SimpleHttpOperator

   http_task = SimpleHttpOperator(
       task_id='http_task',
       http_conn_id='http_default',  # Connection defined in Airflow
       endpoint='/api/data',  # Path to append to connection URL
       method='POST',  # HTTP method
       data=json.dumps({"key": "value"}),  # Request payload
       headers={"Content-Type": "application/json"},  # HTTP headers
       response_check=lambda response: response.status_code == 200,  # Validate response
   )
   ```

### Task Groups

For complex DAGs with many tasks, you can organize tasks into logical groups:

```python
from airflow.utils.task_group import TaskGroup

with DAG('example_dag', ...) as dag:

    start = DummyOperator(task_id='start')

    # Create a task group for data extraction
    with TaskGroup(group_id='extract_tasks') as extract_group:
        extract_task1 = PythonOperator(
            task_id='extract_from_api',
            python_callable=extract_from_api,
        )

        extract_task2 = PythonOperator(
            task_id='extract_from_database',
            python_callable=extract_from_database,
        )

    # Create a task group for data transformation
    with TaskGroup(group_id='transform_tasks') as transform_group:
        transform_task1 = PythonOperator(
            task_id='clean_data',
            python_callable=clean_data,
        )

        transform_task2 = PythonOperator(
            task_id='enrich_data',
            python_callable=enrich_data,
        )

    end = DummyOperator(task_id='end')

    # Define dependencies between groups
    start >> extract_group >> transform_group >> end
```

### Dynamic Task Generation

Tasks can be generated dynamically based on configurations or runtime conditions:

```python
tables = ['customers', 'orders', 'products']

with DAG('process_multiple_tables', ...) as dag:
    start = DummyOperator(task_id='start')

    # Dynamically create tasks for each table
    process_tasks = []
    for table in tables:
        extract = PythonOperator(
            task_id=f'extract_{table}',
            python_callable=extract_data,
            op_kwargs={'table': table},
        )

        transform = PythonOperator(
            task_id=f'transform_{table}',
            python_callable=transform_data,
            op_kwargs={'table': table},
        )

        load = PythonOperator(
            task_id=f'load_{table}',
            python_callable=load_data,
            op_kwargs={'table': table},
        )

        # Set dependencies for this table's tasks
        extract >> transform >> load

        # Add the final task to our list
        process_tasks.append(load)

    end = DummyOperator(task_id='end')

    # All table processing must complete before end
    start >> process_tasks >> end
```

## Defining Task Dependencies

Task dependencies determine the order in which tasks execute in a DAG. They define the workflow's structure by creating directional relationships between tasks.

### Bitshift Operators

The most common and readable way to set dependencies uses bitshift operators:

1. **Linear Dependencies**

   ```python
   # task_1 must complete successfully before task_2 begins, and so on
   task_1 >> task_2 >> task_3 >> task_4

   # Equivalent to:
   task_4 << task_3 << task_2 << task_1
   ```

2. **Fan-Out (One-to-Many)**

   ```python
   # task_1 must complete before task_2, task_3, and task_4 can begin
   task_1 >> [task_2, task_3, task_4]

   # Equivalent to:
   task_1 >> task_2
   task_1 >> task_3
   task_1 >> task_4
   ```

3. **Fan-In (Many-to-One)**

   ```python
   # task_4 can only begin after task_1, task_2, and task_3 all complete successfully
   [task_1, task_2, task_3] >> task_4

   # Equivalent to:
   task_1 >> task_4
   task_2 >> task_4
   task_3 >> task_4
   ```

4. **Complex Dependency Chains**

   ```python
   # Combine multiple dependency patterns
   task_1 >> [task_2, task_3] >> task_4
   task_1 >> task_5 >> task_4

   # Creates this structure:
   #    ┌─► task_2 ─┐
   #    │           ▼
   # task_1        task_4
   #    │           ▲
   #    └─► task_3 ─┘
   #    │
   #    └─► task_5 ─► task_4
   ```

### Set Methods

Before bitshift operators, dependencies were set using explicit methods:

```python
# task_1 must complete before task_2 can begin
task_2.set_upstream(task_1)
# or equivalently:
task_1.set_downstream(task_2)

# For multiple dependencies
task_4.set_upstream([task_1, task_2, task_3])
# or equivalently:
task_1.set_downstream(task_4)
task_2.set_downstream(task_4)
task_3.set_downstream(task_4)
```

These methods are still useful when programmatically creating dependencies:

```python
for source_task in source_tasks:
    for target_task in target_tasks:
        target_task.set_upstream(source_task)
```

### Chain and Cross-Downstream Functions

For more complex dependency structures, Airflow provides utility functions:

1. **`chain`**: Create a linear chain of dependencies

   ```python
   from airflow.models.baseoperator import chain

   # Create a linear chain: task_1 >> task_2 >> task_3 >> task_4
   chain(task_1, task_2, task_3, task_4)

   # Chain can also work with lists (unpacking)
   chain(task_1, [task_2a, task_2b, task_2c], task_3)
   # Creates: task_1 >> task_2a >> task_2b >> task_2c >> task_3
   ```

2. **`cross_downstream`**: Create cross-product dependencies between lists

   ```python
   from airflow.models.baseoperator import cross_downstream

   # Create dependencies where each task in the first list is upstream
   # of every task in the second list
   cross_downstream([task_1, task_2], [task_3, task_4, task_5])

   # Equivalent to:
   task_1 >> task_3
   task_1 >> task_4
   task_1 >> task_5
   task_2 >> task_3
   task_2 >> task_4
   task_2 >> task_5

   # Creates this structure:
   #  task_1 ─────┬─────┬─────┐
   #              │     │     │
   #              ▼     ▼     ▼
   #            task_3 task_4 task_5
   #              ▲     ▲     ▲
   #              │     │     │
   #  task_2 ─────┴─────┴─────┘
   ```

### Conditional Dependencies with Branching

Branching allows for conditional execution paths in the DAG:

```python
from airflow.operators.python import BranchPythonOperator

def branch_func(**context):
    if context['execution_date'].day % 2 == 0:
        return 'even_day_task'
    else:
        return 'odd_day_task'

branch_task = BranchPythonOperator(
    task_id='branch_task',
    python_callable=branch_func,
)

even_day_task = BashOperator(
    task_id='even_day_task',
    bash_command='echo "Today is an even day"',
)

odd_day_task = BashOperator(
    task_id='odd_day_task',
    bash_command='echo "Today is an odd day"',
)

# Only one of these will execute based on the branch_func return value
branch_task >> [even_day_task, odd_day_task]

# If you want to continue the flow after the conditional branch
join_task = DummyOperator(
    task_id='join_task',
    trigger_rule='one_success',  # Important: will run after whichever branch executes
)

even_day_task >> join_task
odd_day_task >> join_task
```

### Trigger Rules

Trigger rules modify how a task determines when to execute based on its upstream tasks:

```python
# Default: Run only when all upstream tasks succeed
default_task = DummyOperator(
    task_id='default_task',
    trigger_rule='all_success',  # This is the default
)

# Run when at least one upstream task succeeds
one_success_task = DummyOperator(
    task_id='one_success_task',
    trigger_rule='one_success',
)

# Run when all upstream tasks are done (regardless of success/failure)
cleanup_task = BashOperator(
    task_id='cleanup_task',
    bash_command='echo "Cleaning up resources"',
    trigger_rule='all_done',
)

# Run when all upstream tasks have failed
notification_task = EmailOperator(
    task_id='failure_notification',
    to='team@example.com',
    subject='DAG failed',
    html_content='All tasks have failed, please investigate.',
    trigger_rule='all_failed',
)

# Run when all upstream tasks have succeeded or been skipped
flexible_task = DummyOperator(
    task_id='flexible_task',
    trigger_rule='none_failed',
)
```

### Dependency Best Practices

1. **Keep DAGs Readable**

   - Group related dependencies together in your code
   - Use comments to explain complex dependency structures
   - Consider using task groups for complex DAGs

2. **Avoid Circular Dependencies**

   - Ensure your DAG remains acyclic
   - Airflow will detect cycles at parse time

3. **Be Mindful of Parallelism**

   - Design dependencies to maximize parallelism where appropriate
   - Consider resource constraints when allowing parallel execution

4. **Create Logical Checkpoints**

   - Use dummy operators as join points for clarity
   - This helps visualize the workflow stages in the UI

5. **Use Consistent Dependency Style**
   - Stick to bitshift operators when possible for readability
   - Use consistent indentation for dependency definitions

### Example: Complete DAG with Dependencies

Here's an example of a complete DAG with various dependency patterns:

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.dummy import DummyOperator
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.utils.task_group import TaskGroup

# Define default arguments
default_args = {
    'owner': 'data_team',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'email_on_failure': True,
    'email': ['alerts@example.com'],
}

# Create the DAG
with DAG(
    'complex_data_workflow',
    default_args=default_args,
    description='Example of complex dependency patterns',
    schedule_interval='@daily',
    start_date=datetime(2023, 1, 1),
    catchup=False,
    tags=['example', 'dependencies'],
) as dag:

    # Start task
    start = DummyOperator(task_id='start')

    # Task group for data extraction
    with TaskGroup(group_id='extract') as extract_group:
        extract_customers = DummyOperator(task_id='extract_customers')
        extract_orders = DummyOperator(task_id='extract_orders')
        extract_products = DummyOperator(task_id='extract_products')

    # Data validation with branching
    def validate_data(**context):
        # Dummy logic for example
        if context['execution_date'].day % 2 == 0:
            return 'validation_success'
        else:
            return 'validation_failed'

    validate = BranchPythonOperator(
        task_id='validate_data',
        python_callable=validate_data,
    )

    validation_success = DummyOperator(task_id='validation_success')
    validation_failed = DummyOperator(task_id='validation_failed')

    # Task group for transformation (only runs after successful validation)
    with TaskGroup(group_id='transform') as transform_group:
        transform_customers = DummyOperator(task_id='transform_customers')
        transform_orders = DummyOperator(task_id='transform_orders')
        transform_products = DummyOperator(task_id='transform_products')

        # Set dependencies within the transform group
        [transform_customers, transform_orders] >> transform_products

    # Load task (runs after transformation)
    load_data = DummyOperator(task_id='load_data')

    # Cleanup and notification tasks (run regardless of success/failure)
    cleanup = DummyOperator(
        task_id='cleanup',
        trigger_rule='all_done',  # Run regardless of upstream success/failure
    )

    notify = DummyOperator(task_id='send_notification')

    # End task
    end = DummyOperator(task_id='end')

    # Define the overall workflow dependencies
    start >> extract_group >> validate >> [validation_success, validation_failed]
    validation_success >> transform_group >> load_data
    validation_failed >> cleanup
    load_data >> cleanup >> notify >> end
```

This example demonstrates multiple dependency patterns including linear dependencies, fan-out, fan-in, conditional branching, and task groups, all working together in a single DAG.
