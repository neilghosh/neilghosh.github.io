---
layout: post
title: "Make API Development faster Using Postman"
description: ""
---

[Postman](https://www.postman.com/) is by far the most popular (discounting `curl`) API testing tool used by developers all over the world. There are other tools like [insomnia](https://insomnia.rest/) which also I really like but Postman ts probably the most complete and others have a lot to catch up.

In addition to be merely able to do CRUD API calls like any other rest cients, it allowes Environments to actually save variables that can be used in multiple APIs and can have different values for deferent environments.

For example, of you are testing your API in your local machine and also the same APIs are occasionally tested in your test enviornment as well then insread of you maintaining two set of API contract, you can just use the hostname as a environment variable. 

You can have the following environment 

| Environment   | Key           | Value                    |
| ------------- |:-------------:| ------------------------ |
| Local         | domain        | http://localhost:8080    |
| Test          | domain        | https://test.example.com |
| Prod          | domain        | https://example.com      |

Postman API URL 

```
GET {{domain}}/api/resource
```

At the run time, depending on the selected environment it substitutes the variables and forms the fully qualifies URL.

### Setting the variables at run time.

Sometimes you need to feed the output of one API to another. In that case you may  use the "Test" tab where you can write script to extract values from the outpit of any API and set a environment variable which can be used as inpout to another API call. For example any auth API gives the following response containing the auth token which can be used in subsequent API calls.

```
{
  "access_token": "eyJz93a...k4laUWw",
  "refresh_token": "GEbRxBN...edjnXbL",
  "id_token": "eyJ0XAi...4faeEoQ",
  "token_type": "Bearer"
}
```

Now you can write a `Tests` script as follows 

```
postman.setEnvironmentVariable("token", JSON.parse(responseBody).access_token);
```

And use it in API header 

| Key           | Value                    |
|:------------- | ------------------------ |
| Content-Type  | application/json         |
| Authorization | Bearer {{token}}         |

Note that these variables can be used anywhere in the API contract including, the body.

You can also write complex script in case the output response is an array. e.g.

For Response 
```
{
  "compositeResponse": [
    {
      "item": a,
      "key": "value"
    },
    {
      "item": b,
      "key": "value"
    },
    {
      "item": c,
      "key": "value"
    }
  ]
}
```

```
var data = JSON.parse(responseBody).compositeResponse;

for(i=0 ; i < data.length ; i++) {
    if(data[i].item == "a") {
        postman.setEnvironmentVariable("a", data[i].key);
    }
    
    if(data[i].referenceId == "b") {
        postman.setEnvironmentVariable("b", data[i].key);
    }
    
    if(data[i].referenceId == "c") {
        postman.setEnvironmentVariable("c", data[i].key);
    }
}
```










