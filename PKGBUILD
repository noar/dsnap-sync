# Maintainer: noar <spadrolelavie cat gmail dog com>
# forked from dsnap-sync by Ralf Zerres <ralf.zerres.de at gmail dot com>
pkgname=snap-back-git
_pkgname=snap-back
pkgver=0.6.7.r1.gc79d096
pkgrel=2
pkgdesc="Mirror btrfs trees on any or remote fs, using snapper snapshots/config"
arch=(any)
url="https://github.com/noar/snap-back"
license=('GPL')
conflicts=('snap-back')
provides=('snap-back')
backup=('etc/snapper/config-templates/snap-back')
makedepends=('git')
depends=('btrfs-progs' 'which' 'gawk' 'sed' 'snapper')
optdepends=('pv: progress bar during backup'
	    'libnotify: desktop notifications'
	    'openssh: sending snapshots to remote hosts'
	    'mbuffer: for buffering data streams'
	    'ionice: io scheduling class and priority'
	    'ltfs: for tape support (filesystem)'
	    'mtx: for tape support (media changer)' 
	    'jq: for tape support ("MediaPool" functionality)'
	    'attr: for tape volume name only (for now)')
#tobechecked ('dash' 'perl' 'systemd' 'util-linux')
#source=(${url}/releases/download/$pkgver/$pkgname-$pkgver.tar.gz{,.sig})
#source=(git+$(echo $url).git)
#source=("$_pkgname::git+https://github.com/noar/$_pkgname")
source=("git+https://github.com/noar/$_pkgname")
#validpgpkeys=('391BC244E24F20CE86E5779C8F7B6F930998B5B7')
sha512sums=('SKIP')

pkgver() {
  cd "$_pkgname"
  # cutting off 'v' prefix that presents in the git tag
  git describe --long | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

package() {
    cd $_pkgname
    make SNAPPER_CONFIG=/etc/conf.d/snapper DESTDIR=$pkgdir install
}
