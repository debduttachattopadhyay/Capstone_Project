Scope of the Project : 

Clearly state the rationale for the choice of tools and technologies for the project.:

The goal of this project is to ensure the right data is gathered for a successful analysis by the team. 
Data Engineering tools like Python - pandas, Amazon Redshift, Spark, and Airflow will be used for displaying, storing and manupilation of data.
It also displays the datatypes of each of the columns of each dataset. 
This exploration is done with the Pandas library. Pandas, because of it's ease of use, and ability to load datasets of different formats (csv, parquet, json) 
into dataframes.

COMPLETE PROJECT WRITE UP:

Data Model
There should be two taging table. Also, there should be two fact tables and 6 dimentional tables. This schema is most suitable for this project due to the fact that two different sets of data are used to achieve this result. Also, it helps minimalize redundancy, and improve data accuracy. Each dimension table is connected to its fact table, except the US_STATES table - connected to the US_Cities dimension table. Although, this requires a little more space taken for a table in the database, and one more join, it still allows for accurate data extractions. Below are each tables schema in the database.

STAGING_IMMIGRATIONS (staging table) as loaded from S3
|-- immigrant_id: (bigint) A unique, spark-generated id representing each immigrant
|-- year: (integer) year the data was captured
|-- month: (integer) month the data was captured
|-- resident_country_code: (integer) The resident country code of the immigrant.
|-- arrival_date: (date) Date of immigrants arrival
|-- address: (varchar) Current US address of the immigrant (state code)
|-- departure_date: (date) Date of immigrants departure (if available)
|-- age: (integer) age of immigrant (as at when this data was captured)
|-- visa_code: (integer) specifies the type of visa used. Equivalent to Business, Pleasure or Student.
|-- birth_year: (integer) Immigrants year of birth
|-- gender: (varchar) Immigrants gender
|-- airline: (varchar) The airline used by immigrant
|-- mode: (varchar) mode of transport (code) equivalent to LAND, SEA, AIR. If available.
|-- resident_country: (varchar) The resident country of the immigrant
|-- visa_type: (varchar) specifies the type of visa used
|-- state_address: (varchar) current US state address (unabbreviated).
|-- transport_mode: (varchar) mode of transport (code derived).

STAGING_CITIES (staging table) as loaded from S3
|-- city_id: (bigint) unique, spark generated city identifier
|-- city: (varchar) US city name
|-- state: (varchar) US state name
|-- median_age: (real) median age of residents in this city
|-- male_population: (integer) male population of residents in the corresponding city
|-- female_population: (integer) female population of residents in the corresponding city
|-- total_population: (integer) total population of residents in the corresponding city
|-- num_of_veterans: (integer) total number of veterans in each corresponding city
|-- no_of_immigrants: (integer) total number of immigrants in each corresponding city
|-- avg_household_size: (real) average household size of city residents
|-- state_code: (varchar) US state code
|-- race: (varchar) most dominant rance in each city

US_GEOGRAPHY (fact table)
|-- city_id: (bigint) unique, spark generated city identifier
|-- male_population: (integer) male population of residents in the corresponding city
|-- female_population: (integer) female population of residents in the corresponding city
|-- total_population: (integer) total population of residents in the corresponding city
|-- num_of_veterans: (integer) total number of veterans in each corresponding city
|-- no_of_immigrants: (integer) total number of immigrants in each corresponding city
|-- avg_household_size: (real) average household size of city residents

IMMIGRANTS_FACTS (fact table)
|-- immigrant_id: (bigint) A unique, spark-generated id representing each immigrant
|-- year: (integer) year the data was captured
|-- month: (integer) month the data was captured
|-- visa_code: (integer) specifies the type of visa used. Equivalent to Business, Pleasure or Student.
|-- mode: (varchar) mode of transport (code) equivalent to LAND, SEA, AIR. If available.

US_CITIES (dim table)
|-- city_id: (bigint) unique, spark generated city identifier
|-- city: (varchar) US city name
|-- state_code: (varchar) US state code

US_STATES (dim table)
|-- state: (varchar) US state name
|-- state_code: (varchar) US state code

VISA_TYPES (dim table)
|-- visa_code: (integer) specifies the type of visa used. Equivalent to Business, Pleasure or Student.
|-- visa_type: (varchar) specifies the type of visa used

TRANSPORT_MODES (dim table)
|-- mode: (varchar) mode of transport (code) equivalent to LAND, SEA, AIR. If available.
|-- transport_mode: (varchar) mode of transport (code derived).

TRAVEL_INFO (dim table)
|-- immigrant_id: (bigint) A unique, spark-generated id representing each immigrant
|-- arrival_date: (date) Date of immigrants arrival
|-- departure_date: (date) Date of immigrants departure (if available)
|-- airline: (varchar) The airline used by immigrant

IMMIGRANTS (dim table)
|-- immigrant_id: (bigint) A unique, spark-generated id representing each immigrant
|-- age: (integer) age of immigrant (as at when this data was captured)
|-- birth_year: (integer) Immigrants year of birth
|-- gender: (varchar) Immigrants gender
|-- resident_country: (varchar) The resident country of the immigrant
|-- address: (varchar) Current US address of the immigrant (state code)

DATA LOADING PROCESSES
Both datasets are compiled and staged first to an S3 bucket in parquet file format. This is done to have a Data Lake representation of both datasets/tables, 
and easy access to redshift. Two major tools are used in this case: Spark - for loading the data, and Amazon S3. Having these datasets stored in an S3 bucket 
allows an opportunity to use the Data Lake method.
Amazons Redshift tool is used as a warehouse to store these datasets in separate facts and dimensional tables. The S3 to Redshift workflow is managed 
Airflow which is DAG based Data Engineering workflow management system. Airflow is used in this case to ensure each of the above processes are carried 
out in the right order, and the right scheduled time, making the ETL process as seamless as possible.


Propose how often the data should be updated and why.
The data should be updated as per scheduled which will be daily refresh.

Write a description of how you would approach the problem differently under the following scenarios:

The data was increased by 100x. :
Spark will be the best possible tools to be used for the Data Exploration and Manipulation processes, as well as loading these datasets to S3. 
Apache Spark data processing is performed in memory, which means computations could be over 10 times faster than the MapReduce processing used in Hadoop. 
Also, Spark helps to automatically partition data based on the data size, and the number of cores in each node. If this data is 100 times increased, partitioning 
should be adopted. Spark evenly distributes data based on the number of available cores per node. Data partitioning can also be done on disk (or storage) 
using Sparks partitionBy() method. For example, storing the data in S3 by partitioning by year and month, gender or arrival date would go allow for effective data 
search and aggregations. Spark is an in memory data engine, it does not provide a storage system. Storage of data can be done using Amazon S3. 
Having Spark installed on Amazons EMR will also help manage processing resources (nodes and memory), but will save the organization compute costs, and use 
EMRFS as file storage.


The data populates a dashboard that must be updated on a daily basis by 7am every day. :
To achieve this daily 7am schedule, below is an example of what the DAG arguments would look like:
dag = DAG(
        dag_id="generated_dag_id",
        start_date=(2021,07,15,07,00,00)
        schedule_interval="@daily"
    )

The database needed to be accessed by 100+ people. :	
Amazon Redshift would do the storage work. Redshift is a data warehouse cloud-based system, storing data on a peta byte scale. 
Replication would be advisable especially in cases where the database users are in several geographical locations. Redshift automatically 
creates snapshots that tracks changes to the cluster. Redshift also offers different data replication strategies for its clusters.
