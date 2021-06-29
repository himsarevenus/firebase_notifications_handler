# FirebaseNotificationsHandler For Flutter
[![pub package](https://img.shields.io/pub/v/firebase_notifications_handler.svg)](https://pub.dev/packages/firebase_notifications_handler)
[![likes](https://badges.bar/firebase_notifications_handler/likes)](https://pub.dev/packages/firebase_notifications_handler/score)
[![popularity](https://badges.bar/firebase_notifications_handler/popularity)](https://pub.dev/packages/firebase_notifications_handler/score)
[![pub points](https://badges.bar/firebase_notifications_handler/pub%20points)](https://pub.dev/packages/firebase_notifications_handler/score)

* Simple notifications handler which provides callbacks like onTap which really make it easy to handle notification taps and a lot more.

## Screenshots
<img src="https://user-images.githubusercontent.com/56810766/123861270-9a9e4800-d944-11eb-8c04-8fd3e9557876.png" height=600/>&nbsp;&nbsp;

## Getting Started
<b>Step 1</b>: Before you can add Firebase to your app, you need to create a Firebase project to connect to your application.
Visit [`Understand Firebase Projects`](https://firebase.google.com/docs/projects/learn-more) to learn more about Firebase projects.

<b>Step 2</b>: To use Firebase in your app, you need to register your app with your Firebase project.
Registering your app is often called "adding" your app to your project.

Also, register a web app if using on the web.
Follow on the screen instructions to initilalize the project.

Add the latest version 'firebase-messaging' CDN from [here](https://firebase.google.com/docs/web/setup#available-libraries) in index.html.
(Tested on version 8.6.1)

<b> Step 3</b>: Add a Firebase configuration file and the SDK's. (google-services)

<b> Step 4</b>: Lastly, add [`firebase_core`](https://pub.dev/packages/firebase_core) as a dependency in your pubspec.yaml file.
and call `Firebase.initializeApp()` in the `main` method as shown:
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}
```

### Android

Add the default channel in AndroidManifest. Pass the same in the channelId parameter in the
`FirebaseNotificationsHandler` widget to enable custom sounds.

```xml
<meta-data
    android:name="com.google.firebase.messaging.default_notification_channel_id"
    android:value="<same as channelId in FirebaseNotificationsHandler>" />
```

Also, add this intent-filter in AndroidManifest

```xml
<intent-filter>
    <action android:name="FLUTTER_NOTIFICATION_CLICK" />
    <category android:name="android.intent.category.DEFAULT" />
</intent-filter>
```

#### Adding custom notification sounds in Android
For custom audio, add the audio file in android/app/src/main/res/raw/____.mp3

### Web
Provide the vapidKey in FirebaseNotificationsHandler from the cloud messaging settings by generating
a new Web push certificate

Add this script tag in index.html after adding the firebase config script
```html
<script>
if ("serviceWorker" in navigator) {
  window.addEventListener("load", function () {
    // navigator.serviceWorker.register("/flutter_service_worker.js");
    navigator.serviceWorker.register("/firebase-messaging-sw.js");
  });
}
</script>
```

Now, finally create a file `firebase-messaging-sw.js` in the `web` folder itself
and paste the following contents. Add your own firebase app config here.

```js
importScripts("https://www.gstatic.com/firebasejs/7.15.5/firebase-app.js");
importScripts("https://www.gstatic.com/firebasejs/7.15.5/firebase-messaging.js");

firebase.initializeApp(
    // YOUR FIREBASE CONFIG MAP HERE
);

const messaging = firebase.messaging();
messaging.setBackgroundMessageHandler(function (payload) {
    const promiseChain = clients
        .matchAll({
            type: "window",
            includeUncontrolled: true
        })
        .then(windowClients => {
            for (let i = 0; i < windowClients.length; i++) {
                const windowClient = windowClients[i];
                windowClient.postMessage(payload);
            }
        })
        .then(() => {
            return registration.showNotification("New Message");
        });
    return promiseChain;
});
self.addEventListener('notificationclick', function (event) {
    console.log('notification received: ', event)
});
```

## Usage

To use this plugin, add [`firebase_notifications_handler`](https://pub.dev/packages/firebase_notifications_handler) as a dependency in your pubspec.yaml file.

```yaml
  dependencies:
    flutter:
      sdk: flutter
    firebase_notifications_handler:
```

First and foremost, import the widget.
```dart
import 'package:firebase_notifications_handler/firebase_notifications_handler.dart';
```

Wrap the `MaterialApp` with `FirebaseNotificationsHandler` to enable your application to receive notifications.
```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FirebaseNotificationsHandler(
      child: MaterialApp(
        debugShowCheckedModeBanner: false,
        home: HomeScreen(),
      ),
    );
  }
}
```

There are multiple parameters that can be passed to the widget, some of them are shown.
```dart
FirebaseNotificationsHandler(
    onFCMTokenInitialize: (_, token) => fcmToken = token,
    onFCMTokenUpdate: (_, token) {
        fcmToken = token;
        // await User.updateFCM(token);
    },
    onTap: (navigatorState, appState, payload) {
        print("Notification tapped with $appState & payload $payload");

        final context = navigatorState.currentContext!;
        navigatorState.currentState!.pushNamed('newRouteName');
        // OR
        Navigator.pushNamed(context, 'newRouteName');
    },
    channelId: 'ChannelId',
    enableLogs: true,
),
```

You can check the remaining parameters [here](https://github.com/rithik-dev/firebase_notifications_handler/blob/master/lib/src/widget.dart).
They are fully documented and won't face an issue while using them

### Send notification using REST API

To send FCM notification using REST API:

Make a `POST` request @[`https://fcm.googleapis.com/fcm/send`](https://fcm.googleapis.com/fcm/send)

Also, add 2 headers:
```
Content-Type: application/json
Authorization: key=<SERVER_KEY_FROM_FIREBASE_CLOUD_MESSAGING>
```

You can find the server key from the cloud messaging settings in the firebase console.

The body is framed as follows:
```json
{
      "to": "<FCM_TOKEN_HERE>",
      "registration_ids": [],

      "notification": {
            "title": "Title here",
            "body": "Body here"
      },
      "data": {
            "click_action":"FLUTTER_NOTIFICATION_CLICK"
      }
}
```
You can pass all the fcm tokens in the "registration_ids" list if there are multiple users
or only pass one fcm token in the "to" parameter for single user.

Add all the rest of the payload data in "data" field which will be provided in the `onTap` callback.

#### Sample Usage
```dart
import 'package:flutter/material.dart';
import 'package:firebase_notifications_handler/firebase_notifications_handler.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(_MainApp());
}

String? fcmToken;

class _MainApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FirebaseNotificationsHandler(
      onOpenNotificationArrive: (_, payload) {
        print("Notification received while app is open with payload $payload");
      },
      onTap: (navigatorState, appState, payload) {
        print("Notification tapped with $appState & payload $payload");

        final context = navigatorState.currentContext!;
        navigatorState.currentState!.pushNamed('newRouteName');
        // OR
        Navigator.pushNamed(context, 'newRouteName');
      },
      onFCMTokenInitialize: (_, token) => fcmToken = token,
      onFCMTokenUpdate: (_, token) {
        fcmToken = token;
        // await User.updateFCM(token);
      },
      child: MaterialApp(
        navigatorKey: FirebaseNotificationsHandler.navigatorKey,
      ),
    );
  }
}

```

### Created & Maintained By `Rithik Bhandari`

* GitHub: [@rithik-dev](https://github.com/rithik-dev)
* LinkedIn: [@rithik-bhandari](https://www.linkedin.com/in/rithik-bhandari/)