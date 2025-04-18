Skip to content
# Updater
GitHub npm  crates.io 
API Reference 
Automatically update your Tauri app with an update server or a static JSON.
## Supported Platforms
_This plugin requires a Rust version of at least**1.77.2**_
Platform | Level | Notes  
---|---|---  
windows  
linux  
macos  
android  
ios  
## Setup
Install the Tauri updater plugin to get started.
  * Automatic 
  * Manual 


Use your project’s package manager to add the dependency:
  * npm 
  * yarn 
  * pnpm 
  * deno 
  * bun 
  * cargo 


```

npmruntauriaddupdater

```

```

yarnruntauriaddupdater

```

```

pnpmtauriaddupdater

```

```

denotasktauriaddupdater

```

```

buntauriaddupdater

```

```

cargotauriaddupdater

```

  1. Run the following command in the `src-tauri` folder to add the plugin to the project’s dependencies in `Cargo.toml`:
```

cargoaddtauri-plugin-updater--target'cfg(any(target_os = "macos", windows, target_os = "linux"))'

```

  2. Modify `lib.rs` to initialize the plugin:
lib.rs```

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pubfnrun() {
tauri::Builder::default()
.setup(|app| {
#[cfg(desktop)]
app.handle().plugin(tauri_plugin_updater::Builder::new().build());
Ok(())
})
.run(tauri::generate_context!())
.expect("error while running tauri application");
}

```

  3. You can install the JavaScript Guest bindings using your preferred JavaScript package manager:
     * npm 
     * yarn 
     * pnpm 
     * deno 
     * bun 
```

npminstall@tauri-apps/plugin-updater

```

```

yarnadd@tauri-apps/plugin-updater

```

```

pnpmadd@tauri-apps/plugin-updater

```

```

denoaddnpm:@tauri-apps/plugin-updater

```

```

bunadd@tauri-apps/plugin-updater

```



## Signing updates
Tauri’s updater needs a signature to verify that the update is from a trusted source. This cannot be disabled.
To sign your updates you need two keys:
  1. The public key, which will be set in the `tauri.conf.json` to validate the artifacts before the installation. This public key can be uploaded and shared safely as long as your private key is secure.
  2. The private key, which is used to sign your installer files. You should NEVER share this key with anyone. Also, if you lose this key you will NOT be able to publish new updates to the users that have the app already installed. It is important to store this key in a safe place!


To generate the keys the Tauri CLI provides the `signer generate` command. You can run this to create the keys in the home folder:
  * npm 
  * yarn 
  * pnpm 
  * deno 
  * bun 
  * cargo 


```

npmruntaurisignergenerate---w~/.tauri/myapp.key

```

```

yarntaurisignergenerate-w~/.tauri/myapp.key

```

```

pnpmtaurisignergenerate-w~/.tauri/myapp.key

```

```

denotasktaurisignergenerate-w~/.tauri/myapp.key

```

```

bunxtaurisignergenerate-w~/.tauri/myapp.key

```

```

cargotaurisignergenerate-w~/.tauri/myapp.key

```

### Building
While building your update artifacts, you need to have the private key you generated above in your environment variables. `.env` files do _not_ work!
  * Mac/Linux 
  * Windows 


```

exportTAURI_SIGNING_PRIVATE_KEY="Path or content of your private key"
# optionally also add a password
exportTAURI_SIGNING_PRIVATE_KEY_PASSWORD=""

```

Run this in `PowerShell`:
```

$env:TAURI_SIGNING_PRIVATE_KEY="Path or content of your private key"
<# optionally also add a password #>
$env:TAURI_SIGNING_PRIVATE_KEY_PASSWORD=""

```

After that, you can run Tauri build as usual and Tauri will generate the update bundles and their signatures. The generated files depend on the `createUpdaterArtifacts` configuration value configured below.
  * v2 
  * v1 compatible 


```

{
"bundle": {
"createUpdaterArtifacts": true
}
}

```

On Linux, Tauri will create the normal AppImage inside the `target/release/bundle/appimage/` folder:
  * `myapp.AppImage` - The standard app bundle. It will be re-used by the updater.
  * `myapp.AppImage.sig` - The signature of the updater bundle.


On macOS, Tauri will create a .tar.gz archive from the application bundle inside the target/release/bundle/macos/ folder:
  * `myapp.app` - The standard app bundle.
  * `myapp.app.tar.gz` - The updater bundle.
  * `myapp.app.tar.gz.sig` - The signature of the update bundle.


