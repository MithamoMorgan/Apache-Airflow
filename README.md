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

## Steps to access the csv file from my scraping dag

1. **Get into the Airflow Worker Container:** You need to access the specific container where the scraping task ran. You can do this with the following command:
   
   `docker exec -it airflow_materials-airflow-worker-1 /bin/bash
`
2. **Navigate to the Directory:** Once you're inside the container, navigate to the directory where the CSV file is supposed to be saved:
   
    `cd /opt/airflow/airflow_data
`
3. **Check for the File:** List the files in that directory to see if your CSV file is there:

   `ls`

4. **Copy the File:** In the new terminal window, use the docker cp command to copy the CSV file from the container to your local machine:

   `docker cp airflow_materials-airflow-worker-1:/opt/airflow/airflow_data/car_df.csv C:\Users\User\Airflow_Materials
`



