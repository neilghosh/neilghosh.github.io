Last few months I has been working on a demo project where I fetch data Stock Data in CSV (zipped) format periodically and store tham in Google Cloud. 
This is a a good end to end use case where data can also be transferred between storage services automatically using serverless compute like 
Google Cloud Functions in a event driven manner Cloud Pub/Sub. Following is the high level architecture the system.

![NSE DATA](https://user-images.githubusercontent.com/726337/115959531-4c2a9280-a52a-11eb-8e55-536e9310103b.png)

### Periodic Event
Google Cloud Scheduler is used like a distributed cron job to invoke an endpoint periodically. Since stock data is updated everyday (we are not talking about 
intra-day numbers ), it was good enough to run the job everyday at midnight 00:00 IST when we know for sure market is closed and the day's data is already available 
in stock exchange's site for all the shares.

<img width="877" alt="image" src="https://user-images.githubusercontent.com/726337/115959960-7c733080-a52c-11eb-9af5-2a9533afe962.png">

### Download Data and Dump In Storage
A Google Cloud Function with [HTTP trigger](https://cloud.google.com/functions/docs/concepts/events-triggers) ( runtime NodeJS ) is used to make a outbound network call to Stock Exchange and download the data. Since the data is fetched in ZIP format, 
given the small size of daily data its extracted to the original CSV file and put into a bucket with an unique name which includes the date. Had it been a large file
one could do the unzipping process asynchronously in another Function.

<img width="870" alt="image" src="https://user-images.githubusercontent.com/726337/115959974-97de3b80-a52c-11eb-9db2-533ebb860323.png">


### Move Data From File To BigQuery
The Google Cloud Storage Bucket where the file was dumped earlier had a trigger which calls another Cloud Function which does a streaming insert to BigQuery and publishes 
a Cloud Pub/Sub event to a Topic. Due to analytical nature of bigquery its very convenient to run ad hock query to get insights or even plot the data using 
[Google Data Studio](https://datastudio.google.com/). 

<img width="858" alt="Screenshot 2021-04-24 at 5 44 39 PM" src="https://user-images.githubusercontent.com/726337/115959511-2d2c0080-a52a-11eb-980a-668910813fc4.png">

### Move Clean Data To Cloud Datastore
While we have all the data in BigQuery we only need certain attributes and sometimes rows are duplicated as corrections may happen and we re-download the data. Also 
We need a database from where we can programmatically render data in UI. RDBMS like Cloud SQL is probably the first choice, here only the date and the ticker symbol 
matters to get hold of any record. Hence If they key is chosen optimally (e.g. SYMBOL_TIMESTAMP), we could leverage distributed NoSQL database like Cloud Datastore
for serving our APIs or UIs. So we subscribe to the topic where we publish data and trigger another cloud function to UPSERT data to datastore. Hence if correction or 
duplicate quote record comes it can update the existing data.

<img width="1412" alt="image" src="https://user-images.githubusercontent.com/726337/115959753-60bb5a80-a52b-11eb-9223-b00a91314b35.png">

### Fetch from UI and Charts
Finally the data can be shown to end user by calling the datastore APIs directly from JavaScript on the UI and Google Chart library can be used to show it in a
tabular or Candlestick chart. To authenticate the user Google Cloud IAM is used to grant permission on the Google Cloud Datatore. Alternatively a REST service 
could have been written to serve the same data instead of directly granting access to Datastore via oAuth.

<img width="1043" alt="image" src="https://user-images.githubusercontent.com/726337/115959801-a5df8c80-a52b-11eb-9fa3-623228eb9486.png">

Following are the source code for reference.
1. [GCloud Data Echange](https://github.com/neilghosh/gcloud-data-exchange)
2. [NSE Historical Data](https://github.com/neilghosh/nse-historical-data)

<img width="1518" alt="image" src="https://user-images.githubusercontent.com/726337/115959853-e50ddd80-a52b-11eb-8a5e-8c036bb0e290.png">