On Windows, Tauri will create the normal MSI and NSIS installers inside the target/release/bundle/msi/ and target/release/bundle/nsis folders:
  * `myapp-setup.exe` - The standard app bundle. It will be re-used by the updater.
  * `myapp-setup.exe.sig` - The signature of the update bundle.
  * `myapp.msi` - The standard app bundle. It will be re-used by the updater.
  * `myapp.msi.sig` - The signature of the update bundle.


```

{
"bundle": {
"createUpdaterArtifacts": "v1Compatible"
}
}

```

On Linux, Tauri will create a .tar.gz archive from the AppImage inside the `target/release/bundle/appimage/` folder:
  * `myapp.AppImage` - The standard app bundle.
  * `myapp.AppImage.tar.gz` - The updater bundle.
  * `myapp.AppImage.tar.gz.sig` - The signature of the update bundle.


On macOS, Tauri will create a .tar.gz archive from the application bundle inside the target/release/bundle/macos/ folder:
  * `myapp.app` - The standard app bundle.
  * `myapp.app.tar.gz` - The updater bundle.
  * `myapp.app.tar.gz.sig` - The signature of the update bundle.


On Windows, Tauri will create .zip archives from the MSI and NSIS installers inside the target/release/bundle/msi/ and target/release/bundle/nsis folders:
  * `myapp-setup.exe` - The standard app bundle.
  * `myapp-setup.nsis.zip` - The updater bundle.
  * `myapp-setup.nsis.zip.sig` - The signature of the update bundle.
  * `myapp.msi` - The standard app bundle.
  * `myapp.msi.zip` - The updater bundle.
  * `myapp.msi.zip.sig` - The signature of the update bundle.


## Tauri Configuration
Set up the `tauri.conf.json` in this format for the updater to start working.
Keys| Description  
---|---  
`createUpdaterArtifacts`| Setting this to `true` tells Tauri’s app bundler to create updater artifacts. If you’re migrating your app from an older Tauri version, set it to `"v1Compatible"` instead. **This setting will be removed in v3** so make sure to change it to `true` once all your users are migrated to v2.  
`pubkey`| This has to be the public key generated from the Tauri CLI in the step above. It **cannot** be a file path!  
`endpoints`| This must be an array of endpoint URLs as strings. TLS is enforced in production mode. Tauri will only continue to the next url if a non-2XX status code is returned!  
`dangerousInsecureTransportProtocol`| Setting this to `true` allows the updater to accept non-HTTPS endpoints. Use this configuration with caution!  
Each updater URL can contain the following dynamic variables, allowing you to determine server-side if an update is available.
  * `{{current_version}}`: The version of the app that is requesting the update.
  * `{{target}}`: The operating system name (one of `linux`, `windows` or `darwin`).
  * `{{arch}}`: The architecture of the machine (one of `x86_64`, `i686`, `aarch64` or `armv7`).


tauri.conf.json```

{
"bundle": {
"createUpdaterArtifacts": true
},
"plugins": {
"updater": {
"pubkey": "CONTENT FROM PUBLICKEY.PEM",
"endpoints": [
"https://releases.myapp.com/{{target}}/{{arch}}/{{current_version}}",
// or a static github json file
"https://github.com/user/repo/releases/latest/download/latest.json"
]
}
}
}

```

### `installMode` on Windows
On Windows there is an additional optional `"installMode"` config to change how the update is installed.
tauri.conf.json```

{
"plugins": {
"updater": {
"windows": {
"installMode": "passive"
}
}
}
}

```

  * `"passive"`: There will be a small window with a progress bar. The update will be installed without requiring any user interaction. Generally recommended and the default mode.
  * `"basicUi"`: There will be a basic user interface shown which requires user interaction to finish the installation.
  * `"quiet"`: There will be no progress feedback to the user. With this mode the installer cannot request admin privileges by itself so it only works in user-wide installations or when your app itself already runs with admin privileges. Generally not recommended.


