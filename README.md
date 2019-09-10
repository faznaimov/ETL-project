# ETL Project


## Extract : 

* Data sets used : ticketmaster API and MTV 10,000 artists info csv file.

1. Extracting ticketmaster API : Wrote code to get data on all upcoming concert in NY state. Refer to [Jupyter Notebook](ticketmaster_extract.ipynb)

2. MTV 10,000 artist.csv : source [data world]

## Transform:

1. Ticketmaster data set clean up
* Ticketmaster API was extracted to a CSV file.
* Artist names were either none existence or multiple separated by a comma. First, dropped all NA artist names and then separated artists column by comma and only took the first column then replaced in as Artists column on DF.
``` python
#droping all the artists are 'nan'
to_drop = ['nan']
concert_df = concert_df[~concert_df['artists'].isin(to_drop)]

#separating multiple artists by comma
df = concert_df["artists"].str.split(",", n = 1, expand = True)
concert_df['artists']=df[0] #taking the first column for the artists name
concert_df['artists']=concert_df['artists'].str.strip("'").astype(str) #one more clean up 
```
* Cleaned up excessive quotation marks and brackets from the original API data.
``` python
concert_df['artists']=concert_df['artists'].str.strip('[]').astype(str)
concert_df['artists']=concert_df['artists'].str.strip("'").astype(str)
concert_df['artists']=concert_df['artists'].str.strip('"').astype(str)
```
[Ticketmaster data Transform Jupyter Notebook](ticketmaster_sqltrans.ipynb)
  
2. Mtv data set clean up
* Dropped duplicate artists then dropped social media links
``` python
df = df.drop_duplicates('name', keep = 'first', inplace=False)
df = df.drop(['facebook','twitter'], axis = 1)
```
* Rearranged dataset
``` python
df = df[['name','genre','website','mtv']]
```
* Renamed Mtv column to bio_link
``` python
df = df.rename(columns={'mtv':'bio_link'})
```
[MTV data Transform Jupyter Notebook](mtv_data_ETL.ipynb)

## Load: 
* By using sqlAlchemy create_engine, uploaded data frame to Postgres. And we were able to confirm two tables [‘ticketmaster’,’mtv_data’]

*The reason why we chose these two data sets and decided to make the tables is that we wanted to see if any notable music artists have any upcoming concerts in New York State area. 




