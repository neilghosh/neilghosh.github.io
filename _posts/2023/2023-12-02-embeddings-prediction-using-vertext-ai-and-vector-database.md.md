---
layout: post
title: "Embeddings Prediction using Vertext AI and Vector Database"
description: ""
---
Google Cloud Vertex AI's Multimodal Prediction Models lets us create embeddings from Images (or even text for that matter). By default the embeddings are generated as 1408-dimension vectors however this value is configurable.

These embeddings can be stored in a vector database and verious queries like finding similar images can be done on the stored vector data.

In this post we will generate embeddings from photos stored in Google Cloud Storage and save them PostgresSQL database with pg-vector enxtension. Then we will query any images which are similar to any given images on the data. We will use a notebook in Collab Enterprise which is a great way of interactively play with code snippets and iterate like a REPL. It is backed by computing resources (called as Runtimes) from Google Cloud which can spin up and scale very quickly. Notebooks in cloud also can be shared with others who can contribute text and code simultaniously.

Let's get started ...

### Setting up
#### Launching Notebook.
[Collab Enterprise](https://console.cloud.google.com/vertex-ai/colab/notebooks) can be accessed directly under the Vertext AI menu in Google Cloud Console. Once all billing account and all necessary accounts are enabled we should be able to create notebooks.

#### Setup gcloud 
Since we will use various CLI commands from Google Cloud SDK, `gcloud` needs to be installed and authorised with the GCP project whose resources will be used.

    !curl https://sdk.cloud.google.com | bash
    !gcloud init
    !gcloud config set project <project_id>

#### Generate embeddings from Images
For this we will use the [multimodal api from Vertext AI](https://cloud.google.com/vertex-ai/docs/generative-ai/embeddings/get-multimodal-embeddings#api-usage). This API takes both text and image. In case of image it takes either the image data (in base64 encoded bytes) or URL of the GCS object. GCS URLs would take less time to transfer the data from local run time storage, so we would go for that option.

Following is the API Request Payload. We have kept the dimension to be the minimum supported so that we get a quick and small response. Larger dimention would gives us more accuracy while searching at later stage (by retaining more information from the images)

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

The response would be a an array of numbers, similar to following 
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
  ],
  "deployedModelId": "DEPLOYED_MODEL_ID"
}
```
The above API (python client) only supports one image at a time, so it can be called for each image found in a GCS bucket were we have uploaded the image files. [There is a Batch API](https://cloud.google.com/vertex-ai/docs/generative-ai/embeddings/batch-prediction-genai-embeddings) for this for that only supports the text as input so far. This is one of the reason why we choose to use GCS objects URLs directly instead of encoding the image into base64 bytes and sending the full content in the API.

### Invoke API and Save Embeddings  
We use google cloud python sdk again to list the images found in the bucket and parse the response from the API and save as CSV file.

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
W
```
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgyNTYwMjY4MSwtMTExMjg3MjY5NSwxOT
kyNzQ5MDE3LDc2MTgxMDAwNF19
-->