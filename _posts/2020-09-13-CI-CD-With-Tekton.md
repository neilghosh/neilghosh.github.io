It has been a while. While I was doing prep for a CPF for one of the top global Indian conferences, I delved into Knative. I was really impressed by Cloud Run by Google Cloud Platform and was curious to know more about the underlying stack which is Knative serving. Serving, Build and Eventing were 3 major pillars of Knative but build is now knows as Tekton kind of has its own reputation know as a CI/CD tool. Once can install Tekton in a Kubernetes cluster and a full fledged CI CD stack is ready to go. K8s acts as the underlying infrastructure. Of course other tools like Jenkins can be container-ised and installed in Kubernetes but Tekton can natively installed a CRDs in its own namesapce. 

Here are the overall steps which one can refer to play around Tekton. This starts with building code from a github repo to deploy the docker image to a registry. And finally use that image to deploy to Kubernetes both as an standard deployment and Knative service. 

[https://github.com/neilghosh/simple-tekton-pipeline](https://github.com/neilghosh/simple-tekton-pipeline)

The good thing is Tekton doesn't really need Knative or has to be used only for Knative apps. Its a general purpose CI/CD just like Jenkins and Spinnaker which can be used to automate anything.

