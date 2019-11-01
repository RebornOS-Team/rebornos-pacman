# Based on the file created for Arch Linux by:
# Dan McGee <dan@archlinux.org>
# Dave Reisner <dreisner@archlinux.org>

# Maintainer: Philip Müller <philm@manjaro.org>
# Maintainer: Guinux <guillaume@manjaro.org>

# Note: since we include pacman-contrib with this package we have to build pacman twice
# Example: pacsort: error while loading shared libraries

pkgname=pacman
pkgver=5.2.1
_pkgver=1.2.0
_commit=
pkgrel=1
pkgdesc="A library-based package manager with dependency support"
arch=('i686' 'x86_64')
url="http://www.archlinux.org/pacman/"
license=('GPL')
groups=('base' 'base-devel')
depends=('bash' 'glibc' 'libarchive' 'curl' 'perl' 'gpgme' 'archlinux-keyring'
         'manjaro-keyring' 'pacman-mirrors>=4.1.0')
checkdepends=('python' 'fakechroot')
makedepends=('asciidoc' 'pacman>=5.2' 'meson')
optdepends=('haveged: for pacman-init.service'
            'perl-locale-gettext: translation support in makepkg-template')
provides=('pacman-contrib' 'pacman-init' 'libalpm.so')
conflicts=('pacman-contrib' 'pacman-init')
replaces=('pacman-contrib' 'pacman-init')
backup=(etc/pacman.conf etc/makepkg.conf)
install=pacman.install
options=('emptydirs' 'strip' 'debug')
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD'  # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121'  # Andrew Gregory (pacman) <andrew@archlinux.org>
              '5134EF9EAF65F95B6BB1608E50FB9B273A9D0BB5') # Johannes Löthberg <johannes@kyriasis.com>
source=(https://sources.archlinux.org/other/pacman/$pkgname-$pkgver.tar.gz{,.sig}
        #https://git.archlinux.org/pacman.git/snapshot/pacman-$_commit.tar.gz
        https://git.archlinux.org/pacman-contrib.git/snapshot/pacman-contrib-$_pkgver.tar.{gz,asc}
        pacman.conf.i686
        pacman.conf.x86_64
        makepkg.conf
        pacman-sync-first-option.patch
        etc-pacman.d-gnupg.mount
        pacman-init.service)

prepare() {
  #mv $srcdir/$pkgname-$_commit $srcdir/$pkgname-$pkgver
  mkdir -p $pkgname-$pkgver/build

  cd $srcdir/$pkgname-$pkgver

  # Manjaro patches
  patch -p1 -i $srcdir/pacman-sync-first-option.patch

  cd build

  meson --prefix=/usr \
        --buildtype=plain \
        -Ddoc=enabled \
        -Duse-git-version=false \
        -Dscriptlet-shell=/usr/bin/bash \
        -Dldconfig=/usr/bin/ldconfig \
        ..

#  ./configure --prefix=/usr --sysconfdir=/etc \
#    --localstatedir=/var --enable-doc \
#    --with-scriptlet-shell=/usr/bin/bash \
#    --with-ldconfig=/usr/bin/ldconfig

  cd $srcdir/pacman-contrib-$_pkgver

  ./autogen.sh
  ./configure \
      --prefix=/usr \
      --sysconfdir=/etc \
      --localstatedir=/var
}

build() {
#  cd $srcdir/$pkgname-$pkgver
#  make V=1

  cd $srcdir/$pkgname-$pkgver/build
  ninja

  cd $srcdir/pacman-contrib-$_pkgver
  make
}

check() {
#  make -C "$srcdir/$pkgname-$pkgver" check
  cd $pkgname-$pkgver/build
  ninja test

  make -C "$srcdir/pacman-contrib-$_pkgver" check
}

package() {
#  cd $srcdir/$pkgname-$pkgver
#  make DESTDIR=$pkgdir install

  cd $srcdir/$pkgname-$pkgver/build
  DESTDIR="$pkgdir" ninja install

  # install Arch specific stuff
  install -dm755 $pkgdir/etc
  install -m644 $srcdir/pacman.conf.$CARCH $pkgdir/etc/pacman.conf
  
  case "$CARCH" in
    i686)    
      mycarch="i686"
      mychost="i686-pc-linux-gnu"
      myflags="-march=i686"
      ;;
    x86_64)
      mycarch="x86_64"
      mychost="x86_64-pc-linux-gnu"
      myflags="-march=x86-64"
      ;;
  esac
  install -m644 $srcdir/makepkg.conf $pkgdir/etc/
  # set things correctly in the default conf file
  sed -i $pkgdir/etc/makepkg.conf \
    -e "s|@CARCH[@]|$mycarch|g" \
    -e "s|@CHOST[@]|$mychost|g" \
    -e "s|@CARCHFLAGS[@]|$myflags|g"

  # install pacman-init
  install -dm755 $pkgdir/usr/lib/systemd/system/
  install -m644 $srcdir/etc-pacman.d-gnupg.mount $pkgdir/usr/lib/systemd/system/etc-pacman.d-gnupg.mount
  install -m644 $srcdir/pacman-init.service $pkgdir/usr/lib/systemd/system/pacman-init.service

  cd $srcdir/pacman-contrib-$_pkgver

  make DESTDIR="$pkgdir" install

  # replace rankmirrors
  rm "$pkgdir/usr/bin/rankmirrors"
  ln -sfv "/usr/bin/pacman-mirrors" "$pkgdir/usr/bin/rankmirrors"
}

sha256sums=('1930c407265fd039cb3a8e6edc82f69e122aa9239d216d9d57b9d1b9315af312'
            'SKIP'
            '317f53819e35647a19138cb0d68e16206af4a80f52115a7cd622c4a367f914b7'
            'SKIP'
            '7e0aa0144d9677ce4fa9e4a53d3007e8e6d3b96ce61639e65a2cd91e37f1664b'
            'b6eb7e06c60f599dc3a1474828a4e8ee79f7c08dfe51cdbd8835b005e6079fa9'
            'a50cd13ca5774f2bcfffd416abf0bca35fa525aba0626d7054863e6b7f1cdf93'
            '8167155d3a3e15fc4a1b1e989fdb826779e7b3690a52e2ca9d307ae0b1550e1d'
            'b6d14727ec465bb66d0a0358163b1bbfafcb4eaed55a0f57c30aabafae7eed68'
            '65d8bdccdcccb64ae05160b5d1e7f3e45e1887baf89dda36c1bd44c62442f91b')
