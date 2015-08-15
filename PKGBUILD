# Maintainer: Cesasol <cesasol@hotmail.com>

# enable this if you run out of memory during linking
#_lowmem=true

# try to build with PGO
#_pgo=true

pkgname=firefox-kde-global-menu
pkgver=30.0
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org with OpenSUSE patch, integrate better with KDE"
arch=('i686' 'x86_64')
license=('MPL' 'GPL' 'LGPL')
url="https://build.opensuse.org/package/show/mozilla:Factory/MozillaFirefox"
depends=('gtk2' 'mozilla-common' 'libxt' 'startup-notification' 'mime-types'
         'dbus-glib' 'alsa-lib' 'desktop-file-utils' 'hicolor-icon-theme'
	 'libvpx' 'icu'  'libevent' 'nss' 'nspr>=4.10.6' 'hunspell' 'sqlite' 'libnotify' 'kmozillahelper')
makedepends=('unzip' 'zip' 'diffutils' 'python2' 'yasm' 'mesa' 'imake'
             'xorg-server-xvfb' 'libpulse' 'gst-plugins-base-libs'
	     'inetutils' 'clang')
optdepends=('networkmanager: Location detection via available WiFi networks'
	    'gst-plugins-good: h.264 video'
	    'gst-libav: h.264 video'
            'libpulse: PulseAudio audio driver')
provides=("firefox=${pkgver}")
conflicts=('firefox')
install=firefox.install
options=('!emptydirs')
_patchrev=8aaf86df3c83
_patchurl=http://www.rosenauer.org/hg/mozilla/raw-file/$_patchrev
source=(https://ftp.mozilla.org/pub/mozilla.org/firefox/releases/$pkgver/source/firefox-$pkgver.source.tar.bz2
        mozconfig firefox.desktop firefox-install-dir.patch vendor.js kde.js firefox-20.0.1-fixed-loading-icon.png
	# Firefox patchset
	$_patchurl/firefox-branded-icons.patch
	$_patchurl/firefox-kde.patch
	$_patchurl/firefox-kde-114.patch
	$_patchurl/firefox-no-default-ualocale.patch
	# Gecko/toolkit patchset
	$_patchurl/mozilla-kde.patch
	$_patchurl/mozilla-language.patch
	$_patchurl/mozilla-nongnome-proxies.patch
	$_patchurl/mozilla-prefer_plugin_pref.patch
	$_patchurl/toolkit-download-folder.patch
	unity-menubar.patch
)

_google_api_key=AIzaSyDwr302FpOSkGRpLlUpPThNTDPbXcIn_FM

prepare() {
  cd mozilla-release

  cp "$srcdir/mozconfig" .mozconfig

  patch -Np1 -i "$srcdir/firefox-install-dir.patch"
  echo -n "$_google_api_key" > google-api-key

  msg "Patching for KDE"
  patch -Np1 -i "$srcdir/toolkit-download-folder.patch"
  patch -Np1 -i "$srcdir/mozilla-nongnome-proxies.patch"
  patch -Np1 -i "$srcdir/mozilla-prefer_plugin_pref.patch"
  patch -Np1 -i "$srcdir/mozilla-kde.patch"
  patch -Np1 -i "$srcdir/mozilla-language.patch"

  patch -Np1 -i "$srcdir/firefox-kde.patch"
  patch -Np1 -i "$srcdir/firefox-kde-114.patch"
  patch -Np1 -i "$srcdir/firefox-no-default-ualocale.patch"
  patch -Np1 -i "$srcdir/firefox-branded-icons.patch"

  # configure script misdetects the preprocessor without an optimization level
  # https://bugs.archlinux.org/task/34644
  sed -i '/ac_cpp=/s/$CPPFLAGS/& -O2/' configure

  # WebRTC build tries to execute "python" and expects Python 2
  mkdir -p "$srcdir/path"
  ln -sf /usr/bin/python2 "$srcdir/path/python"

  msg "Patching for global menu"
  patch -Np1 -i "$srcdir/unity-menubar.patch"
  
  # Fix tab loading icon (flickers with libpng 1.6)
  # https://bugzilla.mozilla.org/show_bug.cgi?id=841734
  cp "$srcdir/firefox-20.0.1-fixed-loading-icon.png" \
    browser/themes/linux/tabbrowser/loading.png

}

