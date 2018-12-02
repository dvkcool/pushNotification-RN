

# Adding Push Notifications in React Native with custom sounds
I was working on a project, and then I had this crazy idea of push notifications with custom sound. Now, first task was to implement a simple push notification and then add custom sound to it. 
Then I thought of writing a guide for the same, so here it is: 


# PART 1: Adding Simple push notification
Steps Involved:
1. Adding react-native-push-notification library.
`yarn add react-native-push-notification`
2. Linking the library
`react-native link react-native-push-notification`
3. iOS configuration:
Add the following to your Project: `node_modules/react-native/Libraries/PushNotificationIOS/RCTPushNotification.xcodeproj`
Add the following to Link Binary With Libraries: libRCTPushNotification.a

Finally, to enable support for notification and register events you need to augment your AppDelegate.
At the top of your AppDelegate.m:
`#import <React/RCTPushNotificationManager.h>`
And then in your AppDelegate implementation add the following:
```sh
// Required to register for notifications
 - (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings
 {
  [RCTPushNotificationManager didRegisterUserNotificationSettings:notificationSettings];
 }
 // Required for the register event.
 - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
 {
  [RCTPushNotificationManager didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
 }
 // Required for the notification event. You must call the completion handler after handling the remote notification.
 - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
                                                        fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
 {
   [RCTPushNotificationManager didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
 }
 // Required for the registrationError event.
 - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
 {
  [RCTPushNotificationManager didFailToRegisterForRemoteNotificationsWithError:error];
 }
 // Required for the localNotification event.
 - (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
 {
  [RCTPushNotificationManager didReceiveLocalNotification:notification];
 }
 ```


4. Android Configuration:
In your android/build.gradle
```sh
ext {
    googlePlayServicesVersion = "<Your play services version>" // default: "+"
    firebaseVersion = "<Your Firebase version>" // default: "+"
    // Other settings
    compileSdkVersion = <Your compile SDK version> // default: 23
    buildToolsVersion = "<Your build tools version>" // default: "23.0.1"
    targetSdkVersion = <Your target SDK version> // default: 23
    supportLibVersion = "<Your support lib version>" // default: 23.1.1
}
```

NOTE: localNotification() works without changes in the application part, while localNotificationSchedule() only works with these changes:
In your AndroidManifest.xml
Be sure to replace `${applicationId}` with your android Application Id, its typically of the form com.appName
```sh
.....
    <!-- < Only if you're using GCM or localNotificationSchedule() > -->
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <permission
        android:name="${applicationId}.permission.C2D_MESSAGE"
        android:protectionLevel="signature" />
    <uses-permission android:name="${applicationId}.permission.C2D_MESSAGE" />
    <!-- < Only if you're using GCM or localNotificationSchedule() > -->
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <application ....>
        <meta-data  android:name="com.dieam.reactnativepushnotification.notification_channel_name"
                android:value="YOUR NOTIFICATION CHANNEL NAME"/>
        <meta-data  android:name="com.dieam.reactnativepushnotification.notification_channel_description"
                    android:value="YOUR NOTIFICATION CHANNEL DESCRIPTION"/>
        <!-- Change the resource name to your App's accent color - or any other color you want -->
        <meta-data  android:name="com.dieam.reactnativepushnotification.notification_color"
                    android:resource="@android:color/white"/>
        <!-- < Only if you're using GCM or localNotificationSchedule() > -->
        <receiver
            android:name="com.google.android.gms.gcm.GcmReceiver"
            android:exported="true"
            android:permission="com.google.android.c2dm.permission.SEND" >
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="${applicationId}" />
            </intent-filter>
        </receiver>
        <!-- < Only if you're using GCM or localNotificationSchedule() > -->
        <receiver android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationPublisher" />
        <receiver android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationBootEventReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
        <service android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationRegistrationService"/>
        <!-- < Only if you're using GCM or localNotificationSchedule() > -->
        <service
            android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationListenerServiceGcm"
            android:exported="false" >
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            </intent-filter>
        </service>
        <!-- </ Only if you're using GCM or localNotificationSchedule() > -->
        <!-- < Else > -->
        <service
            android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationListenerService"
            android:exported="false" >
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
        <!-- </Else> -->
     .....
```

In `android/app/src/main/res/values/colors.xml` (Create the file if it doesn't exist).
```sh
<resources>
    <color name="white">#FFF</color>
</resources>
```


Manually register module in MainApplication.java (if you did not use react-native link):
```sh
import com.dieam.reactnativepushnotification.ReactNativePushNotificationPackage;  // <--- Import Package
public class MainApplication extends Application implements ReactApplication {
  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
      @Override
      protected boolean getUseDeveloperSupport() {
        return BuildConfig.DEBUG;
      }
      @Override
      protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
          new ReactNativePushNotificationPackage() // <---- Add the Package
      );
    }
  };
  ....
}
```



I am thankful to Spencer Carli for a wonderful video tutorial for simple push notifications in React native.(Source: https://medium.com/differential/how-to-setup-push-notifications-in-react-native-ios-android-30ea0131355e)



Now the hard part is over, moving on to the easier part.


---

## Part 2: Adding Custom Sound
Modify the PushNotification call in handleAppStateChange(appState) function as follows:
```sh
PushNotification.localNotificationSchedule({
message: "My Notification Message",
date,
soundName: "rush"
});
```
Where `rush` is the custom sound name.
soundName – Value of 'default' plays the default sound.
### Where to place the custom tone??
1. In android, add your custom sound file to [project_root]/android/app/src/main/res/raw
2. In iOS, add your custom sound file to the project Resources in xCode.

### Full source code: 
https://github.com/dvkcool/pushNotification-RN

### References:
https://github.com/zo0r/react-native-push-notification
https://medium.com/differential/how-to-setup-push-notifications-in-react-native-ios-android-30ea0131355e



___________________________________________________________________________________________________

Happy Coding
---
Divyanshu Kumar
---
