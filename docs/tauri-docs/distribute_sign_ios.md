Skip to content
# iOS Code Signing
Code signing on iOS is required to distribute your application through the official Apple App Store or possibly alternative marketplaces in the European Union and in general to install and execute on end user devices.
## Prerequisites
Code signing on iOS requires enrolling to the Apple Developer program, which at the time of writing costs 99$ per year. You also need an Apple device where you perform the code signing. This is required by the signing process and due to Apple’s Terms and Conditions.
To distribute iOS applications you must have your bundle identifier registered in the App Store Connect, an appropriate iOS code signing certificate and a mobile provisioning profile that links them together and enables the iOS capabilities used by your app. These requirements can be either automatically managed by Xcode or provided manually.
## Automatic Signing
Letting Xcode manage the signing and provisioning for your app is the most convenient way to export your iOS app to be distributed. It automatically registers your bundle identifier, manages iOS capabilities changes, and configures an appropriate certificate based on your export method.
Automatic signing is enabled by default, and uses the account configured in Xcode to authenticate when used on your local machine. To register your account, open the Xcode application and open the Settings page in the `Xcode > Settings` menu, switch to the Accounts tab and click the `+` icon.
To use the automatic signing in CI/CD platforms you must create an App Store Connect API key and define the `APPLE_API_ISSUER`, `APPLE_API_KEY` and `APPLE_API_KEY_PATH` environment variables. Open the App Store Connect’s Users and Access page, select the Integrations tab, click on the Add button and select a name and the Admin access. The `APPLE_API_ISSUER` (Issuer ID) is presented above the keys table, and the `APPLE_API_KEY` is the value on the Key ID column on that table. You also need to download the private key, which can only be done once and is only visible after a page reload (the button is shown on the table row for the newly created key). The private key file path must be set via the `APPLE_API_KEY_PATH` environment variable.
## Manual Signing
To manually sign your iOS app you can provide the certificate and mobile provisioning profile via environment variables:
  * **IOS_CERTIFICATE** : base64 representation of the certificate exported from the Keychain.
  * **IOS_CERTIFICATE_PASSWORD** : password of the certificate set when exporting it from the Keychain.
  * **IOS_MOBILE_PROVISION** : base64 representation of the provisioning profile.


The following sections explain how to get these values.
### Signing Certificate
After enrolling, navigate to the Certificates page to create a new Apple Distribution certificate. Download the new certificate and install it to the macOS Keychain.
To export the certificate key, open the “Keychain Access” app, expand the certificate’s entry, right-click on the key item and select “Export <key-name>” item. Select the path of the exported .p12 file and remember its password.
Run the following `base64` command to convert the certificate to base64 and copy it to the clipboard:
```

base64 -i <path-to-certificate.p12> | pbcopy

```

The value in the clipboard is now the base64 representation of the signing certificate. Save it and use it as the `IOS_CERTIFICATE` environment variable value.
The certificate password must be set to the `IOS_CERTIFICATE_PASSWORD` variable.
### Provisioning Profile
Additionally, you must provide the provisioning profile for your application. In the Identifiers page, create a new App ID and make sure its “Bundle ID” value matches the identifier set in the `identifier` configuration.
Navigate to the Profiles page to create a new provisioning profile. For App Store distribution, it must be an “App Store Connect” profile. Select the appropriate App ID and link the certificate you previously created.
After creating the provisioning profile, download it and run the following `base64` command to convert the profile and copy it to the clipboard:
```

base64 -i <path-to-profile.mobileprovision> | pbcopy

```

The value in the clipboard is now the base64 representation of the provisioning profile. Save it and use it as the `IOS_MOBILE_PROVISION` environment variable value.
Now you can build your iOS application and distribute on the App Store!
© 2025 Tauri Contributors. CC-BY / MIT