build() {
  if pacman -T firefox && ! pacman -T "firefox=$pkgver"; then
    error "Please uninstall firefox temporarily before building it (pacman -Rdd ...)"
    exit 1
  fi

  cd mozilla-release

  export PATH="$srcdir/path:$PATH"
  export LDFLAGS="$LDFLAGS -Wl,-rpath,/usr/lib/firefox"
  export PYTHON="/usr/bin/python2"
  export CC=clang
  export CXX=clang++

  if [[ -n $_lowmem || $CARCH == i686 ]]; then
    LDFLAGS+=" -Wl,--no-keep-memory"
  fi

  if [[ -n $_pgo ]]; then
    # Set up PGO
    export DISPLAY=:99
    Xvfb -nolisten tcp -extension GLX -screen 0 1280x1024x24 $DISPLAY &

    if ! make -f client.mk build MOZ_PGO=1; then
      kill $!
      return 1
    fi

    kill $! || true
  else
    make -f client.mk build
  fi
}

package() {
  cd mozilla-release

  [[ "$CARCH" == "i686" ]] && cp "$srcdir/kde.js" obj-i686-pc-linux-gnu/dist/bin/defaults/pref
  [[ "$CARCH" == "x86_64" ]] && cp "$srcdir/kde.js" obj-x86_64-unknown-linux-gnu/dist/bin/defaults/pref

  make -f client.mk DESTDIR="$pkgdir" INSTALL_SDK= install

  install -Dm644 "$srcdir/vendor.js" "$pkgdir/usr/lib/firefox/browser/defaults/preferences/vendor.js"
  install -Dm644 "$srcdir/kde.js" "$pkgdir/usr/lib/firefox/browser/defaults/preferences/kde.js"

  for i in 16 22 24 32 48 256; do
      install -Dm644 browser/branding/official/default$i.png \
        "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/firefox.png"
  done

  install -Dm644 browser/branding/official/content/icon64.png \
    "$pkgdir/usr/share/icons/hicolor/64x64/apps/firefox.png"
  install -Dm644 browser/branding/official/mozicon128.png \
    "$pkgdir/usr/share/icons/hicolor/128x128/apps/firefox.png"
  install -Dm644 browser/branding/official/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/firefox.png"
  install -Dm644 browser/branding/official/content/about-logo@2x.png \
    "$pkgdir/usr/share/icons/hicolor/384x384/apps/firefox.png"

  install -Dm644 "$srcdir/firefox.desktop" "$pkgdir/usr/share/applications/firefox.desktop"

  # Use system-provided dictionaries
  rm -rf "$pkgdir"/usr/lib/firefox/{dictionaries,hyphenation}
  ln -s /usr/share/hunspell "$pkgdir/usr/lib/firefox/dictionaries"
  ln -s /usr/share/hyphen "$pkgdir/usr/lib/firefox/hyphenation"

  #workaround for now
  #https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -sf firefox "$pkgdir/usr/lib/firefox/firefox-bin"
}

sha256sums=('1e95740a8cf7095e210fb6a2313c4d0fba4fdf44ee7c327d01f202638403c22c'
            '94410d3e8bfe74b79ecd9dace9c4a339c25fa10908a7471001f533d843b582c9'
            '175d03cf3d8eb420d24a5a3072be69c9603e3ddacd3a83f68f49754eaecf6c5a'
            'd86e41d87363656ee62e12543e2f5181aadcff448e406ef3218e91865ae775cd'
            '4b50e9aec03432e21b44d18c4c97b2630bace606b033f7d556c9d3e3eb0f4fa4'
            'b8cc5f35ec35fc96ac5c5a2477b36722e373dbb57eba87eb5ad1276e4df7236d'
            '68e3a5b47c6d175cc95b98b069a15205f027cab83af9e075818d38610feb6213'
            '9c65592e23e242609899c421066e4d9094b8759c529e5608bcb08ec201bf088b'
            'ef9074076c068ccbf624372ad234e75a56f5036892eae9e46b2500e66816d05c'
            '1fd54e93db3a4d454ece5a6dc788a4ae36805eb497482add82a11ac5cf990562'
            '3d439f0a65bc285a43284dfd02d71facdfcf172afc19889bd66edc90c59cdc19'
            '87c1682abe508faa3b59e5f2ef39127f0ba165b4e454040163f22d94abdccdf0'
            '15ba009c12b76c20744cbc8d7d1840b979a5962b536f01798bb0c4bf04500bfb'
            'e8289ea4c1f8191e1e23661312ceee2128b8e790501b9a589d0d7bfc4384553f'
            '5f6b0970284d68d5ed18e6bb7ee1e9fc0025ab3c10aaa14c283adb21a4a20ee8'
            '60ace1c57577878850e05976ccf02f7e71512c4d62a8a9a6d270a7941d9625d3'
            '1582eaaaefe1b4972c84aed46470bc4b3c8fca2620fd2f7a49b6f9706190ce6b')
