# SDK Integration
Follow this guide to install and include the Geo SDK module for React Native
into your project.

## Table of Contents

1. [Reference versions](#reference-versions)
    1. [Node.js](#node.js)
    2. [React-native-cli](#react-native-cli)
    3. [React-native](#react-native)
    4. [Xcode](#xcode)
    5. [Framework Geouniq iOS](#framework-geouniq-ios)
2. [Project Configuration](#project-configuration)
    1. [Installation](#installation)
    2. [Linking Native Code on iOS](#linking-native-code-on-ios)
    3. [Linking Native Code on Android](#linking-native-code-on-android)
    4. [Manual link](#manual-link)
    5. [iOS Settings](#ios-settings)
    6. [Android Settings](#android-settings)
3. [Basic Operations](#basic-operations)
    1. [Usage](#usage)
    2. [Enabling/Disabling the SDK](#enablingdisabling-the-sdk)
    3. [Handling blocking issues (Android only)](#handling-blocking-issues-android-only)
    4. [Handling User consent](#handling-user-consent)
    5. [Getting the device ID](#getting-the-device-id)


## Reference Versions

### Node.js

v12.18.4

### React-native-cli

2.0.1

### React-native

0.63.3

### Expo cli

3.27.13

### Xcode

12.0.1

### Framework GeoUniq iOS

1.6.1 - [Changelog](/sdk/integration/changeLogIos.md)

## Project Configuration

### Installation
Obtain the mobile key for the platforms (Android and iOS) from GeoUniq
Console (https://console.geouniq.com)

Follow the next steps to add the module to the project

* Open the terminal
* Move into the folder of your application
* Type `npm install react-native-geouniq`

The module will be installed both for Android and iOS.

### Linking Native Code on iOS

* Into the folder of your application run: `npx pod-install`
* Once this is complete, re-build the app binary to start using your new library: `npx react-native run-ios`

### Linking Native Code on Android

* React Native uses Gradle to manage Android project dependencies. After you install a library with native dependencies, you will need to re-build the app binary to use your new library: `npx react-native run-android`

### iOS Settings

1. Follow the instruction `Project Settings` [here](https://gitlab.com/geouniq/documentation/blob/master/sdk/integration/ios.md#manual)
2. Add the following code to the AppDelegate

```objc
//Objective C
/* ------ AppDelegate.m ------ */

//importing the framework
#import <GeoUniq/GeoUniq-Swift.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
...
...
    [super application:application didFinishLaunchingWithOptions:launchOptions];
  
    (void)[GeoUniq sharedInstance];

    return YES;
}
```

### Android Settings

##### Providing the mobile key
Open the file
`../android/app/build.gradle`

and put the following line
```gradle
defaultConfig {
    //...
    it.resValue("string", "geouniq_mobile_key", "YOUR_MOBILE_KEY")
}
```

## Basic operations
* [Usage](#usage)
* [Enabling/Disabling the SDK](#enablingdisabling-the-sdk)
* [Handling blocking issues](#handling-blocking-issues)
* [Handling User consent](#handling-user-consent)
* [Getting the device ID](#getting-the-device-id)

### Usage
Add this import in your javascript file
```javascript
import Geouniq from 'react-native-geouniq';
```

### Enabling/Disabling the SDK

The SDK can be enabled and disabled at runtime.
To make Geo SDK start, you need to enable it by calling the method `Geouniq.enable` at least once.

Once enabled, the SDK will not stop until you disable it by calling `Geouniq.disable`. That is, it will keep performing automatic operations, 
such as tracking the device position, even after a device reboot or an update of the app.
Disabling the SDK at runtime is useful if you want the SDK to stop completely. For example, you could remotely control a configuration parameter of your 
app to stop the SDK for all or some of your installations.

> If you simply don't want Geouniq to keep collecting location data for a specific User, you can do that without completely disabling the 
SDK (see [Handle User consent](#handle-user-consent)). This way you can still exploit the mobile-side functionalities that the SDK provides without having 
Geouniq collecting data for the specific user


```javascript
function enableGeouniq() {
    Geouniq.enable(
        () => { console.log("Geouniq enabled"); }
    )
}
```
```javascript
function disableGeouniq() {
    Geouniq.disable(
        () => { console.log("Geouniq disabled"); }
    )
}
```

### Handling blocking issues (Android only)

For the SDK to be able to accomplish its main task, that is tracking the device position, the following requirements must be met:

* The location permission must be granted to the app
* The location functionality must be enabled on the device
* Google Play Services must be installed on the device (almost always met)

The SDK provides a simple way for checking and solve the related issues.
Solving an issue generally implies showing an alert to the User. Depending on the specific requirement, a different alert will be shown.

```javascript
function solveIssues() {
    Geouniq.solveIssues(
        (error) => { console.log("Received callback with error: " + error); },
        () => { console.log("Received success callback"); }
    )
}
```

### Handling User consent

The ability of the SDK to track the device location does not give Geouniq the permission to collect user data.
According to GDPR regulation, you should request the User the consent to collect location data to Geouniq platform.
There are different consents that User can be grant, if at least one consent is granted, Geouniq have the permission to collect user data.

> NOTE: if your app is not under GDPR regulation, you can avoid request the consent to the user and directly inform the SDK that it has the permission to collect data on Geouniq platform. 
This can be done as explained below in [Setting consent explicitely](#setting-the-user-consent-explicitely)

The SDK provides a simple way to handle user consent through the `showPrivacyPolicyAndSet()` method, as shown in the example below.
When this method is called, it will show a dialog to the User requesting the consent to collect data on GeoUniq platform. The privacy policy can also be seen by the User.
If the User accept the whole privacy policy or gives at least a consent, then the SDK will actually start sending data on GeoUniq platform. 
Otherwise, it will keep performing all the other automatic operations without sending any information on GeoUniq platform.
The success function return a map that represents the user choise for each consent.

```javascript
function showPrivacyPolicyAndSet() {
    Geouniq.showPrivacyPolicyAndSet(
        (error) => { console.log("Received callback with error: " + error); },
        (consentsMap) => {
            console.log("Received success callback");
            console.log("Customization and adtargeting: " + consentsMap["CUSTOMIZATION_AND_ADTARGETING"]);
            console.log("Analysis: " + consentsMap["ANALYSIS"]);
        }
    )
}
```

##### Setting the user consent explicitely

The SDK also provides `Geouniq.setPrivacyConsent(String consentItem, boolean isGranted)` to explicitely set a consent.
If there is at least a consent granted, Geouniq is allowed to collect user data.
This is particularly useful in the following cases.

If your app is not under GDPR regulation, then you can call this method with any one `ConsentItem` and the parameter `isGranted` equal to `true` to let the SDK collect data without requesting any consent to the User.

If your app is under GDPR regulation, then you should provide the User a way to remove a consent from your app's settings.
If the User removes a consent, then you should inform the SDK by calling the method above with the releated `ConsentItem` element and parameter `isGranted` equal to `false`.
By doing so, if there isn't at least another consent granted, the SDK will stop sending data to GeoUniq platform immediately.

> Recognized values for consents are CUSTOMIZATION_AND_ADTARGETING and ANALYSIS.

```javascript
function setPrivacyConsent(consent, value) {
    Geouniq.setPrivacyConsent(
        consent,
        value,
        (error) => { console.log("Received callback with error: " + error); },
        () => { console.log("Received success callback, consent " + consent + " set to " + value); }
    )
}
```

##### Getting the consent status
The method `Geouniq.getPrivacyConsentsMap()` allows to know the status of the User consents.
This is the result of the User choise after showing it the privacy policy dialog, or the value explicitely set through the `Geouniq.setPrivacyConsent(ConsentItem consentItem, boolean isGranted)` method.
To check a single consent, there is the method `boolean getPrivacyConsent(ConsentItem)` that return `true`if the consent is granted, `false` otherwise

```javascript
function getPrivacyConsentsMap() {
    Geouniq.getPrivacyConsentsMap(
        (error) => { console.log("getPrivacyConsentsMap error callback, error: " + error); },
        (consentsMap) => {
            console.log("getPrivacyConsentsMap success callback");
            console.log("Customization and adtargeting: " + consentsMap["CUSTOMIZATION_AND_ADTARGETING"]);
            console.log("Analysis: " + consentsMap["ANALYSIS"]);
        }
    )
}
```

### Getting the Device ID

As soon as the SDK starts for the first time, a [Device ID](/service-architecture.md#device-registration-device-ids-and-device-base) is assigned to 
that specific installation of your app
The SDK provides a way to obtain the Device ID as soon as it is assigned, as shown in the example below. 

```javascript
function setGeouniqDeviceIdListener() {
    Geouniq.setDeviceIdListener(
        (deviceId) => { console.log("Received device ID callback with Id: " + deviceId); }
    )
}
```

You can also get the Device ID calling `Geouniq.getDeviceId` and check if is available with method `Geouniq.isDeviceIdAvailable`  
