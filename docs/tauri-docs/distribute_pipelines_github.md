Skip to content
# GitHub
This guide will show you how to use tauri-action in GitHub Actions to easily build and upload your app, and how to make Tauri’s updater query the newly created GitHub release for updates.
Lastly, it will also show how to set up a more complicated build pipeline for Linux Arm AppImages.
## Getting Started
To set up `tauri-action` you must first set up a GitHub repository. You can also use this action on a repository that does not have Tauri configured yet since it can automatically initialize Tauri for you, please see the action’s readme for necessary configuration options.
Go to the Actions tab on your GitHub project page and select “New workflow”, then choose “Set up a workflow yourself”. Replace the file with the workflow from below or from one of the action’s examples.
## Configuration
Please see the `tauri-action` readme for all available configuration options.
When your app is not on the root of the repository, use the `projectPath` input.
You may freely modify the workflow name, change its triggers, and add more steps such as `npm run lint` or `npm run test`. The important part is that you keep the below line at the end of the workflow since this runs the build script and releases your app.
### How to Trigger
The release workflow shown below and in the `tauri-action` examples is triggered by pushed to the `release` branch. The action automatically creates a git tag and a title for the GitHub release using the application version.
As another example, you can also change the trigger to run the workflow on the push of a version git tag such as `app-v0.7.0`:
```

name: 'publish'
on:
push:
tags:
- 'app-v*'

```

For a full list of possible trigger configurations, check out the official GitHub documentation.
## Example Workflow
Below is an example workflow that has been set up to run every time you push to the `release` branch.
This workflow will build and release your app for Linux x64, Windows x64, macOS x64 and macOS Arm64 (M1 and above).
The steps this workflow takes are:
  1. Checkout the repository using `actions/checkout@v4`.
  2. Install Linux system dependencies required to build the app.
  3. Set up Node.js LTS and a cache for global npm/yarn/pnpm package data using `actions/setup-node@v4`.
  4. Set up Rust and a cache for Rust’s build artifacts using `dtolnay/rust-toolchain@stable` and `swatinem/rust-cache@v2`.
  5. Install the frontend dependencies and, if not configured as `beforeBuildCommand`, run the web app’s build script.
  6. Lastly, it uses `tauri-apps/tauri-action@v0` to run `tauri build`, generate the artifacts, and create a GitHub release.


```

name: 'publish'
on:
workflow_dispatch:
push:
branches:
- release
jobs:
publish-tauri:
permissions:
contents: write
strategy:
fail-fast: false
matrix:
include:
- platform: 'macos-latest'# for Arm based macs (M1 and above).
args: '--target aarch64-apple-darwin'
- platform: 'macos-latest'# for Intel based macs.
args: '--target x86_64-apple-darwin'
- platform: 'ubuntu-22.04'
args: ''
- platform: 'windows-latest'
args: ''
runs-on: ${{ matrix.platform }}
steps:
- uses: actions/checkout@v4
- name: install dependencies (ubuntu only)
if: matrix.platform == 'ubuntu-22.04'# This must match the platform value defined above.
run: |
sudo apt-get update
sudo apt-get install -y libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf
- name: setup node
uses: actions/setup-node@v4
with:
node-version: lts/*
cache: 'yarn'# Set this to npm, yarn or pnpm.
- name: install Rust stable
uses: dtolnay/rust-toolchain@stable# Set this to dtolnay/rust-toolchain@nightly
with:
# Those targets are only used on macos runners so it's in an `if` to slightly speed up windows and linux builds.
targets: ${{ matrix.platform == 'macos-latest' && 'aarch64-apple-darwin,x86_64-apple-darwin' || '' }}
- name: Rust cache
uses: swatinem/rust-cache@v2
with:
workspaces: './src-tauri -> target'
- name: install frontend dependencies
# If you don't have `beforeBuildCommand` configured you may want to build your frontend here too.
run: yarn install# change this to npm or pnpm depending on which one you use.
- uses: tauri-apps/tauri-action@v0
env:
GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
with:
tagName: app-v__VERSION__# the action automatically replaces \_\_VERSION\_\_ with the app version.
releaseName: 'App v__VERSION__'
releaseBody: 'See the assets to download this version and install.'
releaseDraft: true
prerelease: false
args: ${{ matrix.args }}

```

