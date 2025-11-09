---
layout: post
title: "Building Apps with Google Photos API"
description: "While Google Photos has an very good UI to search, upload and brows through your Photos stored, it also provides a lot of APIs to read and manipulate photos data if you choose to build any apps of your own to work with Google Photos, of course your user needs to authenticate with their Google account first"
---

[Previously](building-application-with-google-oauth-identity.html) I talked about creating oAuth2 apps in Google Cloud by creating credentials and adding a scope in your code such that the user is prompted for the consent for the permissions such that your app can read/write using specific APIs like Google Drive or Google Photos.

![Google Photos API](/assets/2022/google-photos-icon.png) 

In this case we would just like to list the albums that the user has in Google Photos and the photos that belongs to the album they choose to select.

Firstly, we need to handle the callback that we get after the user enters their credential in Google Login screen and allowed access to services that you have added as scope. e.g. here its https://www.googleapis.com/auth/photoslibrary.readonly. When the redirect happens it appends a Auth Code in the URL which we have to grab and then exchange it with a token using Google oAuth Client. The process is described [here](https://developers.google.com/identity/protocols/oauth2/web-server#exchange-authorization-code).

Once we get the token we could either save it in a secure persistence service in Server, say in a SQL DB or in a Redis cache which we can retrieve later when subsequent request comes from user. However for simplicity we would store it in [session](https://www.npmjs.com/package/express-session) which will be active as the client (user's browser) and server interact back and forth till the user logs out. The session is usually maintained using cookies. Session data is stored in server but the identifier goes to client as a cookie.

Finally we make a call to Google Photos APIs to retrieve the albums. We can use the above token to make a REST call to the `GET /v1/albums`. [API Reference](https://developers.google.com/photos/library/guides/list#listing-albums)

```
function getAlbums(access_token) {
  const options = {
    hostname: "photoslibrary.googleapis.com",
    port: 443,
    path: "/v1/albums",
    method: "GET",
    headers: {
      "Content-Type": "application/json",
      Authorization: "Bearer " + access_token,
    },
  };

  return new Promise((resolve, reject) => {
    const req = https.request(options, (res) => {
      res.setEncoding("utf8");
      let responseBody = "";

      res.on("data", (chunk) => {
        responseBody += chunk;
      });

      res.on("end", () => {
        resolve(JSON.parse(responseBody));
      });
    });

    req.on("error", (err) => {
      reject(err);
    });

    //req.write(data);
    req.end();
  });
}
```

Well this is native NodeJS but the above code could be simple if we use a REST client framework like [axios](https://www.npmjs.com/package/axios).

The above call returns a JSON with an array of albums. We could parse it and render in our app, say in an HTML template. For example we could pass the album data into an HTML page using [EJS template engine](https://ejs.co/).

![EJS Logo](/assets/2022/ejs-logo.png) 


```
  app.set("view engine", "ejs");

  ...
  const albumsData = await getAlbums(access_token);

  res.render("albums", {
    albums: albumsData,
  }); // index refers to index.ejs
```
and in `albums.ejs`

```
<html>
<head>
  <title>Photos Album</title>
</head>

<body>
  <% albums.albums.forEach(function(album) { %>
    <li>
      <h3><%=album.title%></h3>
      <p>
        <img src="<%=album.coverPhotoBaseUrl%>=w300-h300" />
      </p>
    </li>
    <% })%>
</body>
</html>
```

This should show all the albums as a list with each one's cover photo rendered.

In next article we would see how to write back Google photos using an elevated scope. 