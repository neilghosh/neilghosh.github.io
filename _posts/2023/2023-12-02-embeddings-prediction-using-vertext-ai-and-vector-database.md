---
layout: post
title: "Generate Embeddings using Vertext AI and Vector Database"
description: "In this post, we will generate embeddings from photos stored in Google Cloud Storage and save them PostgreSQL database with [pg-vector enxtension](https://github.com/pgvector/pgvector). Then we will query for images which are similar to any given images on the data set. We will use a notebook in Collab Enterprise which is a great way of interactively playing with code snippets and iterating like a REPL. It is backed by computing resources (called Runtimes) from Google Cloud which can spin up and scale very quickly. Notebooks in the cloud also can be shared with others who can contribute text and code simultaneously.
"
---
Google Cloud Vertex AI's Multimodal Prediction Models let us create embeddings from Images (or even text for that matter). By default the embeddings are generated as 1408-dimension vectors however this value is configurable.

These embeddings can be stored in a vector database and various queries like finding similar images on the basis of distances can be done on the stored vector data.

![enter image description here](/assets/2023/plane-bird-distance.jpg)

In this post, we will generate embeddings from photos stored in Google Cloud Storage and save them PostgreSQL database with [pg-vector enxtension](https://github.com/pgvector/pgvector). Then we will query for images which are similar to any given images on the data set. We will use a notebook in Collab Enterprise which is a great way of interactively playing with code snippets and iterating like a REPL. It is backed by computing resources (called Runtimes) from Google Cloud which can spin up and scale very quickly. Notebooks in the cloud can also be shared with others who can contribute text and code simultaneously.

Let's get started ...

### Setting up
#### Launching Notebook.
[Collab Enterprise](https://console.cloud.google.com/vertex-ai/colab/notebooks) can be accessed directly under the Vertext AI menu in Google Cloud Console. Once enabled with billing account and all necessary APIs are enabled we should be able to create notebooks.

#### Setup gcloud 
Since we will use various CLI commands from Google Cloud SDK, `gcloud` needs to be installed and authorised with the GCP project whose resources will be used.

    !curl https://sdk.cloud.google.com | bash
    !gcloud init
    !gcloud config set project <project_id>

#### Generate embeddings from Images
For this we will use the [multimodal API from Vertext AI](https://cloud.google.com/vertex-ai/docs/generative-ai/embeddings/get-multimodal-embeddings#api-usage). This API takes both text and image. In the case of image, it takes either the image data (in base64 encoded bytes) or URL of the GCS object. GCS URLs would take less time to transfer the data from local run-time storage, so we would go for that option.

Following is the API Request Payload. We have kept the dimension to be the minimum supported so that we get a quick and small response for the experiment. Larger dimensions would give us more accuracy while searching at a later stage  by retaining more information from the images into the embedding vectors.

#### Request
```
{
  "instances": [
    {
      "image": {
        "gcsUri": "gs://<gcs_bucket>/IMG001.jpg"
      }
    }
  ],
  "parameters": {
    "dimension": 128
  }
}
```
#### Python Client 
```
location : str = "us-central1",
api_regional_endpoint: str = "us-central1-aiplatform.googleapis.com"):
client_options = {"api_endpoint": api_regional_endpoint}
self.client = aiplatform.gapic.PredictionServiceClient(client_options=client_options)

image_struct = instance.fields['image'].struct_value
image_struct.fields['gcsUri'].string_value = gcs_url
instances = [instance]

parameter = struct_pb2.Struct()
parameter.fields['dimension'].number_value = 128
parameters = parameter

endpoint = (f"projects/{self.project}/locations/{self.location}"
      "/publishers/google/models/multimodalembedding@001")
response = self.client.predict(endpoint=endpoint, instances=instances, parameters=parameters)
```

The response would be an array of numbers, similar to the following 
#### Response
```
{
  "predictions": [
    {
      "imageEmbedding": [
        0.00262696808,
        -0.00198890246,
        0.0152047109,
        -0.0103145819,
        [...]
        0.0324628279,
        0.0284924973,
        0.011650892,
        -0.00452344026
      ]
    }
  ]
}
```
The above API (python client) only supports one image at a time so that it can be called for each image found in a GCS bucket where we have uploaded the image files. [There is a Batch API](https://cloud.google.com/vertex-ai/docs/generative-ai/embeddings/batch-prediction-genai-embeddings) for this but that only supports the text far. This is one of the reasons why we chose to use GCS object URLs directly instead of encoding the image into base64 bytes and sending the full content in the API.

### Invoke API and Save Embeddings  
We use Google Cloud Python SDK again to list the images found in the bucket parse the response from the API and save as a CSV file.

```
bucket_name = "birdwalk"
storage_client = storage.Client()
blobs = storage_client.list_blobs(bucket_name)
for blob in blobs:
	instance = struct_pb2.Struct()
	gcs_url = os.path.join("gs://", bucket_name, blob.name)
	image_struct = instance.fields['image'].struct_value
	image_struct.fields['gcsUri'].string_value = gcs_url

	response = client.get_embedding(image_bytes=image_file_contents, gcs_url=gcs_url)
	
	row = response.image_embedding;
	row.insert(0, blob.name) #add the image name as key
	rows.append(row)

with  open('embeddings.csv', 'w') as f:
write = csv.writer(f)
write.writerows(rows)
```

#### Inspect the results
We chose to save the result locally and inspect it either by clicking on the file directly in the notebook runtime file system or just loading it to pandas and printing.
```
import pandas as pd
DATASET_URL='embeddings.csv'
image_embeddings = pd.read_csv(DATASET_URL, header=None)
image_embeddings.head(5)
```
### Setup Cloud SQL
We will create a small instance of Cloud SQL and install the pg-vector extension to support the `VECTOR` data type.  We chose 1 CPU and 1.7 GB memory to minimise cost as we are not going to store and query a lot of data. Cloud SQL instance is charged as long as it's up so we can stop the instance once we are done and start it when we intend to use it again. It will still charge for the storage and IP etc but they are negligible as compared to the instance running cost.

```
    DATASET_URL='embeddings.csv'
    image_embeddings = pd.read_csv(DATASET_URL, header=None)

    loop = asyncio.get_running_loop()
    async with Connector(loop=loop) as connector:
        # Create connection to Cloud SQL database.
        conn: asyncpg.Connection = await connector.connect_async(
            f"{project_id}:{region}:{instance_name}",  # Cloud SQL instance connection name
            "asyncpg",
            user=f"{database_user}",
            password=f"{database_password}",
            db=f"{database_name}",
        )

        await conn.execute("CREATE EXTENSION IF NOT EXISTS vector")
        await register_vector(conn)

        await conn.execute("DROP TABLE IF EXISTS image_embeddings")
        # Create the `product_embeddings` table to store vector embeddings.
        await conn.execute(
            """CREATE TABLE image_embeddings(
                                image_id VARCHAR(1024),
                                content TEXT,
                                embedding vector(128))"""
        )

        # Store all the generated embeddings back into the database.
        for index, row in image_embeddings.iterrows():
            await conn.execute(
                "INSERT INTO image_embeddings (image_id, content, embedding) VALUES ($1, $2, $3)",
                row[0],
                "",
                np.array(row[1:]),
            )
```
### Query For Similar Images
Now we can query the database for similar image to a specific image.
```
        result = await conn.fetch(
            '''
              SELECT
                image_id,
                embedding <-> ( SELECT
                                  embedding
                                FROM image_embeddings
                                WHERE image_id = $1) as distance
              FROM image_embeddings
              WHERE image_id != $1
              ORDER BY distance
              LIMIT 5;
            ''',
            image_id )
```

This query essentially takes the name of the image file as `image_id` and compares its embeddings with the embeddings of other images and returns the top 5 images (excluding the input image whose distance of course will be zero) whose distance is closer to the supplied image. In this example I wanted to find if there are similar images to the photo of the plane and if you notice in the result, it returns 4 planes of which the first two's distance is very less ( < 0.2 because it was the same plane taken from different angle) and other two have higher distance (betwene 0.3 to .4) . The last image is actually of a bird hence has an even higher distance( < 0.5). By doing the following changes the results could have been more accurate i.e. distance of the same plane would have been very small  as compared to a that of different plan or a bird for which distance was expected to be very high. 

- The dimension of the embedding could be increased so that more features of the images could have been used for comparison/distance.
- The input image itself could have been processed (e.g. cropped to the object)  so that the sky which is common in all images couldn't have dominated the similarity. 

I'll soon upload the the fully working notebook (of course Bring Your Own GCP Project).
<!--stackedit_data:
eyJoaXN0b3J5IjpbODA1NDY5Nzc4LC04Mjc0OTg5NDQsMTgxMD
gwNzU3NywtNDYwODMxMTg5LC0xMTEyODcyNjk1LDE5OTI3NDkw
MTcsNzYxODEwMDA0XX0=
-->
