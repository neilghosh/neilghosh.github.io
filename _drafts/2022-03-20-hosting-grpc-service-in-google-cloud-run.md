REST is the most popular interface to access a web service, gRPC is a modern high performance framework that can run in any environment. If you remember [protochol buffers](https://developers.google.com/protocol-buffers) gRPC is the most straightforward way to use it.

Google Cloud Run is a serverless platform which can host any container with HTTP endpoints (now gRPC too) and it takes care of scaling, authentication and monitoring on its own. Hence its language agnostic and all you need to know how to package your software in a container exposing an endpoint. 

In this article we will create a simple service which exposes an gRPC endpoint in NodeJS and host it in google cloud run.

