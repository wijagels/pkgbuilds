# Maintainer: Mansour Behabadi <mansour@oxplot.com>
# Contributor: Troy Engel <troyengel+arch@gmail.com>
# Contributor: Geoff Hill <geoff@geoffhill.org>
# Contributor: Sebastien Bariteau <numkem@numkem.org>
# Contributor: Justin Dray <justin@dray.be>

pkgname="google-cloud-sdk"
pkgver=163.0.0
pkgrel=1
pkgdesc="Tools and libraries SDK for managing resources on the Google Cloud Platform, plus Python/PHP appengine SDK components"
url="https://cloud.google.com/sdk/"
license=("Apache")
arch=('i686' 'x86_64')
# replaces() only works for sysupgrade, not normal install/upgrade
conflicts=('google-appengine-python-php'
           'google-appengine-python' 'google-appengine-php')
replaces=('google-appengine-python-php'
          'google-appengine-python' 'google-appengine-php')
depends=('python2')
optdepends=('go: for Go version of App Engine'
            'java-environment: for Java version of App Engine'
            'php: for PHP version of App Engine'
            'python2-crcmod: verify the integrity of object contents')
options=('!strip' 'staticlibs')

# 64bit
source_x86_64=("https://dl.google.com/dl/cloudsdk/release/downloads/$pkgname-$pkgver-linux-x86_64.tar.gz"
               "profile.sh")
sha256sums_x86_64=(
  '261bc89bd2306d6bef9185fd0a2241f0f64b1a86a04eb3e12c38d883bfabe70b'
  '36ac88de630e49ea4b067b1f5f229142e4cf97561b98b3bd3d8115a356946692')
# 32bit
source_i686=("https://dl.google.com/dl/cloudsdk/release/downloads/$pkgname-$pkgver-linux-x86.tar.gz"
             "profile.sh")
sha256sums_i686=(
  'e8bfb369823f59e08f877754a5e2df09361345a9165b72439365077093893c4b'
  '36ac88de630e49ea4b067b1f5f229142e4cf97561b98b3bd3d8115a356946692')

prepare() {

  msg2 "Checking for newer upstream release"
  _LATEST=$(curl -s https://dl.google.com/dl/cloudsdk/release/sha256.txt | 
            egrep "google-cloud-sdk-.*-linux-x86_64.tar.gz" | \
            awk '{print $2}' | cut -d'-' -f4)
  if [ "$_LATEST" != "$pkgver" ]; then
    msg2 "This AUR release: $pkgver"
    msg2 "Latest upstream release: $_LATEST"
    msg2 "** Please flag out-of-date at https://aur.archlinux.org/packages/google-cloud-sdk"
  fi

}

package() {

  msg2 "Copying core SDK components"
  mkdir "$pkgdir/opt"
  cp -r "$srcdir/$pkgname" "$pkgdir/opt"

  # app-engine-python is actually the PHP+Python SDK widgets combined
  # NOTE: due to how Google is using argparse we must bare word the components
  msg2 "Running bootstrapping script and adding app-engine-python"
  python2 "$pkgdir/opt/$pkgname/bin/bootstrapping/install.py" \
    --usage-reporting false --path-update false --bash-completion false \
    --rc-path="$srcdir/fake.bashrc" \
    --additional-components app-engine-python

  # https://issuetracker.google.com/issues/35900282
  msg2 "Fixing appengine bug #35900282 (RAND_egd)"
  sed -i 's/from _ssl import RAND_add, RAND_egd, RAND_status/from _ssl import RAND_add, RAND_status/g' "$pkgdir/opt/$pkgname/platform/google_appengine/google/appengine/dist27/socket.py"
  python2 -m py_compile "$pkgdir/opt/$pkgname/platform/google_appengine/google/appengine/dist27/socket.py"

  # This is the strangest design they made to backup a fresh install
  msg2 "Removing unnecessary backups created by bootstrap"
  rm -rf "$pkgdir/opt/$pkgname/.install/.backup"
  mkdir "$pkgdir/opt/$pkgname/.install/.backup"

  msg2 "Setting up environment variables and shell completion"
  install -Dm755 "$pkgdir/opt/$pkgname/completion.bash.inc" \
    "$pkgdir/etc/bash_completion.d/google-cloud-sdk"
  install -Dm755 "$srcdir/profile.sh" \
    "$pkgdir/etc/profile.d/google-cloud-sdk.sh"

  msg2 "Fixing python references for python2"
  grep -Irl 'python' "$pkgdir/opt/$pkgname" | \
    xargs sed -i 's|#!.*python\b|#!/usr/bin/env python2|g'
  find "$pkgdir/opt/$pkgname/bin/" -maxdepth 1 -type f -exec \
    sed -i 's/CLOUDSDK_PYTHON=python\b/CLOUDSDK_PYTHON=python2/g' {} \;

  # These are only present in the direct numbered downloads we use
  msg2 "Installing man pages"
  mkdir -p "$pkgdir/usr/share"
  mv -f "$pkgdir/opt/$pkgname/help/man" "$pkgdir/usr/share/"
  chmod 0755 "$pkgdir/usr/share/man"
  chmod 0755 "$pkgdir/usr/share/man/man1"

  msg2 "Creating symlinks for binaries"
  mkdir -p "$pkgdir/usr/bin"
  find "$pkgdir/opt/$pkgname/bin" -maxdepth 1 -type f -printf \
    "/opt/$pkgname/bin/%f\n" | xargs ln -st "$pkgdir/usr/bin"

  # The tarball is rather sloppy with it's file permissions
  msg2 "Fixing file permissions"
  chown -R root:root "$pkgdir"
  chmod -x "$pkgdir"/usr/share/man/man1/*
  find "$pkgdir/opt/$pkgname" -name "*.html" -print0 | xargs -0 chmod -x
  find "$pkgdir/opt/$pkgname" -name "*.json" -print0 | xargs -0 chmod -x
  find "$pkgdir/opt/$pkgname" -name "*_test.py" -print0 | xargs -0 chmod +x

}

