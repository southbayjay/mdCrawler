Skip to content
# AppImage
`AppImage` is a distribution format that does not rely on the system installed packages and instead bundles all dependencies and files needed by the application. For this reason, the output file is larger but easier to distribute since it is supported on many Linux distributions and can be executed without installation. The user just needs to make the file executable (`chmod a+x MyProject.AppImage`) and can then run it (`./MyProject.AppImage`).
AppImages are convenient, simplifying the distribution process if you cannot make a package targeting the distribution’s package manager. Still, you should carefully use it as the file size grows from the 2-6 MB range to 70+ MB.
## Multimedia support via GStreamer
If your app plays audio/video you need to enable `tauri.conf.json > bundle > linux > appimage > bundleMediaFramework`. This will increase the size of the AppImage bundle to include additional gstreamer files needed for media playback. This flag is currently only fully supported on Ubuntu build systems. Make sure that your build system has all the plugins your app may need at runtime.
## Custom Files
To include custom files in the AppImage that you do not want to include via Tauri’s `resources` feature, you can provide a list of files or folders in `tauri.conf.json > bundle > linux > appimage > files`. The configuration object maps the path in the AppImage to the path to the file on your filesystem, relative to the `tauri.conf.json` file. Here’s an example configuration:
tauri.conf.json```

{
"bundle": {
"linux": {
"appimage": {
"files": {
"/usr/share/README.md": "../README.md", // copies the ../README.md file to <appimage>/usr/share/README.md
"/usr/assets": "../assets/"// copies the entire ../assets directory to <appimage>/usr/assets
}
}
}
}
}

```

## AppImages for ARM-based devices
`linuxdeploy`, the AppImage tooling Tauri uses, currently does not support cross-compiling ARM AppImages. This means ARM AppImages can only be built on ARM devices or emulators.
Check out our GitHub Action guide for an example workflow that leverages QEMU to build the app. Note that this is extremely slow and only recommended in public repositories where Build Minutes are free. In private repositories GitHub’s ARM runners should be more cost-efficient and much easier to set up.
© 2025 Tauri Contributors. CC-BY / MIT
