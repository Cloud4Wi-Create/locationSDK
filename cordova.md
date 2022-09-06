# SDK Integration
Follow this guide to install and include the Geo SDK plugin for Cordova
into your project.

## Table of Contents

1. [Project Configuration](#project-configuration)
    1. [Installation](#Installation)
    1. [Android](#android)
    1. [iOS](#ios)
1. [Basic Operations](#basic-operations)
    1. [Enabling/Disabling the SDK](#enablingdisabling-the-sdk)
    1. [Handling blocking issues](#handling-blocking-issues)
    1. [Handling User consent](#handling-user-consent)
    1. [Getting the device ID](#getting-the-device-id)

## Project Configuration

### Installation
Obtain the mobile key for the platforms (Android and iOS) from GeoUniq
Console (https://console.geouniq.com)

Be sure to have initialized the platform you want to develop for
before to install the plugin with the command `cordova platform add
PLATFORM_NAME`

Follow the next steps to add the plugin to the project

* Open the terminal
* Move into the folder of your application
* type `cordova plugin add com.geouniq.cordova`

The plugin will be installed both for Android and iOS.

### Android

Open the file
`../platforms/android/app/src/main/res/values/strings.xml`

and add a new string resource with your mobile key
`<string name="geouniq_mobile_key">YOUR MOBILE KEY</string>`

### iOS

Once you install the plugin, follow the instruction `Installation` and `Project Settings` [here](https://gitlab.com/geouniq/documentation/blob/master/sdk/integration/ios.md#manual) on the xcode project created by cordova.


## Basic operations

* [Enabling/Disabling the SDK](#enablingdisabling-the-sdk)
* [Handling blocking issues](#handling-blocking-issues)
* [Handling User consent](#handling-user-consent)
* [Getting the device ID](#getting-the-device-id)

### Enabling/Disabling the SDK

The SDK can be enabled and disabled at runtime.
To make Geo SDK start, you need to enable it by calling the method `GUCordovaPlugin.enable` at least once.

You might do that into the main activity of your app, as in the example below.

Once enabled, the SDK will not stop until you disable it by calling `GUCordovaPlugin.disable`. That is, it will keep performing automatic operations, 
such as tracking the device position, even after a device reboot or an update of the app.
Disabling the SDK at runtime is useful if you want the SDK to stop completely. For example, you could remotely control a configuration parameter of your 
app to stop the SDK for all or some of your installations.

> If you simply don't want Geouniq to keep collecting location data for a specific User, you can do that without completely disabling the 
SDK (see [Handle User consent](#handle-user-consent)). This way you can still exploit the mobile-side functionalities that the SDK provides without having 
Geouniq collecting data for the specific user


```javascript

var app = {
    initialize: function () {
        document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);
    },

    onDeviceReady: function () {
        cordova.plugins.GUCordovaPlugin.enable(
            function () {
                console.log("Reveived enable callback");
            }
        )
};

app.initialize();

```

### Handling blocking issues

For the SDK to be able to accomplish its main task, that is tracking the device position, the following requirements must be met:

* The location permission must be granted to the app
* The location functionality must be enabled on the device
* Google Play Servicece must be installed on the device (almost always met)

The SDK provides a simple way for checking and solve the related issues.
Solving an issue generally implies showing an alert to the User. Depending on the specific requirement, a different alert will be shown.

```javascript

var app = {
    initialize: function () {
        document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);
    },

    onDeviceReady: function () {
        cordova.plugins.GUCordovaPlugin.solveIssues()
    },
};

app.initialize();

```

### Handling User consent

The ability of the SDK to track the device location does not give Geouniq the permission to collect user data.
According to [GDPR](https://ec.europa.eu/commission/priorities/justice-and-fundamental-rights/data-protection/2018-reform-eu-data-protection-rules_en) 
regulation, you should request the User the consent to collect location data on Geouniq platform.

> NOTE: if your app is not under GDPR regulation, you can avoid request the consent to the user and directly inform the SDK that it has the permission 
to collect data on Geouniq platform. This can be done as explained below in [Setting consent explicitely](#setting-the-user-consent-explicitely)

The SDK provides a simple way to handle user consent through the `GUCordovaPlugin.showConsentDialogAndSet`as shown in the example below.
When this method is called, the SDK first checks if it has already obtained the consent. If yes, it does nothing. Otherwise, it will show a dialog to 
the user requesting the consent to collect data on GeoUniq platform. The privacy policy can also be seen by the User.
If the User gives the consent, then the SDK will actually start sending data on GeoUniq platform. Otherwise, it will keep performing all the other automatic
operations without sending any information on GeoUniq platform.

```javascript

var app = {
    initialize: function () {
        document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);
    },

    onDeviceReady: function () {
        cordova.plugins.GUCordovaPlugin.showConsentDialogAndSet(
            function (granted) {
                console.log("Called showConsentDialogAndSet callback, granted status is " + granted);
            }
        )    
    },
};

app.initialize();

```

#### Setting the user consent explicitely

The SDK also provides a method to explicitely set whether the User has given or not the consent.
You can call the `GUCordovaPlugin.setConsentStatus` method to explicitely inform the SDK that it Geouniq is allowed to collect user data.
This is particularly useful in the following cases.

If your app is not under GDPR regulation, then you can call this method with the first method parameter equal to `true` to let the SDK collect data 
without requesting any consent to the User

If your app is under GDPR regulation, then you should provide the User a way to remove the consent from your app's settings.
If the User removes the consent, then you should inform the SDK by calling the method above with the first method parameter equal to `false`.
By doing so, the SDK will stop sending data to GeoUniq platform immediately.

```javascript

var app = {
    initialize: function () {
        document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);
    },

    onDeviceReady: function () {
        cordova.plugins.GUCordovaPlugin.setConsentStatus(
            false,
            function () {
                  console.log("Called setConsentStatus callback");
            }
        )    
    },
};

app.initialize();

```

#### Getting the consent status

The method `GUCordovaPlugin.getConsentStatus` allows to know status of the User consent.
This is the result of the User choise after showing it the consent alert, or the value explicitely set through the `GUCordovaPlugin.setConsentStatus` method.

```javascript

var app = {
    initialize: function () {
        document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);
    },

    onDeviceReady: function () {
        cordova.plugins.GUCordovaPlugin.getConsentStatus(
            function (status) {
                console.log("Called getConsentStatus callback, status: " + status);
            }
        )   
    },
};

app.initialize();

```

## Getting the Device ID

As soon as the SDK starts for the first time, a [Device ID](/service-architecture.md#device-registration-device-ids-and-device-base) is assigned to 
that specific installation of your app
The SDK provides a way to obtain the Device ID as soon as it is assigned, as shown in the example below. 

```javascript

var app = {
    initialize: function () {
        document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);
    },

    onDeviceReady: function () {
         cordova.plugins.GUCordovaPlugin.setDeviceIdListener(
            function (deviceId) {
                console.log("Called setDeviceIdListener callback, deviceId: " + deviceId);
            }
        )   
    },
};

app.initialize();

```

You can also get the Device ID calling `GUCordovaPlugin.getDeviceId` and check if is available with method `GUCordovaPlugin.isDeviceIdAvailable`