For more configuration options, check out the `tauri-action` repository and its examples.
## Arm Runner Compilation
This workflow uses `pguyot/arm-runner-action` to compile directly on an emulated Arm runner. This bridges the gap for missing cross-architecture build support in the AppImage tooling.
```

name: 'Publish Linux Arm builds'
on:
workflow_dispatch:
push:
branches:
- release
jobs:
build:
runs-on: ubuntu-22.04
strategy:
matrix:
arch: [aarch64, armv7l]
include:
- arch: aarch64
cpu: cortex-a72
base_image: https://dietpi.com/downloads/images/DietPi_RPi-ARMv8-Bookworm.img.xz
deb: arm64
rpm: aarch64
appimage: aarch64
- arch: armv7l
cpu: cortex-a53
deb: armhfp
rpm: arm
appimage: armhf
base_image: https://dietpi.com/downloads/images/DietPi_RPi-ARMv7-Bookworm.img.xz
steps:
- uses: actions/checkout@v3
- name: Cache rust build artifacts
uses: Swatinem/rust-cache@v2
with:
workspaces: src-tauri
cache-on-failure: true
- name: Build app
uses: pguyot/arm-runner-action@v2.6.5
with:
base_image: ${{ matrix.base_image }}
cpu: ${{ matrix.cpu }}
bind_mount_repository: true
image_additional_mb: 10240
optimize_image: no
#exit_on_fail: no
commands: |
# Prevent Rust from complaining about $HOME not matching eid home
export HOME=/root
# Workaround to CI worker being stuck on Updating crates.io index
export CARGO_REGISTRIES_CRATES_IO_PROTOCOL=sparse
# Install setup prerequisites
apt-get update -y --allow-releaseinfo-change
apt-get autoremove -y
apt-get install -y --no-install-recommends --no-install-suggests curl libwebkit2gtk-4.1-dev build-essential libssl-dev libgtk-3-dev libayatana-appindicator3-dev librsvg2-dev patchelf libfuse2 file
curl https://sh.rustup.rs -sSf | sh -s -- -y
. "$HOME/.cargo/env"
curl -fsSL https://deb.nodesource.com/setup_lts.x | bash
apt-get install -y nodejs
# Install frontend dependencies
npm install
# Build the application
npm run tauri build -- --verbose
- name: Get app version
run: echo "APP_VERSION=$(jq -r .version src-tauri/tauri.conf.json)" >> $GITHUB_ENV
# TODO: Combine this with the basic workflow and upload the files to the Release.
- name: Upload deb bundle
uses: actions/upload-artifact@v3
with:
name: Debian Bundle
path: ${{ github.workspace }}/src-tauri/target/release/bundle/deb/appname_${{ env.APP_VERSION }}_${{ matrix.deb }}.deb
- name: Upload rpm bundle
uses: actions/upload-artifact@v3
with:
name: RPM Bundle
path: ${{ github.workspace }}/src-tauri/target/release/bundle/rpm/appname-${{ env.APP_VERSION }}-1.${{ matrix.rpm }}.rpm
- name: Upload appimage bundle
uses: actions/upload-artifact@v3
with:
name: AppImage Bundle
path: ${{ github.workspace }}/src-tauri/target/release/bundle/appimage/appname_${{ env.APP_VERSION }}_${{ matrix.appimage }}.AppImage

```

## Troubleshooting
### GitHub Environment Token
The GitHub Token is automatically issued by GitHub for each workflow run without further configuration, which means there is no risk of secret leakage. This token however only has read permissions by default and you may get a “Resource not accessible by integration” error when running the workflow. If this happens, you may need to add write permissions to this token. To do this, go to your GitHub project settings, select `Actions`, scroll down to `Workflow permissions`, and check “Read and write permissions”.
You can see the GitHub Token being passed to the workflow via this line in the workflow:
```

env:
GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

© 2025 Tauri Contributors. CC-BY / MIT
