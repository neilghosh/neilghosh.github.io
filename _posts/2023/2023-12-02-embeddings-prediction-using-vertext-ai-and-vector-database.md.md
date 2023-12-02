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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NTgxMDAxNDksNzYxODEwMDA0XX0=
-->