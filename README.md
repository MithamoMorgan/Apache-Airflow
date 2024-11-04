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

* Note:

| Preset    | Meaning                                                             |
|-----------|---------------------------------------------------------------------|
| None      | Don’t schedule, use for exclusively “externally triggered” DAGs     |
| @once     | Schedule once and only once                                          |
| @hourly   | Run once an hour at the beginning of the hour                       |
| @daily    | Run once a day at midnight                                          |
| @weekly   | Run once a week at midnight on Sunday morning                       |
| @monthly  | Run once a month at midnight of the first day of the month         |
| @yearly   | Run once a year at midnight of January 1                            |

## Example Code: Scraping Jumia Flash sales data:

```python
import os
import pandas as pd
from bs4 import BeautifulSoup
import requests
from urllib.parse import urljoin
from airflow import DAG
from datetime import datetime, timedelta
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner':'devscraper',
    'retries':2,
    'retry_delay':timedelta(minutes = 1)
}

def scrape():

    # Create directory if it doesn't exist
    dir_path = '/opt/airflow/airflow_data'
    os.makedirs(dir_path, exist_ok=True)

    url = 'https://www.jumia.co.ke/flash-sales/' 
    details_list = []
    
    while True:
        response = requests.get(url)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        details = soup.find_all('div', class_ = 'info')
        footers  = soup.find_all('footer', class_ = 'ft')
    
        for detail,footer in zip(details,footers):
            name = detail.find('h3', class_ = 'name').text.strip()
            price =  detail.find('div', class_ = 'prc').text.strip()
            try:
                old_price = detail.find('div', class_ = 'old').text.strip()
            except:
                old_price = " "
            try:
                discount = detail.find('div', class_ = 'bdg _dsct _sm').text.strip()
            except:
                discount = " "
            try:
                stars_reviews = detail.find('div', class_ = 'rev').text.strip()
            except:
                stars_reviews = " "
        
            try:
                items_left = footer.find('div', class_ = 'stk').text.strip()
            except:
                items_left = " Out of stock"
                
                
        
            details_dict = {'Name':name,'price':price,'Old Price':old_price,'discount':discount,'Stars and Reviews':stars_reviews,'Items Left':items_left}
            details_list.append(details_dict)
    
        next_page = soup.find('a',{'class':'pg', 'aria-label':'Next Page'})
        if next_page:
            next_url = next_page.get('href')
            url = urljoin(url,next_url)
        else:
            break
    
    flash_sale_data = pd.DataFrame(details_list)
    # save in a csv file
    flash_sale_data.to_csv(os.path.join(dir_path, 'jumia_df.csv'), index=False)


with DAG(
    dag_id = 'jumia_scraping_dag',
    default_args = default_args,
    description = 'Writing my first dag to scrape data',
    start_date = datetime(2024,10,31, 11),
    schedule_interval = '@hourly'
) as dag :

    scrape_task = PythonOperator(
        task_id = 'scrape',
        python_callable = scrape
    )

scrape_task
```

## Code Explanation:

### 1. `dir_path = '/opt/airflow/airflow_data/csv'`:

`/`: This is the root directory in a Unix-like file system.

`opt`: This directory is typically used for optional software packages. It's often where third-party applications are installed.

`airflow`: This refers to Apache Airflow, a platform to programmatically author, schedule, and monitor workflows. The presence of this directory suggests that Airflow is installed on this system.

`airflow_data`: This subdirectory is designated for data specific to Airflow.

`csv`: This is a subdirectory within airflow_data, which stores CSV files.

### 2. Create a directory if it does not exist:

```python
    os.makedirs(dir_path, exist_ok=True)
```
The code above creates a folder inside the filesystem of the Docker container where the command is executed.

`os`: This is the operating system module in Python that provides a way to interact with the file system.

`makedirs`: This function creates the specified directory path. If any intermediate directories in the path do not exist, it will create them as well.

`dir_path`: This variable contain the path to the directory to be created

`exist_ok=True`: This argument tells makedirs to not raise an error if the target directory already exists. If exist_ok were set to False (the default), the function would raise a FileExistsError if the directory already exists.

### 3. `os.path.join(dir_path, 'jumia_df.csv')`:

`os.path.join `: used to create a file path that is compatible with the operating system you're working on (it handles the appropriate path separators ie `/`).

`dir_path`: the directory where I want to save the CSV file.

`'jumia_df.csv'`: is the name of the file I want to create.

`index=False`: This argument tells the to_csv method not to write row indices to the CSV file.

The result is a complete file path, like `/opt/airflow/airflow_data/csv/jumia_df.csv.`

## How to check if the csv file exist

1. **Get into the Airflow Worker Container:** You need to access the specific container where the scraping task ran. You can do this with the following command:
   
   `docker exec -it airflow_materials-airflow-worker-1 /bin/bash
`
2. **Navigate to the Directory:** Once you're inside the container, navigate to the directory where the CSV file is supposed to be saved:
   
    `cd /opt/airflow/airflow_data/csv
`
3. **Check for the File:** List the files in that directory to see if your CSV file is there:

   `ls`

4. **Copy the File:** In the new terminal window, use the docker cp command to copy the CSV file from the container to your local machine:

   `docker cp airflow_materials-airflow-worker-1:/opt/airflow/airflow_data/csv/jumia_df.csv C:\Users\User\your_directory_choice`

