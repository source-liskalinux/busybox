# PKGBUILD For busybox

# Contributor: Janorovic Volkov <janorovicvolkov@gmail.com>
# Maintainer: Janorovic Volkov <janorovicvolkov@gmail.com>

pkgname=busybox
pkgver=1.36.1
pkgrel=1
pkgdesc="Utilities for rescue and embedded systems"
arch=("x86_64")
url="https://www.busybox.net"
license=('GPL-2.0-only')
makedepends=("ncurses" "musl" "kernel-headers-musl" "patch" "make")
source=("${url}/downloads/${pkgname}-${pkgver}.tar.bz2"
        "https://gitlab.archlinux.org/archlinux/packaging/packages/busybox/-/raw/main/config"
        "https://gitlab.archlinux.org/archlinux/packaging/packages/busybox/-/raw/main/extra_version.patch")
sha256sums=('SKIP'
            'SKIP'
            'SKIP')

prepare() {
    cd "${srcdir}/${pkgname}-${pkgver}"
    patch -p1 < "${srcdir}"/extra_version.patch
    patch -p1 < "${srcdir}"/fix-cbq-header.patch
}

build() {
    cd "${srcdir}/${pkgname}-${pkgver}"
    cp "${srcdir}/config" .config
    export KCONFIG_NOTIMESTAMP=1
    make CC=musl-gcc BB_EXTRA_VERSION="-${pkgrel}"
}

package() {
    cd "${srcdir}/${pkgname}-${pkgver}"
    install -Dm0755 busybox "${pkgdir}"/usr/bin/busybox
}

