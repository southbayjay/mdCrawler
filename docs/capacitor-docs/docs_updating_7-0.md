Skip to main content
An **OutSystems** Company →
Version: v7
On this page
In this guide, you'll find steps to update your project to the current Capacitor 7 version as well as a list of breaking changes for our official plugins.
## Breaking changes in Capacitor config file​
`bundledWebRuntime` configuration option has been removed. If you had it set it to `false` you can safely remove it. If you had it set to `true` you'll have to use a bundler to bundle `@capacitor/core` code within your app.
`cordova.staticPlugins` configuration option has been removed. If you still have some Cordova plugin that needs to be static it should be updated to use `podspec` tag with `use-framework` attribute instead of using `framework` tag as that's not supported in cordova-ios 7+.
## NodeJS 20+​
Node 18 has reached end of active support on October 18th, 2023. Capacitor 7 requires NodeJS 20 or greater. (Latest LTS version is recommended.)
## Telemetry is now opt-out​
This only affects new users as if you have used any Capacitor command before it will already have a preference saved. Also telemetry does not run in non-interactive environments, such as CI servers, ensuring no data is collected in those scenarios. It can be disabled with `npx cap telemetry off`.
## Using the CLI to Migrate​
Install the `latest` version of the Capacitor CLI to your project:
```
npm i -D @capacitor/cli@latest
```

Once installed, simply run the following to have the CLI handle the migration for you.
```
npx cap migrate
```

If any of the steps for the migration are not able to be completed, additional information will be made available in the output in the terminal. The steps for doing the migration manually are listed out below.
## iOS​
The following guide describes how to upgrade your Capacitor 6 iOS project to Capacitor 7.
### Upgrade Xcode​
Capacitor 7 requires Xcode 16.0+.
### Raise iOS Deployment Target​
Do the following for your Xcode project: select the **Project** within the project editor and open the **Build Settings** tab. Under the **Deployment** section, change **iOS Deployment Target** to **iOS 14.0**. Repeat the same steps for any app **Targets**.
Then, open `ios/App/Podfile` and update the iOS version to 14.0:
```
platform :ios,'14.0'
```

