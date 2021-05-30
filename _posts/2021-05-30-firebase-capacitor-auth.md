---
layout: post
title: "Problem with Social Login Provider Tokens using Firebase Authentication with capacitor"
date: 5th May, 2021 18:41:48 +0530
categories: ionic hybridapps capacitor mobile
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

Problem:  angular-fire doesn't work for capacitor based native apps.
Requirement: I needed firebase authentication that works for native apps as well since my application will not be just used as a PWA.

Poking around a bit, I found a few good blog posts which shared the similar problem to some extent.
-   [Josh](https://twitter.com/intent/user?screen_name=joshuamorony)'s Blog : [BlogLink](https://www.joshmorony.com/native-web-facebook-authentication-with-firebase-in-ionic/)

I easily found a package that helps enabling it: [capacitor-firebase-auth](https://github.com/baumblatt/capacitor-firebase-auth)

The library plugin is pretty good.  After following the steps as mentioned in the readme, I was able to get the authentication working for google, facebook & phone login for web view & native app as well.

Pure Web Implementation:  This is when you run your ionic app in browsers.
Native app web view implementation:  This is when you build your app via android/ios native studio & generate native apps like apk/bundles for android.

Now, here is how things start getting different.  In web view layer whenever you authenticate via social providers like google, you are going to get the firebase authentication token & google oauth token as well.
<img src="/assets/img/gauth_res_obfsctd.png" style="width:100%;">

Why would one needs social identity provider's token at all?