## Server Support
The updater plugin can be used in two ways. Either with a dynamic update server or a static JSON file (to use on services like S3 or GitHub gists).
### Static JSON File
When using static, you just need to return a JSON containing the required information.
Keys| Description  
---|---  
`version`| Must be a valid SemVer, with or without a leading `v`, meaning that both `1.0.0` and `v1.0.0` are valid.  
`notes`| Notes about the update.  
`pub_date`| The date must be formatted according to RFC 3339 if present.  
`platforms`| Each platform key is in the `OS-ARCH` format, where `OS` is one of `linux`, `darwin` or `windows`, and `ARCH` is one of `x86_64`, `aarch64`, `i686` or `armv7`.  
`signature`| The content of the generated `.sig` file, which may change with each build. A path or URL does not work!  
When using custom targets the provided target string is matched against the `platforms` key instead of the default `OS-ARCH` value.
The required keys are `"version"`, `"platforms.[target].url"` and `"platforms.[target].signature"`; the others are optional.
```

{
"version": "",
"notes": "",
"pub_date": "",
"platforms": {
"linux-x86_64": {
"signature": "",
"url": ""
},
"windows-x86_64": {
"signature": "",
"url": ""
},
"darwin-x86_64": {
"signature": "",
"url": ""
}
}
}

```

Note that Tauri will validate the whole file before checking the version field, so make sure all existing platform configurations are valid and complete.
### Dynamic Update Server
When using a dynamic update server, Tauri will follow the server’s instructions. To disable the internal version check you can overwrite the plugin’s version comparison, this will install the version sent by the server (useful if you need to roll back your app).
Your server can use variables defined in the `endpoint` URL above to determine if an update is required. If you need more data, you can include additional request headers in Rust to your liking.
Your server should respond with a status code of `204 No Content` if there is no update available.
If an update is required, your server should respond with a status code of `200 OK` and a JSON response in this format:
Keys| Description  
---|---  
`version`| This Must be a valid SemVer, with or without a leading `v`, meaning that both `1.0.0` and `v1.0.0` are valid.  
`notes`| Notes about the update.  
`pub_date`| The date must be formatted according to RFC 3339 if present.  
`url`| This Must be a valid URL to the update bundle.  
`signature`| The content of the generated `.sig` file, which may change with each build. A path or URL does not work!  
The required keys are `"url"`, `"version"` and `"signature"`; the others are optional.
```

{
"version": "",
"pub_date": "",
"url": "",
"signature": "",
"notes": ""
}

```

## Checking for Updates
The default API for checking updates and installing them leverages the configured endpoints and can be accessed by both JavaScript and Rust code.
  * JavaScript 
  * Rust 


```

import { check } from'@tauri-apps/plugin-updater';
import { relaunch } from'@tauri-apps/plugin-process';
const update = await check();
if (update) {
console.log(
`found update ${update.version} from ${update.date} with notes ${update.body}`
);
let downloaded = 0;
let contentLength = 0;
// alternatively we could also call update.download() and update.install() separately
awaitupdate.downloadAndInstall((event)=> {
switch (event.event) {
case'Started':
contentLength=event.data.contentLength;
console.log(`started downloading ${event.data.contentLength} bytes`);
break;
case'Progress':
downloaded+=event.data.chunkLength;
console.log(`downloaded ${downloaded} from ${contentLength}`);
break;
case'Finished':
console.log('download finished');
break;
}
});
console.log('update installed');
awaitrelaunch();
}

```

For more information see the JavaScript API documentation.
src-tauri/src/lib.rs```

use tauri_plugin_updater::UpdaterExt;
pubfnrun() {
tauri::Builder::default()
.setup(|app| {
lethandle=app.handle().clone();
tauri::async_runtime::spawn(asyncmove {
update(handle).await.unwrap();
});
Ok(())
})
.run(tauri::generate_context!())
.unwrap();
}
asyncfnupdate(app: tauri::AppHandle) -> tauri_plugin_updater::Result<()> {
iflet Some(update) =app.updater()?.check().await? {
letmutdownloaded=0;
// alternatively we could also call update.download() and update.install() separately
update
.download_and_install(
|chunk_length, content_length| {
downloaded+=chunk_length;
println!("downloaded {downloaded} from {content_length:?}");
},
|| {
println!("download finished");
},
)
.await?;
println!("update installed");
app.restart();
}
Ok(())
}

```

For more information see the Rust API documentation.
Note that restarting your app immediately after installing an update is not required and you can choose how to handle the update by either waiting until the user manually restarts the app, or prompting him to select when to do so.
When checking and downloading updates it is possible to define a custom request timeout, a proxy and request headers.
  * JavaScript 
  * Rust 


