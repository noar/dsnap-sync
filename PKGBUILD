# Maintainer: No ar <spadrolelavie at gmail dot com>
# Arch repackage forked from Ralf Zerres <ralf.zerres.de at gmail dot com>
pkgname=dsnap-sync
pkgver=0.6.6-beta3
pkgrel=1
pkgdesc="Use snapper snapshots to backup to external drive"
arch=(any)
url="https://github.com/noar/dsnap-sync"
license=('GPL')
depends=('btrfs-progs' 'gawk' 'dash' 'openssh' 'sed' 'snapper' 'systemd')
optdepends=('attr' 'ionice' 'jq: for "MediaPool" functionality' 'libnotify' 'ltfs' 'mtx' 'perl' 'pv' 'util-linux' 'mbuffer')
#source=(${url}/releases/download/$pkgver/$pkgname-$pkgver.tar.gz{,.sig})
source=(${url}/archive/refs/tags/v$pkgver.tar.gz{,.asc})
validpgpkeys=('391BC244E24F20CE86E5779C8F7B6F930998B5B7')
sha512sums=('93ca63b0df07c6d480b62d4b76e7f2ccbae339fa30ef249b5b01e06b0efe1cf72a0d475c309eeae7ecc25b740d06f9f5c0d8fc1977bf9abc6b286ce63c05c5af' 'SKIP')

package() {
    cd $pkgname-$pkgver
    make SNAPPER_CONFIG=/etc/conf.d/snapper DESTDIR=$pkgdir install
}
