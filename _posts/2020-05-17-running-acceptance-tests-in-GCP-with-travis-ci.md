---
published: false
---
## Running Aacceptance Tests In GCP With Travis CI

In the various stages of continous integration, its really important to run all the tests. Some of them are unit tests which can run without any dependency. However you may have acceptance tests or integration tests which can have dependency on databases and other 3rd party services.

Firstly, often these tests are run on a specific environment (other than dev/test/stage and of-course prod) which contains the latest build (or at the code level for which we are trying to run the build). This is often a temporary environment/deployment whcich is torn down after the tests has run.

In this example I have a [simple microservice written in go](https://github.com/neilghosh/go-starter-service) and it has a test which tests an API which internally reads/writes to database, Google Cloud Datastore in this case.

In an ideal case, where you don't want to have any external dependency and run locally the databases must use the emulators. E.g. [emulator for Google Cloud Datatsore](https://cloud.google.com/datastore/docs/tools/datastore-emulator). However if you are running a more high level tests in your build pipeline which is running against a real environment you need to provide a GCP project with datastore access.

This could be a dedicated project for continous integration or an existing project with a dedicated namespace so that it does not mess with data of the project. Of course, it is advisable to always setup and clean up your data before and after the test, also isolate the tests with unique set of data i.e. you can create the dataset for tests with an unique UUID as prefix so that if the similar tests are running in parallel, it will have not have contention of data with each other.

How would the test access the data? The tests are running in your build machine or the cloud based CI system like Travis CI or Google Cloud Build. So this needs a service account key of the specific GCP project to talk to Datastore. Since we can't really checkin the secret key file in the code repository we need to checkin an encrypted version and configure travis ci with dycription process.

Following are the steps 

- [Connect your GitHub](https://github.com/marketplace/travis-ci) to Travis CI account

- [Create a Service Account](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) and download the json key. This can have only datastore read write permission (priciple of least access) and named appropriately. 

- Use the [Travis CI CLI](https://github.com/travis-ci/travis.rb#readme) to [encrypt](https://docs.travis-ci.com/user/encrypting-files/) and [configure the key](https://docs.travis-ci.com/user/deployment/google-app-engine/). 

```
travis encrypt-file client-secret.json --add
```

- Chek-in the `.enc` file. DO NOT CHECK-IN the secret json key file ever. The above command will also add a command in your `.travis.yaml` file to decrypt the file back to json file.

- Tell the build system to look for this JSON file for authentication to the GCP project. Add an environment variable GOOGLE_APPLICATION_CREDENTIALS=<KEY_FILE_NAME>.json. The final `.travis.yaml` looks like 

```
language: go
env:
  - GOOGLE_APPLICATION_CREDENTIALS=demoneil-adc7162180c1.json
before_install:
- openssl aes-256-cbc -K $encrypted_4767c4c300b2_key -iv $encrypted_4767c4c300b2_iv
  -in demoneil-adc7162180c1.json.enc -out demoneil-adc7162180c1.json -d
```

The above configuration should by default run the `go test` in the project when you raise a pull request as checks.

![travis-check.jpg](/assets/travis-check.jpg)

You can also embed the build status badge in your README file with the following snippet.

```
[![Build Status](https://travis-ci.com/neilghosh/go-starter-service.svg?branch=master)](https://travis-ci.com/neilghosh/go-starter-service)
```

![build-status.png](/assets/build-status.png)

