# Maintainer: https://github.com/arturo-lang

pkgname=arturo
pkgver=0.10.0
pkgrel=1
pkgdesc="Simple, expressive & portable programming language for efficient scripting."
arch=('x86_64')
url="https://arturo-lang.io/"
license=("spdx:MIT")
purl=("pkg:github/arturo-lang/arturo@v${pkgver}")

msys2_repository_url="https://github.com/arturo-lang/arturo"
mingw_arch=('mingw64')

depends=(
  mingw-w64-x86_64-gmp
  mingw-w64-x86_64-mpfr
  mingw-w64-x86_64-gcc-libs
  mingw-w64-x86_64-sqlite3
  mingw-w64-x86_64-webview2-loader
)

makedepends=(
  base-devel
  git
  zip
  unzip
)

source=("https://github.com/arturo-lang/arturo/archive/refs/tags/v${pkgver}.zip")
sha256sums=('D7317D3DD0DA4E72F60F08242D332D2C807A3E8CBF790B6B2FD20E81ADE008FC')

install=arturo.install

options=(
  !staticlibs
  !libtool
  !strip
)

prepare() {
  git clone https://github.com/nim-lang/Nim "$srcdir/Nim"
  cd "$srcdir/Nim"
  git checkout v2.2.6
  cmd //C build_all.bat
  
  # Build webview.dll
  cd "$srcdir/arturo-$pkgver/src/extras/webview"
  cmd //C deps/build.bat deps build
  
  # Create the dlls directory structure if it doesn't exist
  mkdir -p "$srcdir/arturo-$pkgver/src/extras/webview/deps/dlls/x64"
  
  # Copy the generated webview.dll to the expected location
  cp build/library/webview.dll "$srcdir/arturo-$pkgver/src/extras/webview/deps/dlls/x64/"
}

build() {
  cd "$srcdir/arturo-$pkgver"
  ../Nim/bin/nim build.nims -l
}

package() {
  install -d "$pkgdir/opt/arturo" 
  cp -r "$srcdir/arturo-$pkgver/bin" "$pkgdir/opt/arturo"

  # Bundle runtime DLLs so arturo.exe works without relying on system PATH
  local bindir="$pkgdir/opt/arturo/bin"
  local dlls=(
    libgcc_s_seh-1.dll
    libgmp-10.dll
    libmpfr-6.dll
    libwinpthread-1.dll
    libsqlite3-0.dll
  )

  for dll in "${dlls[@]}"; do
    if [ -f "/mingw64/bin/$dll" ]; then
      cp "/mingw64/bin/$dll" "$bindir/"
    fi
  done

  # Copy webview DLLs built in prepare()
  cp "$srcdir/arturo-$pkgver/src/extras/webview/deps/dlls/x64/webview.dll" "$bindir/"
  
  # Copy WebView2Loader.dll from system or from deps
  if [ -f "/mingw64/bin/WebView2Loader.dll" ]; then
    cp "/mingw64/bin/WebView2Loader.dll" "$bindir/"
  elif [ -f "$srcdir/arturo-$pkgver/src/extras/webview/deps/dlls/x64/WebView2Loader.dll" ]; then
    cp "$srcdir/arturo-$pkgver/src/extras/webview/deps/dlls/x64/WebView2Loader.dll" "$bindir/"
  fi

  # Rename sqlite3 DLL
  mv "$bindir/libsqlite3-0.dll" "$bindir/sqlite3_64.dll"
}
