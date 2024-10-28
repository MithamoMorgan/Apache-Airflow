![](https://github.com/MithamoMorgan/Apache-Airflow/blob/master/AirflowLogo.png)

## What is Apache Airflow

It is an open source platform designed to programmatically author, schedule and monitor workflows. It allows users to define workflows as Directed Acyclic Graphs (DAGs) using python code, enabling complex data pipelines to be managed easily.

## Run Airflow in Python Env

Create a python environment using the code:
```
python -m venv folder_name
```

Activate the virtual environment using the code:
```
py_env\Scripts\activate.ps1
```
Incase you get an error: cannot be loaded because running scripts is disabled on this system, open power shell and run this code:
```
 Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```
then activate the virtual env using the code above. Once activated it will look like,

![](https://github.com/MithamoMorgan/Apache-Airflow/blob/master/venv%20activation.jpg)

To set the `AIRFLOW-HOME` variable to current directory, use the following syntax:
```
$env:AIRFLOW_HOME = "."
```
