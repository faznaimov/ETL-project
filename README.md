# ETL Project


## Extract : 

* Data sets used : ticketmaster API and MTV 10,000 artists info csv file.

(1) Extracting ticketmaster API : Wrote code to get data on all upcoming concert in NY state. Refer to "ticketmaster_extract.ipynb"

(2) MTV 10,000 artist.csv : source [data world]



## Transform:

(1) Ticketmaster data set clean up
* Ticketmaster API was extracted to a CSV file.
* Artist names were either none existence or multiple separated by a comma. First, dropped all NA artist names and 
* then separated artists column by comma and only took the first column then replaced in as Artists column on DF.
* Cleaned up excessive quotation marks and brackets from the original API data.


  
(2) Mtv data set clean up
* Dropped duplicate artists then dropped social media links
* Rearranged dataset
* Renamed Mtv column to bio_link



## Load: 
* By using sqlAlchemy create_engine, uploaded data frame to Postgres. And we were able to confirm two tables [‘ticketmaster’,’mtv_data’]

*The reason why we chose these two data sets and decided to make the tables is that we wanted to see if any notable music artists have any upcoming concerts in New York State area. 




