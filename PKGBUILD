# Local port of wearzdk/nahimic-linux v0.3.3 with ASRock desktop Line Out support,
# the vendor EQ presets from the Nahimic3 app bundle and a system tray icon.
# Same package name/version as the AUR package, pkgrel bumped: a later official
# release will cleanly upgrade over this local build.
# Differs from upstream: adds ASRock boards (B650M-H/M.2+ ALC897 verified),
# extracts the Nahimic3 EQ preset curves (Heavy Bass, Dynamic, …) as hash-pinned
# runtime data, shows them as per-profile preset chips in the equalizer, keeps
# a Windows-like tray icon (hide-to-tray, autostart in tray, Plasma Wayland
# handled via a single XWayland restart), and makes the generic AIstone/MSI
# restore-tool settings optional at build time, because nahimic.helprace.com
# is unreachable from some networks. ASRock-only builds do not need them.
pkgname=nahimic-linux
pkgver=0.3.3
pkgrel=4
pkgdesc='Nahimic speaker effects and equalizer for supported Linux laptops and desktop boards (ASRock port)'
arch=('x86_64')
url='https://github.com/wearzdk/nahimic-linux'
license=('MIT' 'LicenseRef-Nahimic')
depends=('python' 'pyside6' 'wine' 'pipewire' 'pipewire-pulse' 'wireplumber>=0.5' 'libpulse' 'systemd' 'hicolor-icon-theme' 'bash' 'glibc')
makedepends=('mingw-w64-gcc' 'pkgconf' 'cabextract' 'git' 'innoextract')
install=nahimic-linux.install
options=('!strip' '!debug')
source=("$pkgname::git+$url#tag=v$pkgver"
        'nahimic-apo4-4.14.cab::https://catalog.s.download.windowsupdate.com/d/msdownload/update/driver/drvs/2025/06/5038101c-bafc-45b6-b338-d086f6877059_8ba41f8fd185d004f674891c937eb6b815acb443.cab'
        'nahimic-asrock-apo4.zip::https://download.asrock.com/Utility/Others/Nahimic(v1.10.11_APO4).zip'
        '0001-Add-ASRock-desktop-Line-Out-support.patch'
        '0002-Add-vendor-EQ-presets.patch'
        '0003-Add-system-tray-icon.patch'
        'nahimic-linux.install')
noextract=('nahimic-apo4-4.14.cab' 'nahimic-asrock-apo4.zip')
sha256sums=('SKIP'
            'f3b1aae44826f85045cff8e085367dd8b48ffb3510e074adf552cc8eafbf8bde'
            '262bc2f10e8594197394f70759eb365238dba7270219089d3916083452111e29'
            'e3d84605f4f89ac7f8d872fab844eed3788263ca95fdfbef9734eb8adae29175'
            '8b9571226ad84d6ee031bd951334b979ac45cc27d60156c9ae73297a1cf34e4e'
            'dcc0d59d4aa77a60f737f9dd1bc35f375b63e7ad38ab65d3af8265feecbcff32'
            'f8206f52fd53c647e8ab1707545d5d977a2dd1495ba76b1f24abdb4faf35ce6e')

prepare() {
    git -C "$srcdir/$pkgname" checkout -f
    git -C "$srcdir/$pkgname" clean -fd
    patch -d "$srcdir/$pkgname" -Np1 -i "$srcdir/0001-Add-ASRock-desktop-Line-Out-support.patch"
    patch -d "$srcdir/$pkgname" -Np1 -i "$srcdir/0002-Add-vendor-EQ-presets.patch"
    patch -d "$srcdir/$pkgname" -Np1 -i "$srcdir/0003-Add-system-tray-icon.patch"

    python "$srcdir/$pkgname/packaging/extract_runtime.py" \
        "$srcdir/nahimic-apo4-4.14.cab" "" \
        "$srcdir/nahimic-asrock-apo4.zip" "$srcdir/runtime"
}
build() {
    make -C "$srcdir/$pkgname"
    strip --strip-unneeded "$srcdir/$pkgname/bin/pulse_state"
}
package() {
    make -C "$srcdir/$pkgname" DESTDIR="$pkgdir" install
    install -d "$pkgdir/usr/share/nahimic-linux"
    cp -r "$srcdir/runtime/vendor" "$srcdir/runtime/factory-asrock" "$srcdir/runtime/eqpresets" "$pkgdir/usr/share/nahimic-linux/"
}
