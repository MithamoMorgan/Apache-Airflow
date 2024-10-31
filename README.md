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

**NB:** For `docker-compose up -d` to work, Docker Desktop must be running. Docker Desktop provides the Docker Engine, which is necessary to manage and run your containers. If it's not open, you'll need to start it before running your command.

## Note:
The default link for the Apache Airflow UI is:
`http://localhost:8080
`

## Example

### Using Apache Airflow to Build a Pipeline for Scraped Data:

Here is the [link](https://oxylabs.io/blog/building-scraping-pipeline-apache-airflow?utm_source=youtube&utm_medium=organic_video&utm_content=Building%20Scraping%20Pipelines%20With%20Apache%20Airflow) to the website

## DAGs Runs

* Note that if you are running a DAG on schedule_interval of one day, the run stamped 2024-10-30 will be triggered as soon after 2024-10-30T23:59. In other words, the job instance is started once the period it covers has ended.

* The scheduler runs your job one schedule interval after the start date, at the end of the period.
