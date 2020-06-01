---
layout: post
title: Push Notifications in Android
category: Introduction
comments: true
---

# Push Notifications for Android

Push notification is the ability to push newly available information to a client without the client having to poll for information, therefore potentially saving resources like CPU & Battery and achieving real-time messaging.

## Android

There is an Android Transport Layer (Plaform-level message transport) which routes information delivered to the target device and handles it. This is provided by Google Play Services.

This channel is currently always open on GMS-devices and is basically a BUS for push notifications. Google apps like Gmail and Calendar use this service for push sync. The fact that there are already apps in your phone using this service means that you can freely integrate Google's push service with almost no impact on device performance. The integration is currently done with Firebase Clouse Messaging - FCM (GCM is deprecated) and it basically looks like this: 

![](/assets/images/firebase-arch.png =600x330)

From Firebase Clouse Messaging architecture documentation.

## Firebase Cloud Messaging

FCM is a messaging solution that lets you reliably send messages for free.

"*Using FCM, you can notify a client app that new email or other data is available to sync. You can send notification messages to drive user re-engagement and retention. For use cases such as instant messaging, a message can transfer a payload of up to 4KB to a client app."*

It is an easy to integrate solution and most services that rely on push notifications have FCM support (take Intercom as an example: [https://developers.intercom.com/installing-intercom/docs/about-the-sdk-android](https://developers.intercom.com/installing-intercom/docs/about-the-sdk-android)).

The big factor here is that FCM is tied to the Google Play Services dependency and that comes with one big advantage and one potentially big disadvantage, depending on where globally you want to send notifications to. 

The advantage is reliability, devices with Google Services are very tied to the Android OS Framework and have privilieges that third-party solutions don't have or have to workaround those limitations. Which means that FCM, which is a Google solution may comunicate directly with Google Play Services (often found with system level privilieges) to guarantee that the data is delivered to the device and then re-routed to the application.

The disadvantage is that being a Google solution a proprietary solution makes it unavailable for third-party players and locations like China where Google servers may be blocked. If this is your target audience, this might have a big impact on your app.

## Flurry, Parse & OneSignal

Push Notifications Services like Flurry (Yahoo), Parse (Facebook) and OneSignal seem to use FCM backend, which imply registering in Firebase console with a Google account.

OneSignal github issue referencing push notifications in Android in China ([https://github.com/OneSignal/OneSignal-iOS-SDK/issues/227](https://github.com/OneSignal/OneSignal-iOS-SDK/issues/227#issuecomment-296530161))

## Web Push's & Firefox

Websites can install a Service Worker, a background web page with a limited set of functionality, that can subscribe to the push service. The website can then send a push message through Web Push service to your browser, which can process that message and display a notification on your screen.

Web push notifications are browser dependent and each broswer has its own implementation. "*On Firefox for desktop, the push service is operated by Mozilla. **Firefox for Android uses** a combination of the Mozilla Web Push service and Google’s Cloud Messaging platform to deliver notifications to Firefox for Android. In both cases, push messages are encrypted per the IETF spec, and only your copy of Firefox can decipher them. The encrypted messages are stored on the server until they are delivered or expire."*

## Third Party Alternatives

### Pushy.me

Alternatives to FCM are not in demand but currently there seems to be at least Pushy (pushy.me) which completely replaces FCM and does not depend on GMS. They use MQTT ([https://en.wikipedia.org/wiki/MQTT](https://en.wikipedia.org/wiki/MQTT)) which is lightweight protocol alternative to WebSockets and XMPP (regarding push notifications) leaving a small-footprint, low bandwidth that minimizes battery usage and network traffic. This is ideal in a mobile environment. Pushy is not a free service.

Pushy has a good demo page for testing: [https://pushy.me/docs/resources/demo](https://pushy.me/docs/resources/demo) where we can install an apk and send a push from that page. 

App deals well with push notifications in a Pixel 3 with both app process alive and app killed. Force stopping the app will prevent any further notifications to be delivered until app opens again (also shows the notification when app opens - which must be when the connection is remade).

Pushy does have a fallback delivery method using FCM for improved reliability, which implies it has issues in certain environments.

### JPush (China)

Jpush ([https://www.jpush.cn/](https://www.jpush.cn/)) is a third-party push system that operates in China. [https://docs.jiguang.cn/en/jpush/client/Android/android_sdk/](https://docs.jiguang.cn/en/jpush/client/Android/android_sdk/), They say they have 600M push notifications being sent every day and this is a solution to solve the market gap that Google being blocked in China left open.

I'm assuming here that Jpush is basically a set of custom foreground/background services, but its actually a library with a lot of work on it which can be good to use or to learn from if you have a small team and are dealing with this complex feature. The only issue is that it is chinese, documentation has an English page but there are parts that are not and have to use translator - which actually works good. 

## DIY

- *Custom foreground service using SSLSocket in Java combined with RabbitMQ on the backend. Works instantly and wonderfully.*

    I'm assuming here that the RabbitMQ can be replaced by MQTT and that the foreground service might have issues being efficient, also needing something like a notification to be shown (like Gmail sync notification). A very good explanation on Foreground service limitations can be found here: [https://stackoverflow.com/a/14293528/5177335](https://stackoverflow.com/a/14293528/5177335). Push notifications being a critical service might be hard to maintain but you can achieve efficiency (real-time) using this method. 

- Polling notifications: would basically be a background service brought up from time to time with the AlarmManager (or WorkManager nowadays) to poll the server for new information and sending it to the device..

    This has the advantage of being simple to implement but not very efficient. A dead app can't do background task (process dies) so it has to relly on secondary services to do work. These are usally provided by a Service, which can live in it's own process. Using the Alarm Manager (Work Manager) to schedule a task that fires a service can make this method eficient, depending on the polling interval. Polling too much will also have a negative impact in your battery and bandwidth, aswell as server load (depending in the number of clients). 

## Conclusion

Implementing Push Notifications for your app can be a very simple task if you are happy with it just working on an optimal environment (Google devices with FCM) or really painful if it's a critical service that you need to maintain and to target a global userbase, which will imply Non-Google devices and China. Most of Push Notification providers use FCM underneath and there are only a handful of them which don't, making information on how to solve this problem more scarce.