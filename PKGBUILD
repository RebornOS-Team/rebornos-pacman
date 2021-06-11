# Based on the file created for Arch Linux by:
# Dan McGee <dan@archlinux.org>
# Dave Reisner <dreisner@archlinux.org>

# Maintainer: Philip Müller <philm@manjaro.org>
# Maintainer: Guinux <guillaume@manjaro.org>

# Note: since we include pacman-contrib with this package we have to build pacman twice
# Example: pacsort: error while loading shared libraries

pkgname=pacman
pkgver=6.0.0
_pkgver=1.4.0
pkgrel=1
pkgdesc="A library-based package manager with dependency support"
arch=('x86_64')
url="http://www.archlinux.org/pacman/"
license=('GPL')
groups=('base-devel')
depends=('bash' 'glibc' 'libarchive' 'curl' 'perl' 'gpgme' 'archlinux-keyring'
         'manjaro-keyring' 'pacman-mirrors>=4.1.0')
checkdepends=('python' 'fakechroot')
makedepends=('asciidoc' 'pacman>=6.0.0' 'meson' 'doxygen')
optdepends=('haveged: for pacman-init.service'
            'perl-locale-gettext: translation support in makepkg-template'
            'diffutils: for pacdiff'
            'findutils: for pacdiff --find'
            'mlocate: for pacdiff --locate'
            'sudo: privilege elevation for several scripts'
            'vim: default merge program for pacdiff')
provides=('pacman-contrib' 'pacman-init' 'libalpm.so')
conflicts=('pacman-contrib' 'pacman-init')
replaces=('pacman-contrib' 'pacman-init')
backup=(etc/pacman.conf etc/makepkg.conf)
install=pacman.install
options=('strip' 'debug')
validpgpkeys=('6645B0A8C7005E78DB1D7864F99FFE0FEAE999BD'  # Allan McRae <allan@archlinux.org>
              'B8151B117037781095514CA7BBDFFC92306B1121'  # Andrew Gregory (pacman) <andrew@archlinux.org>
              '5134EF9EAF65F95B6BB1608E50FB9B273A9D0BB5') # Johannes Löthberg <johannes@kyriasis.com>

source=(https://sources.archlinux.org/other/pacman/$pkgname-$pkgver.tar.xz{,.sig}
        pacman-6.0.0-fix-404-download.patch::https://git.archlinux.org/pacman.git/patch/?id=3401f9e142ac4c701cd98c52618cb13164f2146b
        pacman-6.0.0-fix-key-import-double-free.patch::https://git.archlinux.org/pacman.git/patch/?id=542910d684191eb7f25ddc5d3d8fe3060028a267
        https://git.archlinux.org/pacman-contrib.git/snapshot/pacman-contrib-$_pkgver.tar.gz
        0001-pactree-fix-compilation-with-pacman-6.patch
        pacman.conf
        makepkg.conf
        pacman-sync-first-option.patch
        etc-pacman.d-gnupg.mount
        pacman-init.service)
sha256sums=('004448085a7747bdc7a0a4dd5d1fb7556c6b890111a06e029ab088f9905d4808'
            'SKIP'
            'fe7e037e1b84bfa5bc401650d148c2a2e87d827705a6ec18a596ff5eea3cc0fd'
            'a51b57dd47818d97e29ac0b8604b4b409916bbe6029adfbb03ac7e7c2b2a0819'
            '8746f1352aaad990fe633be34f925d5ab8bd8a19a5f1274e72dbde426deb86bd'
            '774d27532a91e2fe490ccc8d21c2d1d4d2a2dbfc8678a8406abb8bb8f9e6626c'
            'a71fabbf3cce40c8df7b2d9897d01c77bcc51c692258224acf492a4440f0feb7'
            '9a1e997163e43f0ed178973317ef0110d2faa5bfcb1e64546072d64a14889a59'
            '8167155d3a3e15fc4a1b1e989fdb826779e7b3690a52e2ca9d307ae0b1550e1d'
            'b6d14727ec465bb66d0a0358163b1bbfafcb4eaed55a0f57c30aabafae7eed68'
            '65d8bdccdcccb64ae05160b5d1e7f3e45e1887baf89dda36c1bd44c62442f91b')
prepare() {
  cd $pkgname-$pkgver
  
  # Arch patches
  patch -p1 -i "$srcdir"/pacman-6.0.0-fix-404-download.patch
  patch -p1 -i "$srcdir"/pacman-6.0.0-fix-key-import-double-free.patch

  # Manjaro patches
  patch -Np1 < "$srcdir"/pacman-sync-first-option.patch

  cd $srcdir/pacman-contrib-$_pkgver
  patch --forward --strip=1 --input=../0001-pactree-fix-compilation-with-pacman-6.patch
  ./autogen.sh
}

build() {
  cd $pkgname-$pkgver

  meson --prefix=/usr \
        --buildtype=plain \
        -Ddoc=enabled \
        -Ddoxygen=enabled \
        -Dscriptlet-shell=/usr/bin/bash \
        -Dldconfig=/usr/bin/ldconfig \
        build

  meson compile -C build

  cd $srcdir/pacman-contrib-$_pkgver

  ./configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var
  make
}

check() {
  cd "$pkgname-$pkgver"
  meson test -C build
  cd ..
  
  make -C pacman-contrib-$_pkgver check
}

package() {
  cd "$pkgname-$pkgver"

  DESTDIR="$pkgdir" meson install -C build

  # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  install -m644 "$srcdir/pacman.conf" "$pkgdir/etc"
  install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"

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
