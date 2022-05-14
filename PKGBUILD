# Maintainer: No ar <spadrolelavie at gmail dot com>
# Arch repackage forked from Ralf Zerres <ralf.zerres.de at gmail dot com>
pkgname=dsnap-sync-git
_pkgname=dsnap-sync
pkgver=v0.6.5.102.r6.g0320108
pkgrel=1
pkgdesc="Use snapper snapshots to backup to external drive"
arch=(any)
conflicts=('dsnap-sync')
provides=('dsnap-sync')
url="https://github.com/noar/$_pkgname"
license=('GPL')
provides=('dsnap-sync')
conflicts=('dsnap-sync')
makedepends=('git')
depends=('btrfs-progs' 'gawk' 'dash' 'openssh' 'sed' 'snapper' 'systemd')
optdepends=('attr' 'ionice' 'jq: for "MediaPool" functionality' 'libnotify' 'ltfs' 'mtx' 'perl' 'pv' 'util-linux' 'mbuffer')
#source=(${url}/releases/download/$pkgver/$pkgname-$pkgver.tar.gz{,.sig})
#source=(${url}/archive/refs/tags/v$pkgver.tar.gz{,.asc})
source=(git+$(echo $url).git)
validpgpkeys=('391BC244E24F20CE86E5779C8F7B6F930998B5B7')
sha512sums=('SKIP')
#sha512sums=('93ca63b0df07c6d480b62d4b76e7f2ccbae339fa30ef249b5b01e06b0efe1cf72a0d475c309eeae7ecc25b740d06f9f5c0d8fc1977bf9abc6b286ce63c05c5af' 'SKIP')

pkgver() {
  cd "$_pkgname"
  git describe --long --tags | sed 's/\([^-]*-g\)/r\1/;s/-/./g'
}

package() {
#    cd $pkgname-$pkgver
    cd "$_pkgname"
    make SNAPPER_CONFIG=/etc/conf.d/snapper DESTDIR=$pkgdir install
}
