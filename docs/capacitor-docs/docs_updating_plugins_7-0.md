Skip to main content
An **OutSystems** Company →
Version: v7
On this page
Plugin's `success()` and `error()` methods have been removed, use `resolve()` and `reject()` instead.
Some deprecated plugin definitions and methods such as `registerWebPlugin` have been removed, check the Capacitor 3 plugins upgrade guide for more information.
Capacitor's helper properties `platform` and `isNative` have been removed, use `getPlatform()` and `isNativePlatform()` methods instead.
## Android​
`BridgeFragment` class has been removed, if a plugin was using that class to present a `Fragment`, they'll have to create their own version of it.
## iOS​
### Add SPM support​
Capacitor 6 added experimental SPM support, you can add support for your plugin following Converting existing plugins to SPM
# Updating Capacitor to 7.0 in your plugin
## Using @capacitor/plugin-migration-v6-to-v7​
From the plugin folder, run `npx @capacitor/plugin-migration-v6-to-v7@latest` and it will perform all the file changes automatically.
## Updating the files manually​
### Updating npm dependencies​
#### Updating Capacitor dependencies​
Update `@capacitor/cli`, `@capacitor/core`, `@capacitor/android` and `@capacitor/ios` in devDependencies to `^7.0.0` version. Update `@capacitor/core` in peerDependencies to `>=7.0.0` version.
#### Updating eslint dependencies​
Update `@ionic/eslint-config` to `^0.4.0` and `eslint` to `^8.57.0`.
### Updating swiftlint dependencies​
Update `@ionic/swiftlint-config` and `swiftlint` to `^2.0.0`.
#### Update prettier dependencies​
Update `@ionic/prettier-config` to `^4.0.0`, `prettier` to `^3.4.2` and `prettier-plugin-java` to `^2.6.6`. Then update `prettier` npm script to add `--plugin=prettier-plugin-java`.
```
-  "prettier": "prettier \"**/*.{css,html,ts,js,java}\"",+  "prettier": "prettier \"**/*.{css,html,ts,js,java}\" --plugin=prettier-plugin-java",
```

Finally, you can remove the entries of `.prettierignore` that are already in `.gitignore` as from Prettier 3.0.0, it ignores files in `.gitignore` file by default. If all the entries of `.prettierignore` are already in `.gitignore`, you can remove the `.prettierignore` file entirely.
#### Update rollup​
Update `rollup` to `^4.30.1`. Then update rename the `rollup.config.js` file to `rollup.config.mjs`. Finally, update the build script to use the new file extension:
```
-  "build": "npm run clean && npm run docgen && tsc && rollup -c rollup.config.js",+  "build": "npm run clean && npm run docgen && tsc && rollup -c rollup.config.mjs",
```

#### Update other dependencies​
Update `rimraf` to `^6.0.1` and `@capacitor/docgen` to `^0.3.0`.
### Update Android Plugin Variables​
In your `build.gradle` file, update the following package version minimums:
```
ext {  junitVersion = project.hasProperty('junitVersion') ? rootProject.ext.junitVersion : '4.13.2'-  androidxAppCompatVersion = project.hasProperty('androidxAppCompatVersion') ? rootProject.ext.androidxAppCompatVersion : '1.6.1'+  androidxAppCompatVersion = project.hasProperty('androidxAppCompatVersion') ? rootProject.ext.androidxAppCompatVersion : '1.7.0'-  androidxJunitVersion = project.hasProperty('androidxJunitVersion') ? rootProject.ext.androidxJunitVersion : '1.1.5'+  androidxJunitVersion = project.hasProperty('androidxJunitVersion') ? rootProject.ext.androidxJunitVersion : '1.2.1'-  androidxEspressoCoreVersion = project.hasProperty('androidxEspressoCoreVersion') ? rootProject.ext.androidxEspressoCoreVersion : '3.5.1'+  androidxEspressoCoreVersion = project.hasProperty('androidxEspressoCoreVersion') ? rootProject.ext.androidxEspressoCoreVersion : '3.6.1'}
```

