# Data-Modeling-with-Postgres
This project arose from the need for a startup (a fictitious company) to analyze the data collected from music and user activity in its new music streaming app. 

The company's data is stored in JSON files which makes it difficult to analyze by queries. So, a data modelling composed of a Postgres database and an ETL pipeline by Python was proposed, which will allow the analytics team to carry out their analysis. 


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

###Results
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