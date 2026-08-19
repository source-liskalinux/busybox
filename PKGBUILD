# PKGBUILD For busybox

# Contributor: Janorovic Volkov <janorovicvolkov@gmail.com>
# Maintainer: Janorovic Volkov <janorovicvolkov@gmail.com>

pkgname=busybox
pkgver=1.36.1
pkgrel=1
pkgdesc="Statically-linked BusyBox multi-call binary for Liska Linux (used in the lkinit initramfs and as a general-purpose fallback toolset)"
arch=('x86_64')
url="https://www.busybox.net/"
license=('GPL-2.0-only')
depends=()
makedepends=('perl' 'musl' 'kernel-headers-musl')
options=('!strip' 'staticlibs')
source=("https://busybox.net/downloads/busybox-${pkgver}.tar.bz2")
sha256sums=('SKIP')
# Built against musl instead of glibc, on top of CONFIG_STATIC. Statically
# linking glibc is fragile in practice - functions like getpwnam/NSS still
# reach for dlopen() at runtime even in a "static" glibc binary, which can
# warn or outright fail. musl has no such caveat, so the resulting binary
# genuinely doesn't touch glibc at all - same resilience goal as init.rs
# being a self-contained Rust binary: this stays usable as a fallback tool
# even if the system's glibc is missing, corrupted, or not yet mounted.
_cc=musl-gcc

prepare() {
    cd "${srcdir}/busybox-${pkgver}"
    if ! command -v "${_cc}" >/dev/null 2>&1; then
        echo "!! [PREPARE] '${_cc}' not found - install the 'musl' package (provides musl-gcc)." >&2
        exit 1
    fi
    echo "--> [PREPARE] Generating BusyBox's full-featured default config...."
    make CC="${_cc}" defconfig
    _set_config() {
        local sym="$1"
        if grep -q "^${sym}=" .config; then
            sed -i "s/^${sym}=.*/${sym}=y/" .config
        elif grep -q "^# ${sym} is not set\$" .config; then
            sed -i "s/^# ${sym} is not set\$/${sym}=y/" .config
        else
            echo "${sym}=y" >> .config
        fi
    }
    echo "--> [PREPARE] Statically linking against musl, not glibc...."
    _set_config CONFIG_STATIC
    # "sh" applet: make it resolve to ash, matching what a bare `sh` on
    # PATH is expected to be everywhere else in the toolchain (init.rs
    # itself falls back to "/bin/sh" as a shell candidate).
    _set_config CONFIG_FEATURE_SH_IS_ASH
    # blkid: make sure TYPE="..." is actually emitted, since that's what
    # find_device_by_tag()'s manual parser and our earlier debugging
    # session both depended on seeing in the plain `blkid` listing.
    _set_config CONFIG_FEATURE_BLKID_TYPE
    echo "--> [PREPARE] Resolving any newly-exposed sub-options to their defaults...."
    yes "" | make CC="${_cc}" oldconfig
}

build() {
    cd "${srcdir}/busybox-${pkgver}"
    make CC="${_cc}" -j$(nproc)
}

package() {
    cd "${srcdir}/busybox-${pkgver}"
    echo "--> [PACKAGE] Verifying the binary is actually statically linked...."
    if command -v ldd >/dev/null 2>&1 && ldd ./busybox 2>&1 | grep -qv "not a dynamic executable"; then
        echo "!! [PACKAGE] busybox appears to be dynamically linked - static build failed!" >&2
        ldd ./busybox >&2
        exit 1
    fi
    echo "--> [PACKAGE] Installing multi-call binary to /usr/bin/busybox...."
    install -Dm755 busybox "${pkgdir}/usr/bin/busybox"
    install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
