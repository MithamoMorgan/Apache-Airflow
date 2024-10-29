![](https://github.com/MithamoMorgan/Apache-Airflow/blob/master/AirflowLogo.png)

## What is Apache Airflow

It is an open source platform designed to programmatically author, schedule and monitor workflows. It allows users to define workflows as Directed Acyclic Graphs (DAGs) using python code, enabling complex data pipelines to be managed easily.

## Resources:

Get everything you need to kick start your Airflow journey [here](https://www.datacamp.com/tutorial/getting-started-with-apache-airflow)

## Fixing the error: Missing AIRFLOW_UID Variable
[Link](https://ourtechroom.com/tech/fix-airflow-error-missing-airflow-uid-variable/)

## Running Airflow with docker

Navigate to Your Project Directory (where your docker-compose.yml is located):

* **Start Airflow:** Run the following command: `docker-compose up`
  
* `docker-compose down`: Stops and removes containers and networks but keeps volumes. Use when you want to stop and clean up your containers but keep the data in the volumes for later use

* `docker-compose down -v`: Stops and removes containers, networks, and also deletes volumes, losing all stored data. Use when you want to completely clean up your Docker environment, including any persistent data stored in volumes. This is useful when you want to start fresh or when you're troubleshooting issues and want to ensure no leftover data affects the new setup.

## Note:
The default link for the Apache Airflow UI is:
`http://localhost:8080
`


