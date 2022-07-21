---
layout: post
title: "Building Application With Google oAuth Identity"
description: "Building Application With Google oAuth Identity"
--- 
**oAuth**

Today we see so many applications with "Sign In With Google" button (Well 10 years back we thought everything will be a "G+" button but that didn't happen :)). What it does that is it lets you provide Google Credential (of course by redirecting to a valid Google Sign In page which you trust) and sign in exactly how you would sign in to a Google service like Gmail or Google Photos then it comes back to the original website and continue as a logged in user. This works as a "Single Sign On" where you don't have to sign in to website using its own username/password and remember those hundreds of credentials for various sites you use on daily basis. Even yahoo.com who ones was known as rivals of google services (especially in terms of email) has a "Sign In With Google Button". Even if you have a dedicated username/password for the website since your google account was linked to those credential the website could recognise you if you can just proove that you own that gmail id. An oAuth Client is created for interecting with Google behalf of you.

<table>
  <tr>
    <td>
<img src="./yahoo-signin.png" /> </td><td> <img src="google-oauth-client.png" /> 
    </td>
    </tr>
  </table>

One can go to Google Cloud Console (https://console.cloud.google.com/apis/credentials) and create a Client. Here are more documentation.
https://support.google.com/cloud/answer/6158849?hl=en

**Scope**

There may be websites which actually may provide some services by interacting with Google services via your google account. Say a site lets you download a file and also gives you an option to save to your google drive directly without even having to save it locally. In that case this website is uploading a file directly from their server directly using your credentials. You signed in with google, doesn't mean the website has unrestricted access to all your google service. It needs to have your concent to do so. So the developer of the website needs to prompt your for some permission to be granted. This is scope. When the website redirects to the login page of Google it also needs to tell that it is seeking user's permission to grant access. For example in the Google Drive example you should see a "Concent Screen" asking Read/Write permission to Google Drive. Once you allow this concent is stored permanently against your oAuth Client (Google identifies the website using this.). If you have developed an app with Facebook Connect then you may have developed a Facebook App which acts as a oAuth Client. The following code tells what scopes to be asked for where to redirect back once the concent received.

```
// Access scopes for read-only Photos.
const scopes = ["https://www.googleapis.com/auth/photoslibrary.readonly"];

// Generate a url that asks permissions for the Drive activity scope
const authorizationUrl = oauth2Client.generateAuthUrl({
  // 'online' (default) or 'offline' (gets refresh_token)
  access_type: "offline",
  /** Pass in the scopes array defined above.
   * Alternatively, if only one scope is needed, you can pass a scope URL as a string */
  scope: scopes,
  // Enable incremental authorization. Recommended as a best practice.
  include_granted_scopes: true,
});
```

![Consent Screen](./consent-screen.png) 

Documentation writting server side web apps with Google oAuth 
https://developers.google.com/identity/protocols/oauth2/web-server

Finally once the page is redirected to your app you can get hold of the access token and establish a session. In future articles we can discuss the session handling and how using this access token further calls can be made to Google Services i.e. Google Drive and Google Photos.
