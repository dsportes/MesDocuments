---
layout: page
title: Notes techniques à propos de FCM (Firebase Cloud Messaging)
---

## FCM Credentials

Depending on which FCM features you implement, you may need the following credentials from your Firebase project:

### Project ID 	
A unique identifier for your Firebase project, used in requests to the FCM v1 HTTP endpoint. This value is available in the Firebase console Settings pane.

### Registration token 	
A unique token string that identifies each client app instance. The registration token is required for single device and device group messaging. Note that registration tokens must be kept secret.

### Sender ID 	
A unique numerical value created when you create your Firebase project, available in the Cloud Messaging tab of the Firebase console Settings pane. The sender ID is used to identify each sender that can send messages to the client app.

### Access token 	
A short-lived OAuth 2.0 token that authorizes requests to the HTTP v1 API. This token is associated with a service account that belongs to your Firebase project. To create and rotate access tokens, follow the steps described in Authorize Send Requests.

### Server key (for **deprecated** legacy protocols) 	
A server key that authorizes your app server for access to Google services, including sending messages via the deprecated Firebase Cloud Messaging legacy protocols.

Important: Do not include the server key anywhere in your client code. Also, make sure to use only server keys to authorize your app server. Android, Apple platform, and browser keys are rejected by FCM. 

## Web SDK
Configuration à transmettre à l'application Web pour recevoir des messages.

https://firebase.google.com/docs/cloud-messaging/js/client#web

firebaseConfig

    import { initializeApp } from "firebase/app";
    import { getMessaging } from "firebase/messaging";

    // TODO: Replace the following with your app's Firebase project configuration
    // See: https://firebase.google.com/docs/web/learn-more#config-object
    const firebaseConfig = {
      // ...
    };

    // Initialize Firebase
    const app = initializeApp(firebaseConfig);


    // Initialize Firebase Cloud Messaging and get a reference to the service
    const messaging = getMessaging(app);

vapidKey

    import { getMessaging, getToken } from "firebase/messaging";

    const messaging = getMessaging();
    // Add the public key generated from the console here.
    getToken(messaging, {vapidKey: "BKagOny0KF_2pCJQ3m....moL0ewzQ8rZu"});

Access the registration token

When you need to retrieve the current registration token for an app instance, first request notification permissions from the user with Notification.requestPermission(). When called as shown, this returns a token if permission is granted or rejects the promise if denied:

    function requestPermission() {
      console.log('Requesting permission...');
      Notification.requestPermission().then((permission) => {
        if (permission === 'granted') {
          console.log('Notification permission granted.');

