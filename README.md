# ETL Project

Our final goal is to have a dataset of all upcoming concerts in NY state area and at the same time user will be able to join MTV artist data and have more information about artists and concerts. 

## Extract : 

Datasets used : ticketmaster API and MTV 10,000 artists info.

1. Extracting ticketmaster API : Pulled data on all upcoming concert in NY state.

``` python
url = "https://app.ticketmaster.com/discovery/v2/events.json?"
segment = "music"
start = ["2019-07-19T00:01:00Z","2019-08-15T00:01:00Z","2019-09-23T00:01:00Z","2019-11-17T00:01:00Z"]
end = ["2019-08-15T00:00:00Z","2019-09-23T00:00:00Z","2019-11-17T00:00:00Z",""]
us = "US"
state = "NY"
#max limit of data per page
size=100

#list to append the data
data = []

for start,end in zip(start,end):

    #final url
    m_url = f"{url}classificationName={segment}&startDateTime={start}&endDateTime={end}&countryCode{us}&stateCode={state}&size={size}&apikey={api_key}"
    
    for page in range(10):
        query_url = f"{m_url}&page={page}"
        print(query_url)
    
        try:
            results = requests.get(query_url).json()['_embedded']['events']
        except:
            pass
        
        #loop thru result to get the data
        for item in results:
            try:
                concert_name = item['name']
            except:
                concert_name = "N/A"
            #gets all the artists in the concert
            try:
                list_lenth = len(item['_embedded']['attractions'])
                artists = []
                for i in range(list_lenth):
                    try:
                        artists.append(item['_embedded']['attractions'][i]['name'])
                    except:
                        artists.append("N/A")
            except:
                artists = "N/A"
            try:
                date = item['dates']['start']['localDate']
            except:
                date = "N/A"
            try:
                time = item['dates']['start']['localTime']
            except:
                time = "N/A"
            try:
                city = item['_embedded']['venues'][0]['city']['name']
            except:
                city = "N/A"
            try:
                state = item['_embedded']['venues'][0]['state']['stateCode']
            except:
                state = "N/A"
            try:
                p_min = item['priceRanges'][0]['min']
            except:
                p_min = "N/A"
            try:
                p_max = item['priceRanges'][0]['min']
            except:
                p_max = "N/A"
        
            data.append([concert_name,
                         artists,
                         date,
                         time,
                         city,
                         state,
                         p_min,
                         p_max])

print("Data collection has been successfully completed")

#creating dataframe
concerts = pd.DataFrame(data)
concerts.columns = ['concert_name',
                    'artists',
                    'date',
                    'time',
                    'city',
                    'state',
                    'price_min',
                    'price_max']

#saving as scv 
concerts.to_csv('output/concerts.csv',index = False)
```

[Ticketmaster Data Extraction Jupyter Notebook](ticketmaster_extract.ipynb)

2. MTV 10,000 artist.csv : [Source](https://gist.github.com/mbejda/9912f7a366c62c1f296c)

## Transform:

### Ticketmaster data set clean up
* Artists colum had N/A and lists separated by a comma. First, dropped all NA artist names and then separated artists column by comma and only took the first column then replaced in as Artists column on DF. (Future enhancement get all the artists in the list)

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
  
### MTV data set clean up
* Dropped duplicate artists and social media links

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
Uploaded data frame to Postgres by using sqlAlchemy.
