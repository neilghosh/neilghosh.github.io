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

Following is the API contract 
```

```

<!--stackedit_data:
eyJoaXN0b3J5IjpbOTE5MDg2NjA1LDc2MTgxMDAwNF19
-->