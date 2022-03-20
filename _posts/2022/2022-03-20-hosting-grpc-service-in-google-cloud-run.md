![gRPC + Cloud Run](/assets/grpc-plus-cloudrun.png)

REST is the most popular interface to access a web service, [gRPC](https://grpc.io/) is a modern high performance framework that can run in any environment. If you remember [protochol buffers](https://developers.google.com/protocol-buffers), gRPC is the most straightforward way to use it.

[Google Cloud Run](https://cloud.google.com/run) is a serverless platform which can host any container with HTTP endpoints (now gRPC too) and it takes care of scaling, authentication and monitoring on its own. Hence its language agnostic and all you need to know how to package your software in a container exposing an endpoint. 

In this article we will create a simple service which exposes a gRPC endpoint in NodeJS and host it in Google Cloud Run.

First thing, we define the interface in which data is exchanged. So we create a proto file. Complete file [ping.proto](https://github.com/neilghosh/node-grpc/blob/master/protos/ping.proto). 

This contains not just the message format (like a JSON contract of a REST service) but also the operations that is allowrd (like methods in a SOAP WSDL)

```
service PingServer {
   rpc doPing(Ping) returns (Pong) {}
}

message Ping {
    string greetings = 1;
}

message Pong {
    string acknowledgement = 1;
}

```
Next we create the server. We create a main function that starts the gRPC server 

```
function main() {
  var server = new grpc.Server();
  server.addService(ping_proto.PingServer.service, {doPing: doPing});

  const port = parseInt(process.env.PORT) || 50052;

  server.bindAsync('0.0.0.0:'+port, grpc.ServerCredentials.createInsecure(), () => {
    console.log("Starting server in port "+port);
    server.start();
  });
}
```
Now that we have added the service `doPing`, lets add an implementation.

```
function doPing(call, callback) {
  console.log("Request received "+call.request);
  callback(null, {acknowledgement: 'Thank you for ' + call.request.greetings});
}

```

Here is the complete server code. [ping-server.js](https://github.com/neilghosh/node-grpc/blob/master/pingserver/ping-server.js)

We can run and test the server locally 

Lets run the server locally 

`node ./pingserver/ping-server.js &`

We can also build a dcoker image and run so that we know that its packaged properly and good to be deployed in cloud. This is the beauty of containers, once the image is created it would behave exacly same in any other host like it does locally, so a lot of environment setup issues can be ruled out while debugging.

```
docker build . -t gcr.io/$GCP_PROJECT/grpc-ping:latest
docker run -d -p 50051:50051 -e PORT=50051 gcr.io/$GCP_PROJECT/grpc-ping          

```

To invoke the method on the runnning service we could either 
- Create a client in NodeJS. See [ping-client.js](https://github.com/neilghosh/node-grpc/blob/master/pingserver/ping-client.js)
OR
- Use `grpcurl`

```
grpcurl \                                                                                        
    -plaintext -proto protos/ping.proto \
    -d '{"greetings": "Hello"}' \
    localhost:50051 \
    ping.PingServer.doPing
```

You should see a response from the server

```
Response: Thank you for Namaste
```

Now we can deploy this in one of the projects in Google Cloud Platform.

```
gcloud auth login ## Authentocate 
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/grpc-ping ## build the image 
cloud auth configure-docker ## login to the container image registry
docker run -d -p 50051:50051 -e PORT=50051 gcr.io/$GOOGLE_CLOUD_PROJECT/grpc-ping ## run the container locally
gcloud run deploy ping-upstream --image gcr.io/$GOOGLE_CLOUD_PROJECT/grpc-ping
```

Again using `grpcurl` we could test invoke the service 

```
grpcurl \                                                                                        
    -proto protos/ping.proto \           
    -d '{"greetings": "Hello"}' \
    ping-upstream-b3zzuedwgq-uc.a.run.app:443 \
    ping.PingServer.doPing
```

If you are trying to call the service hosted in cloud run you would need SSL in the client code `grpc.credentials.createSsl()` because cloud run ingress adds a default SSL layer.

The latest version of [Postman also supports gRPC](https://blog.postman.com/postman-now-supports-grpc/), if you like a nice UI. It doesn't work in [Postman web](https://twitter.com/neilghosh/status/1494675412277886993) due to browser limitation of gRPC. 

If you want to consume such gRPC service from a Flutter client, [Debkanchan Samadder](https://twitter.com/debkanchans) has created a minimal demo app [DebkanchanSamadder/flutter-grpc-demo](https://github.com/DebkanchanSamadder/flutter-grpc-demo) consuming it. 