```

import { check } from'@tauri-apps/plugin-updater';
const update = await check({
proxy: '<proxy url>',
timeout: 30000/* milliseconds */,
headers: {
Authorization: 'Bearer <token>',
},
});

```

```

use tauri_plugin_updater::UpdaterExt;
letupdate=app
.updater_builder()
.timeout(std::time::Duration::from_secs(30))
.proxy("<proxy-url>".parse().expect("invalid URL"))
.header("Authorization", "Bearer <token>")
.build()?
.check()
.await?;

```

### Runtime Configuration
The updater APIs also allows the updater to be configured at runtime for more flexibility. For security reasons some APIs are only available for Rust.
#### Endpoints
Setting the URLs that should be requested to check updates at runtime allows more dynamic updates such as separate release channels:
```

use tauri_plugin_updater::UpdaterExt;
letchannel=ifbeta { "beta" } else { "stable" };
letupdate_url=format!("https://{channel}.myserver.com/{{{{target}}}}-{{{{arch}}}}/{{{{current_version}}}}");
letupdate=app
.updater_builder()
.endpoints(vec![update_url])?
.build()?
.check()
.await?;

```

#### Public key
Setting the public key at runtime can be useful to implement a key rotation logic. It can be set by either the plugin builder or updater builder:
```

tauri_plugin_updater::Builder::new().pubkey("<your public key>").build()

```

```

use tauri_plugin_updater::UpdaterExt;
letupdate=app
.updater_builder()
.pubkey("<your public key>")
.build()?
.check()
.await?;

```

#### Custom target
By default the updater lets you use the `{{target}}` and `{{arch}}` variables to determine which update asset must be delivered. If you need more information on your updates (e.g. when distributing a Universal macOS binary option or having more build flavors) you can set a custom target.
  * JavaScript 
  * Rust 


```

import { check } from'@tauri-apps/plugin-updater';
const update = await check({
target: 'macos-universal',
});

```

Custom targets can be set by either the plugin builder or updater builder:
```

tauri_plugin_updater::Builder::new().target("macos-universal").build()

```

```

use tauri_plugin_updater::UpdaterExt;
letupdate=app
.updater_builder()
.target("macos-universal")
.build()?
.check()
.await?;

```

#### Allowing downgrades
By default Tauri checks if the update version is greater than the current app version to verify if it should update or not. To allow downgrades, you must use the updater builder’s `version_comparator` API:
```

use tauri_plugin_updater::UpdaterExt;
letupdate=app
.updater_builder()
.version_comparator(|current, update| {
// default comparison: `update.version > current`
update.version !=current
})
.build()?
.check()
.await?;

```

#### Windows before exit hook
Due to a limitation of Windows installers, Tauri will automatically quit your application before installing updates on Windows. To perform an action before that happens, use the `on_before_exit` function:
```

use tauri_plugin_updater::UpdaterExt;
letupdate=app
.updater_builder()
.on_before_exit(|| {
println!("app is about to exit on Windows!");
})
.build()?
.check()
.await?;

```

## Permissions
By default all potentially dangerous plugin commands and scopes are blocked and cannot be accessed. You must modify the permissions in your `capabilities` configuration to enable these.
See the Capabilities Overview for more information and the step by step guide to use plugin permissions.
src-tauri/capabilities/default.json```

{
"permissions": [
...,
"updater:default",
]
}

```

## Default Permission
This permission set configures which kind of updater functions are exposed to the frontend.
#### Granted Permissions
The full workflow from checking for updates to installing them is enabled.
  * `allow-check`
  * `allow-download`
  * `allow-install`
  * `allow-download-and-install`


## Permission Table
Identifier | Description  
---|---  
`updater:allow-check` |  Enables the check command without any pre-configured scope.  
`updater:deny-check` |  Denies the check command without any pre-configured scope.  
`updater:allow-download` |  Enables the download command without any pre-configured scope.  
`updater:deny-download` |  Denies the download command without any pre-configured scope.  
`updater:allow-download-and-install` |  Enables the download_and_install command without any pre-configured scope.  
`updater:deny-download-and-install` |  Denies the download_and_install command without any pre-configured scope.  
`updater:allow-install` |  Enables the install command without any pre-configured scope.  
`updater:deny-install` |  Denies the install command without any pre-configured scope.  
© 2025 Tauri Contributors. CC-BY / MIT