*NB*: The command copies the file  `jumia_df.csv` from the specified path inside the Docker container `/opt/airflow/airflow_data/csv/jumia_df.csv` to the local directory `C:\Users\User\your_directory_choice`

### Note:

* When Airflow is running inside a Docker container, it does not have direct access to the host machine's filesystem unless that filesystem is explicitly shared with the container. So one need to mount the host directory to the container's directory. Here is an example:

```
C:\Users\User\Desktop\AirflowData\csv:/opt/airflow/airflow_data/csv
```

* Where `C:\Users\User\Desktop\AirflowData\csv` is the host directory and `/opt/airflow/airflow_data/csv` is the container's directory.

* Add the line above in the yaml file under `volumes` in the `x-airflow-common` section and it should look something similar to this:

```yaml
volumes:
    - ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags
    - ${AIRFLOW_PROJ_DIR:-.}/logs:/opt/airflow/logs
    - ${AIRFLOW_PROJ_DIR:-.}/config:/opt/airflow/config
    - ${AIRFLOW_PROJ_DIR:-.}/plugins:/opt/airflow/plugins
    - C:/Users/User/Desktop/AirflowData/csv:/opt/airflow/airflow_data/csv
```

### Example of code with 2 tasks to scrape and clean

```python
import os
import pandas as pd
from bs4 import BeautifulSoup
import requests
from urllib.parse import urljoin
from airflow import DAG
from datetime import datetime, timedelta
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner':'devscraper',
    'retries':2,
    'retry_delay':timedelta(minutes = 1)
}

def scrape():

    # Create directory if it doesn't exist
    dir_path = '/opt/airflow/airflow_data/csv'
    os.makedirs(dir_path, exist_ok=True)

    url = 'https://www.jumia.co.ke/flash-sales/' 
    details_list = []
    
    while True:
        response = requests.get(url)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        details = soup.find_all('div', class_ = 'info')
        footers  = soup.find_all('footer', class_ = 'ft')
    
        for detail,footer in zip(details,footers):
            name = detail.find('h3', class_ = 'name').text.strip()
            price =  detail.find('div', class_ = 'prc').text.strip()
            try:
                old_price = detail.find('div', class_ = 'old').text.strip()
            except:
                old_price = " "
            try:
                discount = detail.find('div', class_ = 'bdg _dsct _sm').text.strip()
            except:
                discount = " "
            try:
                stars_reviews = detail.find('div', class_ = 'rev').text.strip()
            except:
                stars_reviews = " "
        
            try:
                items_left = footer.find('div', class_ = 'stk').text.strip()
            except:
                items_left = " Out of stock"
                
                
        
            details_dict = {'Name':name,'price':price,'Old Price':old_price,'discount':discount,'Stars and Reviews':stars_reviews,'Items Left':items_left}
            details_list.append(details_dict)
    
        next_page = soup.find('a',{'class':'pg', 'aria-label':'Next Page'})
        if next_page:
            next_url = next_page.get('href')
            url = urljoin(url,next_url)
        else:
            break
    
    flash_sale_data = pd.DataFrame(details_list)
    # save in a csv file
    flash_sale_data.to_csv(os.path.join(dir_path, 'jumia_df.csv'), index=False)

def clean_data():
    dir_path = '/opt/airflow/airflow_data/csv'
    flash_sale_data = pd.read_csv(os.path.join(dir_path, 'jumia_df.csv'))

    # Change numeric columns to numeric types
    flash_sale_data['price'] = flash_sale_data['price'].str.replace(r'[^\d.]', '', regex=True).astype('float')
    flash_sale_data['Old Price'] = flash_sale_data['Old Price'].str.replace(r'[^\d.]', '', regex=True)
    flash_sale_data['Old Price'] = pd.to_numeric(flash_sale_data['Old Price'], errors='coerce')
    flash_sale_data['discount'] = flash_sale_data['discount'].str.replace(r'[^\d.]', '', regex=True)
    flash_sale_data['discount'] = pd.to_numeric(flash_sale_data['discount'], errors='coerce')
    flash_sale_data['Items Left'] = flash_sale_data['Items Left'].str.replace(r'[^\d.]', '', regex=True)
    flash_sale_data['Items Left'] = pd.to_numeric(flash_sale_data['Items Left'], errors = 'coerce')


    # Extract Stars and Reviews from Stars and Reviews column
    flash_sale_data[['Stars', 'Reviews']] = flash_sale_data['Stars and Reviews'].str.extract(r'([\d.]+) out of 5\((\d+)\)')
    flash_sale_data['Stars'] = pd.to_numeric(flash_sale_data['Stars'], errors='coerce')
    flash_sale_data['Reviews'] = pd.to_numeric(flash_sale_data['Reviews'], errors='coerce')
    
    # Drop the column Stars and Reviews
    flash_sale_data = flash_sale_data.drop('Stars and Reviews', axis=1)

    # Save the cleaned data back to CSV
    flash_sale_data.to_csv(os.path.join(dir_path, 'cleaned_jumia_df.csv'), index=False)


with DAG(
    dag_id = 'jumia_scraping_dag',
    default_args = default_args,
    description = 'Writing my first dag to scrape data',
    start_date = datetime(2024,10,31, 11),
    schedule_interval = '@daily'
) as dag :

    scrape_task = PythonOperator(
        task_id = 'scrape',
        python_callable = scrape
    )

    clean_task = PythonOperator(
    task_id='clean_data',
    python_callable=clean_data
)

scrape_task >> clean_task
```
