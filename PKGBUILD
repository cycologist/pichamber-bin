# Maintainer: Matthias von Davier <mvondavier@gmail.com>
# Contributor: debtap 3.6.3 (initial metadata extraction)

pkgname=pichamber-bin
_pkgname=PiChamber
pkgver=1.0.7
pkgrel=1
pkgdesc='A Pi-native workspace for coding sessions on desktop (prebuilt Electron app)'
arch=('x86_64' 'aarch64')
url='https://github.com/RyderAsKing/PiChamber'
license=('MIT')
depends=('gtk3' 'nss' 'alsa-lib' 'libsecret' 'libnotify' 'at-spi2-core' 'xdg-utils')
optdepends=('libdbusmenu-glib: system tray menu support')
provides=("${pkgname%-bin}")
conflicts=("${pkgname%-bin}")
options=('!strip' '!debug' '!emptydirs')
install="${pkgname}.install"

source_x86_64=("${url}/releases/download/v${pkgver}/${_pkgname}-${pkgver}-linux-amd64.deb"
               "pichamber-license::${url}/raw/v${pkgver}/LICENSE")
source_aarch64=("${url}/releases/download/v${pkgver}/${_pkgname}-${pkgver}-linux-arm64.deb"
                "pichamber-license::${url}/raw/v${pkgver}/LICENSE")

sha256sums_x86_64=('af19f7f8315026b914219943afbb52123c2b2cf629b407ad1b614124dd020244'
                   '6efeeb8befe1280ed29b0770f1d916dc591a86ec807131deb21f5ba82ddbbe4c')
sha256sums_aarch64=('26c0af1e3e56be4562f3fade94ab7a19c7508e3de829bdd23a3c5b24095bc361'
                    '6efeeb8befe1280ed29b0770f1d916dc591a86ec807131deb21f5ba82ddbbe4c')

package() {
	local _deb
	if [[ "$CARCH" == x86_64 ]]; then
		_deb="${_pkgname}-${pkgver}-linux-amd64.deb"
	else
		_deb="${_pkgname}-${pkgver}-linux-arm64.deb"
	fi

	# Unpack the Debian control/data members, then the payload itself.
	bsdtar -xf "$_deb" data.tar.*
	bsdtar -xf data.tar.* -C "$pkgdir"

	# The .deb relies on update-alternatives; on Arch a plain symlink
	# tracked by pacman is the correct equivalent.
	install -d "$pkgdir/usr/bin"
	ln -s /opt/PiChamber/pichamber "$pkgdir/usr/bin/pichamber"

	install -Dm644 pichamber-license "$pkgdir/usr/share/licenses/${pkgname}/LICENSE"
}
