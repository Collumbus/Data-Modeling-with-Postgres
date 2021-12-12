# Data-Modeling-with-Postgres
This project arose from the need for a startup (a fictitious company) to analyze the data collected from music and user activity in its new music streaming app. 

The company's data is stored in JSON files, which makes it hard to analyze by queries. So, a data modelling composed of a Postgres database and an ETL pipeline by Python was proposed, which will allow the analytics team to carry out their analysis. 

## Datasets
In the Song Dataset each file is in JSON format and contains metadata about a song and the artist of that song. The files are partitioned by the first three letters of each song's track ID. For example, here are filepaths to two files in this dataset.

```
song_data/A/B/C/TRABCEI128F424C983.json
song_data/A/A/B/TRAABJL12903CDCF1A.json
```

And below is an example of what a single song file, TRAABJL12903CDCF1A.json, looks like.
```
{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0
```

The Log Dataset consists of log files in JSON format generated by this event simulator based on the songs in the dataset above. These simulate activity logs from a music streaming app based on specified configurations.

The log files in the dataset you'll be working with are partitioned by year and month. For example, here are filepaths to two files in this dataset.

```
log_data/2018/11/2018-11-12-events.json
log_data/2018/11/2018-11-13-events.json
```

We can take a look at one of the log files visually using a pandas dataframe to read the data:
```
df = pd.read_json(filepath, lines=True)
df = pd.read_json('data/log_data/2018/11/2018-11-01-events.json', lines=True)
df.head()
```
This will allow us to view, as in the image below, the data from the 2018-11-01-event.json file.

![log_sample](/img/log_data.png)




## Project Structure

```
Data Modeling with Postgres
|____data			        # Datasets (json files)
| |____log_data                         # Log Dataset 
| | |____...
| |____song_data                        # Song Dataset 
| | |____...
|
|____jupyter_notebooks			# Notebooks for developing and testing ETL
| |____etl.ipynb    	    	        # Developing ETL builder
| |____test.ipynb	    	        # Testing ETL builder
|
|____scripts        			# python codes
| |____etl.py		    	        # ETL builder
| |____sql_queries.py		        # ETL query helper functions
| |____create_tables.py		        # Database/table creation script
```


## ETL Pipeline
### etl.py
ETL pipeline builder

1. `process_data`
	* Iterating dataset to apply `process_song_file` and `process_log_file` functions
2. `process_song_file`
	* Process song dataset to insert record into _songs_ and _artists_ dimension table
3. `process_log_file`
	* Process log file to insert record into _time_ and _users_ dimensio table and _songplays_ fact table

### create_tables.py
Creating Fact and Dimension table schema

1. `create_database`
2. `drop_tables`
3. `create_tables`

### sql_queries.py
Helper SQL query statements for `etl.py` and `create_tables.py`

1. `*_table_drop`
2. `*_table_create`
3. `*_table_insert`
4. `song_select`


## Database Schema
![Entity Relationship Diagram - ERD](/img/sparkifydb_erd.png)

### Fact table
```
songplays
	- songplay_id 	Primary key created serially
	- start_time 	The foreign key related to the time table
	- user_id	The foreign key related to the users table
	- level
	- song_id 	The foreign key related to the songs table
	- artist_id 	The foreign key related to the artists table
	- session_id
	- location
	- user_agent
```

### Dimension table
```
users
	- user_id 	Primary key created serially
	- first_name
	- last_name
	- gender
	- level

songs
	- song_id 	Primary key created serially
	- title
	- artist_id     Foreign key related to the artists table
	- year
	- duration

artists
	- artist_id 	Primary key created serially
	- name
	- location
	- latitude
	- longitude

time
	- start_time 	Primary key created serially
	- hour
	- day
	- week
	- month
	- year
	- weekday
```
## Implementing the project...
Initially, we have to create database and tables in postgres:
* For this, the creation and drop statements of each table were written in the [sql_queries.py](https://github.com/Collumbus/Data-Modeling-with-Postgres/blob/main/scripts/sql_queries.py) file. Drop statements are needed so that we can delete everything and run the project more than once (for test purpose).

* Next, we have to run the [create_tables.py](https://github.com/KentHsu/Udacity-Data-Engineering-Nanodgree/blob/main/Data%20Modeling%20with%20Postgres/src/create_tables.py) file to create your database and tables.
To run the file, just go to the project's root folder in the terminal and enter the command ```python scripts\create_tables.py ```.

Once that's done, we need to create the ETL pipeline:
*  For this, the entire ETL pipeline process was written in the [etl.py](https://github.com/Collumbus/Data-Modeling-with-Postgres/blob/main/scripts/etl.py) file.
*  Like the previous file, we need to run the ```etl.py``` file through the terminal running the ```python scripts\etl.py ``` command.

P.S.:During the ETL pipeline development process an MVP was first created that can be checked in the [etl.ipynb](https://github.com/Collumbus/Data-Modeling-with-Postgres/blob/main/jupyter_notebooks/etl.ipynb) notebook jupyter. Before running ```etl.ipynb``` or running the [test.ipynb](https://github.com/Collumbus/Data-Modeling-with-Postgres/blob/main/jupyter_notebooks/test.ipynb) tests the ```create_tables.py``` file must be run.

### Example of query and results for song play analysis

One of the analysts wrote a query to analyze the top 10 hours of the day with the highest number of free users using the platform to prepare a price list for ads.

### Query
```
SELECT 
        COUNT(songplay_id) count_users,
        t.hour 
FROM songplays sp
JOIN time t ON sp.start_time = t.start_time
WHERE sp.level = 'free'
GROUP BY 2
ORDER BY 1 desc
lIMIT 10
```

### Results
```
  | count_users	| Hour
-----------------------
0 |	118     | 15
1 |	111     | 14
2 |	100     | 16
3 |	81      | 13
4 |	74      | 18
5 |	63      | 10
6 |	62      | 17
7 |	57      | 21
8 |	57      | 12
9 |	48      | 4
```
**P.S.: We can run this query directly in pgAdmin's query editor for example.**