## Android​
The following guide describes how to upgrade your Capacitor 6 Android project to Capacitor 7.
### Upgrade Android Studio​
Capacitor 7 requires Android Studio Ladybug | 2024.2.1 or newer and Java JDK 21. Java 21 ships with Android Studio Ladybug. No additional downloads needed!
Once it's updated, Android Studio can assist with some of the updates related to gradle and moving package into build files. To start, run `Tools -> AGP Upgrade Assistant`.
![APG Upgrade Assistant](https://capacitorjs.com/docs/assets/images/agp-upgrade-assistant-59a6aa5263bea07f9dd39eb8ed537743.png)
### Update Android Project Variables​
In your `variables.gradle` file, update your values to the following new minimums
```
minSdkVersion =23compileSdkVersion =35targetSdkVersion =35androidxActivityVersion ='1.9.2'androidxAppCompatVersion ='1.7.0'androidxCoordinatorLayoutVersion ='1.2.0'androidxCoreVersion ='1.15.0'androidxFragmentVersion ='1.8.4'coreSplashScreenVersion ='1.0.1'androidxWebkitVersion ='1.12.1'junitVersion ='4.13.2'androidxJunitVersion ='1.2.1'androidxEspressoCoreVersion ='3.6.1'cordovaAndroidVersion ='10.1.1'
```

### Update google services plugin​
```
# build.gradle  dependencies {    classpath 'com.android.tools.build:gradle:8.2.1'-    classpath 'com.google.gms:google-services:4.4.0'+    classpath 'com.google.gms:google-services:4.4.2'
```

### Update gradle plugin to 8.7.2​
```
# build.gradle  dependencies {-    classpath 'com.android.tools.build:gradle:8.2.1'+    classpath 'com.android.tools.build:gradle:8.7.2'
```

### Update gradle wrapper to 8.11.1​
```
# gradle-wrapper.propertiesdistributionBase=GRADLE_USER_HOMEdistributionPath=wrapper/dists- distributionUrl=https\://services.gradle.org/distributions/gradle-8.2.1-all.zip+ distributionUrl=https\://services.gradle.org/distributions/gradle-8.11.1-all.zipzipStoreBase=GRADLE_USER_HOMEzipStorePath=wrapper/dists
```

### Update kotlin version​
If your project is using kotlin, update the `kotlin_version` variable to `'1.9.25'`.
### add navigation to configChanges​
This is an optional change to prevent app restarts on some devices when using bluetooth keyboards. Add `navigation` to `configChanges` of the app `activity` in `AndroidManifest.xml`.
```
- android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|smallestScreenSize|screenLayout|uiMode"+ android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|smallestScreenSize|screenLayout|uiMode|navigation"
```

## Plugins​
Plugins have been updated to version 7.0.0, make sure to update them to use latest version.
The following plugin functionality has been modified or removed. Update your code accordingly.
### Action Sheet​
  * `androidxMaterialVersion` variable has been updated to `1.12.0`.


### App​
  * Deprecated type `AppRestoredResult` was removed, use `RestoredListenerEvent`.
  * Deprecated type `AppUrlOpen` was removed, use `URLOpenListenerEvent`.


### Browser​
  * `androidxBrowserVersion` variable has been updated to `1.8.0`.


### Camera​
  * `androidxExifInterfaceVersion` variable has been updated to `1.3.7`.
  * `androidxMaterialVersion` variable has been updated to `1.12.0`.


### Device​
  * `getInfo()` no longer returns `diskFree`, `diskTotal`, `realDiskFree` and `realDiskTotal`, so the `PrivacyInfo.xcprivacy` entries for this plugin can be removed.
  * Deprecated type `DeviceBatteryInfo` was removed, use `BatteryInfo`.
  * Deprecated type `DeviceLanguageCodeResult` was removed, use `GetLanguageCodeResult`.


### Geolocation​
  * `playServicesLocationVersion` variable has been updated to `21.3.0`.


### Haptics​
  * Deprecated type `HapticsImpactOptions` was removed, use `ImpactOptions`.
  * Deprecated type `HapticsNotificationOptions` was removed, use `NotificationOptions`.
  * Deprecated type `HapticsNotificationType` was removed, use `NotificationType`.
  * Deprecated type `HapticsImpactStyle` was removed, use `ImpactStyle`.


### Push Notifications​
  * `firebaseMessagingVersion` variable has been updated to `24.1.0`.


### Share​
  * `androidxCoreVersion` variable has been updated to `1.15.0`.


### Splash Screen​
  * Deprecated type `SplashScreenShowOptions` was removed, use `ShowOptions`.
  * Deprecated type `SplashScreenHideOptions` was removed, use `HideOptions`.


### Status Bar​
  * `setOverlaysWebView()` and `setBackgroundColor()` are now supported on iOS.
  * `androidxCoreVersion` variable has been updated to `1.15.0`.


## Contents
  * Breaking changes in Capacitor config file
  * NodeJS 20+
  * Telemetry is now opt-out
  * Using the CLI to Migrate
  * iOS
    * Upgrade Xcode
    * Raise iOS Deployment Target
  * Android
    * Upgrade Android Studio
    * Update Android Project Variables
    * Update google services plugin
    * Update gradle plugin to 8.7.2
    * Update gradle wrapper to 8.11.1
    * Update kotlin version
    * add navigation to configChanges
  * Plugins
    * Action Sheet
    * App
    * Browser
    * Camera
    * Device
    * Geolocation
    * Haptics
    * Push Notifications
    * Share
    * Splash Screen
    * Status Bar


Edit this page![](https://images.prismic.io/ionicframeworkcom/50ede1c5-d69d-4c9d-bf0d-4c9ab7c14724_doc-ad-appflow.png?auto=compress,format&rect=0,0,280,200&w=280&h=200)
Mobile CI/CD made easy. Build, publish, and update from the cloud.
![](https://cdn.bizible.com/ipv?_biz_r=&_biz_h=802059049&_biz_u=ed6d98ad223740ddbf99774ce8c4ab02&_biz_l=https%3A%2F%2Fcapacitorjs.com%2Fdocs%2Fupdating%2F7-0&_biz_t=1739811917316&_biz_i=Capacitor%20Documentation&_biz_n=19&rnd=646720&cdn_o=a&_biz_z=1739811917316)
