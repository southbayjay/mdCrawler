Skip to content
# AUR
# Publishing To The Arch User Repository
## Setup
First go to `https://aur.archlinux.org` and make an account. Be sure to add the proper ssh keys. Next, clone an empty git repository using this command.
Terminal window```

gitclonehttps://aur.archlinux.org/your-repo-name

```

After completing the steps above, create a file with the name `PKGBUILD`. Once the file is created you can move onto the next step.
### Writing a PKGBUILD file
PKGBUILD```

pkgname=<pkgname>
pkgver=1.0.0
pkgrel=1
pkgdesc="Description of your app"
arch=('x86_64''aarch64')
url="https://github.com/<user>/<project>"
license=('mit')
depends=('cairo''desktop-file-utils''gdk-pixbuf2''glib2''gtk3''hicolor-icon-theme''libsoup''pango''webkit2gtk-4.1')
options=('!strip''!emptydirs')
install=${pkgname}.install
source_x86_64=("https://github.com/<user>/<project>/releases/download/v$pkgver/appname_"$pkgver"_amd64.deb")
source_aarch64=("https://github.com/<user>/<project>/releases/download/v$pkgver/appname_"$pkgver"_arm64.deb")

```

  * At the top of the file, define your package name and assign it the variable `pkgname`.
  * Set your `pkgver` variable. Typically it is best to use this variable in the source variable to increase maintainability.
  * The `pkgdesc` variable on your aur repo’s page and tells vistors what your app does.
  * The `arch` variable controls what architectures can install your package.
  * The `url` variable, while not required, helps to make your package appear more professional.
  * The `install` variable defines a file that runs the installation commands.
  * The `depends` variable includes a list of items that are required to make your app run. For any Tauri app you must include all of the dependencies shown above.
  * The `source` variable is required and defines the location where your upstream package is. You can make a `source` architecture specific by adding the architecture to the end of the variable name.


### Generating `SRCINFO`
In order to push your repo to the aur you must generate an `srcinfo` file. This can be done with this command.
Terminal window```

makepkg--printsrcinfo>>.SRCINFO

```

### Testing
Testing the app is extremely simple. All you have to do is run makepkg -f within the same directory as the pkgbuild file and see if it works
### Publishing
Finally, after the testing phase is over, you can publish the application to the arch user repository with these commands.
Terminal window```

gitadd.
gitcommit-m"Initial Commit"
gitpush

```

If all goes well, your repository should now appear on the aur website.
## Examples
### Extracting From A Debian Package
PKGBUILD```

# Maintainer:
# Contributor:
pkgname=<pkgname>
pkgver=1.0.0
pkgrel=1
pkgdesc="Description of your app"
arch=('x86_64''aarch64')
url="https://github.com/<user>/<project>"
license=('mit')
depends=('cairo''desktop-file-utils''gdk-pixbuf2''glib2''gtk3''hicolor-icon-theme''libsoup''pango''webkit2gtk-4.1')
options=('!strip''!emptydirs')
install=${pkgname}.install
source_x86_64=("https://github.com/<user>/<project>/releases/download/v$pkgver/appname_"$pkgver"_amd64.deb")
source_aarch64=("https://github.com/<user>/<project>/releases/download/v$pkgver/appname_"$pkgver"_arm64.deb")
sha256sums_x86_64=('ca85f11732765bed78f93f55397b4b4cbb76685088553dad612c5062e3ec651f')
sha256sums_aarch64=('ed2dc3169d34d91188fb55d39867713856dd02a2360ffe0661cb2e19bd701c3c')
package() {
# Extract package data
tar -xz -f data.tar.gz -C "${pkgdir}"
}

```

my-tauri-app.install```

post_install() {
gtk-update-icon-cache -q -t -f usr/share/icons/hicolor
update-desktop-database -q
}
post_upgrade() {
post_install
}
post_remove() {
gtk-update-icon-cache -q -t -f usr/share/icons/hicolor
update-desktop-database -q
}

```

### Building from source
PKGBUILD```

# Maintainer:
pkgname=<pkgname>-git
pkgver=1.1.0
pkgrel=1
pkgdesc="Description of your app"
arch=('any')
url="https://github.com/<user>/<project>"
license=('mit')
depends=('cairo''desktop-file-utils''gdk-pixbuf2''glib2''gtk3''hicolor-icon-theme''libsoup''pango''webkit2gtk-4.1')
makedepends=('git''file''openssl''appmenu-gtk-module''libappindicator-gtk3''librsvg''base-devel''curl''wget''rustup''npm''nodejs''dpkg')
provides=('<pkgname>')
conflicts=('<binname>''<pkgname>')
options=('!strip''!emptydirs')
source=('git+https://github.com/<user>/<project>')
sha256sums=('SKIP')
prepare() {
cd <project>
npm install
npm run tauri build
}
package() {
cd "$srcdir"/<project>/src-tauri/target/*unknown-linux*/release/bundle/deb
dpkg-deb -x *.deb here
cd here
install -Dm755 usr/bin/myapp "$pkgdir"/usr/bin/myapp
# Install desktop file
install -Dm644 usr/share/applications/myapp.desktop "$pkgdir"/usr/share/applications/myapp.desktop
# Install icons
install -Dm644 usr/share/icons/hicolor/128x128/apps/myapp.png "$pkgdir"/usr/share/icons/hicolor/128x128/apps/myapp.png
install -Dm644 usr/share/icons/hicolor/256x256@2/apps/myapp.png "$pkgdir"/usr/share/icons/hicolor/256x256@2/apps/myapp.png
install -Dm644 usr/share/icons/hicolor/32x32/apps/myapp.png "$pkgdir"/usr/share/icons/hicolor/32x32/apps/myapp.png
# Extract package data
}

```

© 2025 Tauri Contributors. CC-BY / MIT
