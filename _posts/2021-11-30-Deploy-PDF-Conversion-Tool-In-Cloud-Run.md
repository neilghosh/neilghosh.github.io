---
layout: post
title: "Deploying PDF Conversion Tool In Google Cloud Run"
description: "Google Cloud Run is a great service to quickly deploy any containerized application. In this demo we will see how one can develope and deploy a PDF conversion tool which reduces size of scanned multi-page PDF file using ImageMagick"
---

Google Cloud Run is a great service to quickly deploy any containerized application. In this demo we will see how one can develope and deploy a PDF conversion tool which reduces size of scanned multi-page PDF file using ImageMagick

![gcping](/assets/imagemagick-convert.png)
      
[ImageMagick](https://imagemagick.org/index.php) is is a great open source command line tool which can convert files with various image file format including PDF.

To reduce size of a multiple page PDF file we would do the following 
- Split the PDF file to multiple image of smaller size (lossy .jpeg) one per page.
- Reduce contrast level of each image
- Combine reduced image files back to a single multi-page PDF file.

In terminal you would typically run the following commands.

- `convert -density 75 input.pdf output-%02d.jpg`
- `convert output*.jpg -level 15% final-%02d.jpg`
- `convert final*.jpg output.pdf`

![gcping](/assets/resize-pdfs.png)

This works great in local machine but to run this utility in cloud we would need to install it in the container in which the application is packaged. So in `Dockerfile` we would add commands to install this.

```
RUN apk add  --no-cache imagemagick
```

We will also need some storage to save the intermediate .jpg files temporarily. For this we would make use of the ephemeral storage inside the container. 

 *Note* 
This is not recommended because the file size can grow and when multiple requests comes to same instance it would run out of storage. A better way to store such files is to use a distributed storage like Google Cloud Storage which is more reliable and scalable. 

Also in this demo we are doing all the conversions synchronously i.e. we will get the output file in the response of the upload request itself. It is recommended to process the file asynchronously and return some identifier or url to access the output later as part of a different request when processing is finished. One can also give a response with a different endpoint which provides a status (Pending/Processing/Completed etc) of the long running process like this.

This is a NodeJS app so we would use the [ImageMagic Node](https://www.npmjs.com/package/imagemagick) library. The API equivalent of the command would be 

```
    ImageMagick.convert(
      ["-density", density, "uploads/" + source, "results/" + path_to],
      function (err, stdout) {
        if (err) {
          reject(err);
        }
        resolve(stdout);
      }
    );
```
At the end we would also want to clean up the intermediate files from the file system. We use [glob](https://www.npmjs.com/package/glob)

```
    glob("results/" + jpegFilePattern, {}, function (er, files) {
      files.forEach(function (file) {
        fs.unlinkSync(file);
        console.log("Deleting " + file);
      });
      console.log("cleaned up jpegs asynchronously");
    });
```

### Testing Locally 
We can test locally by bringing our express server up with 
```
NPM_CONFIG_REGISTRY=https://registry.npmjs.org npm install
npm start
```
For a realistic testing how it would behave in cloud we containerize it and then run it with the port exposed.

```
# Build
docker build . -t neilghosh/pdf-tools
# Run the image
docker run  -p 3000:3000  neilghosh/pdf-tools
```

If we need to get into the container to see if file is getting uploaded and converted

```
docker exec -it c72973eea10a  ash
```
*Note - since we are using the light weight `node:alpine` base image we use ash instead of bash

A simple pdf conversion can be done like follows.

```
curl -v -F "density=50" -F "foo=@input.pdf" http://localhost:3000/upload --output output.pdf
```
** I know foo is a bad choice for attribute :) I will refactor later.

### Deploy in Cloud Run

Authenticate `gcloud` cli with your project and deploy directly from the code directory.
```
gcloud auth login  
gcloud config set project <GCP_PROJECT_NAME>
gcloud run deploy
```
In the interactive console we will be asked for various parameters like which region this should be hosted. I choose Mumbai because as per [gcping.com](https://gcping.com/) Mumbai had the least latency from my location.

![gcping](/assets/gcping.png)

Finally after successful deployment it would give a fully qualifies URL where the service would be accessible. This is the beauty of serverless platform like Google Cloud Run. So now you can convert the local PDF file to a smaller size as follows 

![gcping](/assets/cloudrun.png)

```
curl -v -F "density=75" -F "foo=@input.pdf" https://pdf-tools-b3zzuedwgq-el.a.run.app/upload --output output.pdf
```

**Note - Since image conversion is a relatively heavy process, when I tried the default Cloud Run deployment configuration and use it, I got insufficient memory error. So I had to edit the cloud run app to have 1GB of memory instead default 512Mb. 

** Next, we would try to implement the following.
- Make this synchronous. 
- Use Cloud Storage instead if ephemeral storage.
- Expose more knobs to the use control the quality of the conversion i.e.  contrast level.

