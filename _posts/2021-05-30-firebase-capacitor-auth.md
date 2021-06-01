---
layout: post
title: "Problem with Social Login Provider Tokens using Firebase Authentication with capacitor"
date: 5th May, 2021 18:41:48 +0530
categories: tech mobile hybridapps ionic capacitor 
tags: [ionic, hybridapps, capacitor, mobile]
---

In this post, I'll try to write up on a problem I faced while trying to use capacitor with firebase authentication.  This can be a bit advanced post for newcomers in firebase usage with capacitor.  Basic knowledge of the following is required before going any further.
1. Capacitor [Site](https://capacitorjs.com/) [Github](https://github.com/ionic-team/capacitor)
2. Firebase  [Site](https://firebase.google.com/)  [firebase](https://github.com/firebase)
3. Ionic  [Site](https://ionicframework.com/) [Github](https://github.com/ionic-team/ionic-framework)
4. AngularFire [Github](https://github.com/angular/angularfire)


## Context:

I have been working on an application where I decided to use Firebase for its obvious advantages.  I was quickly able to get the pieces together using ionic framework, then added `angular-fire` auth to get basic datastore & firebase authentication working.

My requirements/desire from the project changed over a period of time & thought of adding mobile capabilities as well.  The chances of this late decision working out was quite good from 10 thousand feet view since anyway the project was bootstrapped with Ionic.

`Problem`:  angular-fire doesn't work for capacitor based native apps.\
`Requirements`: I needed firebase authentication that works for native apps as well since my application will not be just used as a PWA/browser based.  After logging in via social identity provider(Google), I also needed to fetch user contacts to check if any of the contacts are already registered on the application. 

Poking around a bit, I found really good blog post which shared the similar problem to some extent.
-   [Josh](https://twitter.com/intent/user?screen_name=joshuamorony)'s Blog : [BlogLink](https://www.joshmorony.com/native-web-facebook-authentication-with-firebase-in-ionic/)

Josh has indeed written a very good blog on the same subject matter but my problems were a bit different.
-   I was using capacitor & not cordova & I really didn't know How Exactly Capacitor Plugins Work? & why angular-fire just not work.
-   Anyway, Angular-fire wasn't going to work for native apps. (The why needs a seperate blog of its own! )
-   I needed to get data from google provider to fetch contacts for every user & for that I needed oAuth token.
-   So, I had to change my auth service layer from using angular-fire auth module to something else. 

I searched a bit more, found a library/capacitor plugin that helps enabling it: [capacitor-firebase-auth](https://github.com/baumblatt/capacitor-firebase-auth)

The library/plugin is pretty good.  After following the steps as mentioned in the readme, I was able to get the authentication working for google, facebook & phone login for web view & native app as well.

<div style="text-align:center;">
    <img src="/assets/img/sub-buddy-login.png" style="width:50%;display:unset">
</div>


Lets try to bolster a few meanings here!
<div style="display:flex;">
    <img src="/assets/img/warning-17.png" style="width:30px"> 
    <span style="align-self:center;color:yellow;">Nomenclature alert!</span>
</div>
{% highlight markdown %}
`Pure Web Implementation`:  This is when your app is made with intention to run only in browsers.(mobile or otherwise)
`Native app web view based implementation`:  This is when you build your app via android/ios native studios & generate native apps.
Example: Generating apk/bundles for android via android studio.
{% endhighlight %}


Okay, so you must obviously be thinking about what does all of the above have in relation to the title & whats the problem with it??  So lets dive in.
### Why would one needs social identity provider's token at all?
There can be many use cases for the same.  Lets see a few for examples:
-   Fetching user's contacts via gmail using people api.
-   Fetching user's followers/friends via twitter or facebook apis.
-   Fetching user's information stored in different services like google analytics or google drive.

All of this data can be fetched by requesting to provider using an oauth token that one recieves on succesfull authentication & authorization by one's account.  Hope you all are familiar with the google consent screen.
<div style="text-align:center;">
    <img src="/assets/img/google_consent.png" style="display:unset;"> 
</div>
<br />
So, here is the situation.  In `pure web implementation` whenever you do firebase authentication via social providers like google, you are going to get the firebase authentication token & google oauth token as well.  The picture attached shows the object you get once you do firebase authentication via google identity provider on web.
<img src="/assets/img/gauth_res_obfsctd.png" style="width:100%;">

<br/>
The `idToken` you recieve is a valid JWT token, given to you via firebase authentication.  \
The `accessToken` or `oAuthAccessToken` is the social identity provider's token which can be used to fetch further information specific to the identity provider like google.

Using this access token, you can make calls to People Api, drive API etc which give you access to contacts, google sheets & other data or documents.


### Problem with library for identity provider's token
When I build the capacitor app for android device, the firebase authentication flow is for a non-browser ie. an android native flow.  The [library](https://github.com/baumblatt/capacitor-firebase-auth) does a great job at implementing it but this specific use case like fetching & passing through to use the access-token in grant flow got left or may be it was simply not intended.  The web flow which is browser based flow (probably using `authorization code` or `implicit` flow) indeed takes care of it.
I must recommend going through OAuth flows in case of lack of clarity. [Oauth](https://oauth.net/2/grant-types/authorization-code/).

So in order to get access token on native platform one have to complete the remaining flow.  As also mentioned in the firebase documentation [here](https://firebase.google.com/docs/auth/web/google-signin#handle_the_sign-in_flow_with_the_firebase_sdk) 
It is clearly written that in custom implementation cases you must complete the flow manually.
<img src="/assets/img/firebase-manual-flow.png" style="width:100%">

I filed [issue 159](https://github.com/baumblatt/capacitor-firebase-auth/issues/159) on the same & this blog post is pretty much in depth explanation of how I contributed towards enabling the functionality.

In order to reach our objective to fetch contacts via People API against Google Oauth token, I am taking a step back to show the flow & steps needed diagrammatically. 

Standard OAuth flow.
{% highlight markdown %}
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+

Src: https://datatracker.ietf.org/doc/html/rfc6749#section-1.2
{% endhighlight %}

-   (A) is taken care by firebase authentication.  But we need to additionally specify while making the request from native app to get offline access.
On Android, how to do this is very kindly explained [here](https://developers.google.com/identity/sign-in/android/offline-access).\
Further difference between google builder specific functions can be seen here in [Documentation](https://developers.google.com/android/reference/com/google/android/gms/auth/api/signin/GoogleSignInOptions.Builder#public-googlesigninoptions.builder-requestserverauthcode-string-serverclientid)
-   (B) An authorization code/grant is provided with the response.  Now for obtaining the access_token, we need to send the `authorization code` to our backend server to get it exchanged for access_tokens.
One can try and do it on the frontend as well but I believe that such tasks must be delegated to backend to maintain sanity, security & order in the frontend code.\
On the server side, I wrote a cloud function for firebase which does the following:
    -   Validate for accessToken or Authorization code.
    -   If valid authorization code, get oauth accessToken in return.
    -   Make api calls to People Api resource endpoint at google & fetch connections. 

Lets walk through the code which takes care of step C & D from the above diagram.


<!-- #TODO: Gotta get these code blocks to collapsible blocks -->
<!-- <details>
    <summary>Click to see code!</summary>

    ```ts
        function whatIsLove() {
            console.log('Baby Don't hurt me. Don't hurt me');
            return 'No more';
        }
    ```
</details> -->

<br />
```ts

// this call expects request having a body of type 
export interface SyncConnectionsRequestBody {
    accessToken?: string;
    authCode?: string;
    email: string;
    uid: string;
    native: boolean;
}

// mounted an express app for firebase functions.
app.use("/syncConnections", async (req: any ,res: any) => {
    console.log("gonna sync user connections on reqeust!!",req.hostname,req.method,req.path,req.body);
    let data : SyncConnectionsRequestBody = req.body.data;

    console.debug("data recieved from user request",data);
    // custom object wrapper to sent response wrapped in.
    let resp : CloudFunctionsResponseType = {
        functionName : "syncUserConnections",
        status : "OK",
        data: ""
    };

    // this check if either or accessToken or authCode are present!
    // This is only done to ensure this function works in case of auth-code based flow or if accesstoken is provided directly. 
    if(!data.accessToken && !data.authCode){
      let ermsg="no access token or authCode given!";
      console.debug(ermsg);
      resp.status="ERR";
      resp.data=ermsg;
      return res.status(400).send(ermsg);
    }

    // checks if call originated from native devices; this gets set at client ionic capacitor app via isNative plugin call. 
    if(data.native && data.authCode){
      console.debug("call recvd from native platform");
      getOauthTokenFromAuthCode( data.authCode )
        .then(resp => {
          console.debug("token recieved using authCode",resp);
          if(resp.res?.status == 200 && resp.tokens.access_token){
            let newBody : SyncConnectionsRequestBody = {
              accessToken : resp.tokens.access_token,
              email : data.email,
              uid: data.uid,
              native: data.native
            }
            return getConnections(newBody,res);
          }
          else{
            throw new Error(`${resp.res?.status} Error`);
          }
        })
        .catch(err => {
          console.error("error occured while getting token from authCode",err);
          return res.status(400).send(err);
        })
    }
    else{
      console.debug("call recvd from NON-native platform");
      return getConnections(data, res);
      // return res.send(resp);
    }
})
```

Add another file in scope by the name of `getGoogleTokensFromAuthCode.ts`
```ts
// var {google} = require("googleapis");
import { google } from "googleapis";
import * as functions from "firebase-functions";

const OAuth2 = google.auth.OAuth2;
const functionsConfig : functions.config.Config = functions.config();
console.debug("functions config on start!",functionsConfig);

if( !functionsConfig || !functionsConfig["web"] ){
    let ermsg = "OAuth config not Set";
    console.error(ermsg,functionsConfig);
    throw new Error(ermsg);
}


// this you need to supply from your firebase app admin settings/environment.
const webOauthCreds = functionsConfig["web"];

function getOAuthClient() {
    return new OAuth2(webOauthCreds.client_id, webOauthCreds.client_secret);
}


export async function verifyIdToken( oauth2Client:any , oauthIdToken : string ){
    console.debug("verifying id token:",oauth2Client,oauthIdToken);
    let ticket = await oauth2Client.verifyIdToken({
        idToken : oauthIdToken,
        audience: webOauthCreds.web.client_id
    });
    if(!ticket){
        let ermsg = "somehow not ticket is obtained!!"
        console.error(ermsg);
        throw new Error(ermsg);
    }
    return ticket;
}

export function getOauthTokenFromAuthCode(authCode: string){
    var oauth2Client = getOAuthClient();
    return oauth2Client.getToken(authCode);
}

```


```ts
// sample code for getting connections from google poeople api.

export async function getConnections( data: any, res: any){
  let connectionsStore : Array<any> = [];
  let resp : CloudFunctionsResponseType = {
    functionName : "syncUserConnections",
    status : "OK",
    data: ""
  };
  let allConnectionsPulled;
  // let data = req.body.data;

    try{
      allConnectionsPulled = await getGoogleContactsByPageToken(data, connectionsStore);
    }
    catch( err ){
      console.error("error occured while getting google contacts",err);
      return res.send(resp);
    }
    
    const onAllConnectionLoaded = () => {
        console.log("all connections pull completed!", connectionsStore);
        // this is using lamdajs function to flatten the array here.
        connectionsStore = _.flatten(connectionsStore);
        let chunks = splitToChunks( connectionsStore ,environment.DOCUMENT_CONNECTIONS_LIMIT);
        

        // delete all previous connections in case user had.
        let connectionsQuery = db.collection("connections").where("_owner","==",data.uid);
        return connectionsQuery.get()
            .then( (querySnapshot) => {
                // delete all previously existing connections
                let allpromises : Promise<any>[] = [];
                querySnapshot.forEach( (doc) => {
                    allpromises.push(doc.ref.delete());
                });
                return Promise.all(allpromises)
            })
            .then( () => {
                // break the new connections list into chunks & upload the chunks
                return Promise.all(
                    chunks.map( (chunk : any) => {
                    let connectionBody : UserConnections = {
                        connections: chunk,
                        syncdAt : new Date().toISOString(),
                        _owner : data.uid,
                        src : "google.com"
                    };
                    // this is simply a wrapper for creating documents inside connections collection in firebase database.
                    return doDocument("create","connections", null,connectionBody)
                    })
                )
            })
            .then(resp => {
                console.log("all connections uploaded to DB",resp);
                return doDocument("update","users",data.uid,{
                syncContacts : false
                })
            })
    };

    try{
      if(allConnectionsPulled){
        await onAllConnectionLoaded();
        resp.status="OK";
        resp.data = connectionsStore;
        return res.send(resp);
      }
      else{
        // resp.status="ERR";
        throw new Error("not all connection be pulled!");
      }
    }
    catch(err){
      resp.status="ERR";
      console.error("error occured in onAllConnection Loaded promise",err);
      return res.send(resp);
    }
}


//  this function is the layer that makes the API call.
function getGoogleContacts (credential : any, pageToken : string|null =null) : Promise<FetchResponse> {
    if(!credential)
      return Promise.reject("No credentials for call");
    
    console.debug("get google contacts called",credential);
    const googleHost = "people.googleapis.com";
    const pageSize=500;
    const contactsDetailRequired = ["phoneNumbers","genders","emailAddresses", "names", "ageRanges", "interests", ];
  
    const requiredFields = contactsDetailRequired.join(",");
    const targetUrl = `https://${googleHost}/v1/people/me/connections?personFields=${requiredFields}&access_token=${credential["accessToken"]}&pageSize=${pageSize}`;
    
    if(pageToken){
      targetUrl += `&pageToken=${pageToken}`;
    }
  
    return fetch(targetUrl)
      
}
  
async function getGoogleContactsByPageToken(credential : any, connectionsStore : Array<any>) : Promise<any> {
let nextPageToken=null;
do{
    console.debug("running loop for nextPageToken Value",nextPageToken, credential);
    try{
      await getGoogleContacts(credential, nextPageToken)
      .then( (res: any) => res.json() )
      .then( (connectionsResponse : any) => {
          // console.debug("connections call response",connectionsResponse);
          connectionsStore.push(connectionsResponse.connections);
          nextPageToken = connectionsResponse.nextPageToken;
          console.log("setting next page token!",nextPageToken);
      })
    }
    catch(err){
      console.error("error occured while fetching all google contacts!!",err);
    return false;
    }
}
while(nextPageToken);
return true;
}
```


## Summing Up:  
-   So, we enhanced library by adding support for obtaining oauth token for android apps.
-   Looked up some code for validating & getting code->oauthtoken in return.
-   Finally, pulled all user connections & uploaded to firebase db.
-   Client side code (java + javascipt as capacitor plugin) is in my [forked repo](https://github.com/beyondszine/capacitor-firebase-auth)

Make sure, you don't save people's connections in your private database unless you have explicit permissions for the same.  This code is only intended for educational & reference purposes.
In case your use-case pushes you to do the same, I recommend you read the famous [signal blog](https://signal.org/blog/private-contact-discovery/) as well.