### Update targetSDK / compileSDK to 35 and minSDK to 23​
```
# build.gradleandroid {-  compileSdk project.hasProperty('compileSdkVersion') ? rootProject.ext.compileSdkVersion : 34+  compileSdk project.hasProperty('compileSdkVersion') ? rootProject.ext.compileSdkVersion : 35defaultConfig {-    minSdkVersion project.hasProperty('minSdkVersion') ? rootProject.ext.minSdkVersion : 22+    minSdkVersion project.hasProperty('minSdkVersion') ? rootProject.ext.minSdkVersion : 23-    targetSdkVersion project.hasProperty('targetSdkVersion') ? rootProject.ext.targetSdkVersion : 34+    targetSdkVersion project.hasProperty('targetSdkVersion') ? rootProject.ext.targetSdkVersion : 35...  }}
```

### Update gradle plugin to 8.7.2​
```
  dependencies {-    classpath 'com.android.tools.build:gradle:8.2.1'+    classpath 'com.android.tools.build:gradle:8.7.2'  }
```

### Update gradle wrapper to 8.11.1​
```
# gradle-wrapper.propertiesdistributionBase=GRADLE_USER_HOMEdistributionPath=wrapper/dists- distributionUrl=https\://services.gradle.org/distributions/gradle-8.2.1-all.zip+ distributionUrl=https\://services.gradle.org/distributions/gradle-8.11.1-all.zipzipStoreBase=GRADLE_USER_HOMEzipStorePath=wrapper/dists
```

### Update to Java 21​
```
# build.gradlecompileOptions {-  sourceCompatibility JavaVersion.VERSION_17+  sourceCompatibility JavaVersion.VERSION_21-  targetCompatibility JavaVersion.VERSION_17+  targetCompatibility JavaVersion.VERSION_21}
```

### Update kotlin_version​
If your plugin uses kotlin, update the default `kotlin_version`
```
# build.gradlebuildscript {-  ext.kotlin_version = project.hasProperty("kotlin_version") ? rootProject.ext.kotlin_version : '1.9.10'+  ext.kotlin_version = project.hasProperty("kotlin_version") ? rootProject.ext.kotlin_version : '1.9.25'  repositories {
```

### Raise iOS Deployment Target to 14​
Update your plugin's .podspec file
```
- s.ios.deployment_target = '13.0'+ s.ios.deployment_target = '14.0'
```

#### SPM compatible plugins​
Update `Package.swift` file
```
-  platforms: [.iOS(.v13)],+  platforms: [.iOS(.v14)],
```

#### Plugins with old structure​
Do the following for your Xcode project: select the **Project** within the project editor and open the **Build Settings** tab. Under the **Deployment** section, change **iOS Deployment Target** to **iOS 14.0**. Repeat the same steps for any app **Targets**.
Then, open `ios/Podfile` and update the iOS version to 14.0:
```
-platform :ios, '13.0'+platform :ios, '14.0'
```

### Update Capacitor SPM dependency​
In SPM compatible plugins, update `Package.swift` file to use a fixed version instead of main branch.
```
  dependencies: [-    .package(url: "https://github.com/ionic-team/capacitor-swift-pm.git", branch: "main")+    .package(url: "https://github.com/ionic-team/capacitor-swift-pm.git", from: "7.0.0")  ],
```

## Contents
  * Android
  * iOS
    * Add SPM support
  * Using @capacitor/plugin-migration-v6-to-v7
  * Updating the files manually
    * Updating npm dependencies
    * Updating swiftlint dependencies
    * Update Android Plugin Variables
    * Update targetSDK / compileSDK to 35 and minSDK to 23
    * Update gradle plugin to 8.7.2
    * Update gradle wrapper to 8.11.1
    * Update to Java 21
    * Update kotlin_version
    * Raise iOS Deployment Target to 14
    * Update Capacitor SPM dependency


Edit this page![](https://images.prismic.io/ionicframeworkcom/d3d3f7a3-023b-4cdf-93af-84674f623818_portals+ad.png?auto=compress,format&rect=0,0,280,200&w=280&h=200)
Micro Frontends for any React Native, Android, or iOS mobile apps.
![](https://cdn.bizible.com/ipv?_biz_r=&_biz_h=802059049&_biz_u=ed6d98ad223740ddbf99774ce8c4ab02&_biz_l=https%3A%2F%2Fcapacitorjs.com%2Fdocs%2Fupdating%2Fplugins%2F7-0&_biz_t=1739811917867&_biz_i=Capacitor%20Documentation&_biz_n=20&rnd=934530&cdn_o=a&_biz_z=1739811917867)
