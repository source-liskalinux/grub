# PKGBUILD For Liska Linux GRUB (Grand Unified Bootloader)

# Contributor: Janorovic Volkov <janorovicvolkov@gmail.com>
# Maintainer: Janorovic Volkov <janorovicvolkov@gmail.com>

pkgname=grub
pkgver=2.14
pkgrel=1
pkgdesc="GNU GRand Unified Bootloader (Liska Linux Edition)"
arch=('x86_64')
url="https://www.gnu.org/software/grub/"
license=('GPL-3.0-or-later')
depends=('sh' 'xz' 'gettext' 'device-mapper')
makedepends=('git' 'python' 'flex' 'bison' 'autogen' 'texinfo' 'help2man' 'freetype2' 'fuse3' 'gcc-multilib')
optdepends=('freetype2' 'fuse3' 'dosfstools' 'efibootmgr' 'mtools' 'os-prober')
provides=('grub')
conflicts=('grub')
backup=('etc/default/grub')
options=('!strip' '!debug' '!buildflags')
source=("https://ftp.gnu.org/gnu/grub/grub-${pkgver}.tar.xz" "grub.default")
sha256sums=('SKIP' 'SKIP')
_targets=(i386-pc i386-efi x86_64-efi)

build() {
  unset CC CXX LD
  for _target in "${_targets[@]}"
  do
    echo "===> Preparing clean source tree for targeted platform: ${_target}"
    cd "${srcdir}"
    rm -rf "${pkgname}-${pkgver}-${_target}"
    cp -a "${pkgname}-${pkgver}" "${pkgname}-${pkgver}-${_target}"
    cd "${pkgname}-${pkgver}-${_target}"
    local _platform _arch
    case "${_target}" in
      i386-pc)
        _platform=pc
        _arch=i386
        ;;
      i386-efi)
        _platform=efi
        _arch=i386
        ;;
      x86_64-efi)
        _platform=efi
        _arch=x86_64
        ;;
    esac
    export CFLAGS="-O2 -pipe -fno-cf-protection -fno-stack-protector -Wno-error=discarded-qualifiers -Wno-error=maybe-uninitialized -Wno-error=attributes"
    export TARGET_CFLAGS="-O2 -pipe -fno-cf-protection -fno-stack-protector -Wno-error=discarded-qualifiers -Wno-error=maybe-uninitialized -Wno-error=attributes"
    export CPPFLAGS=""
    export LDFLAGS=""
    ./configure \
        --prefix="/usr" \
        --bindir="/usr/bin" \
        --sbindir="/usr/bin" \
        --mandir="/usr/share/man" \
        --infodir="/usr/share/info" \
        --datarootdir="/usr/share" \
        --sysconfdir="/etc" \
        --program-prefix="" \
        --with-bootdir="/boot" \
        --with-grubdir="grub" \
        --enable-boot-time \
        --enable-cache-stats \
        --disable-werror \
        --with-platform="${_platform}" \
        --target="${_arch}"
    make
  done
}

package() {
  for _target in "${_targets[@]}"
  do
    echo "===> Installing targeted platform: ${_target}"
    cd "${srcdir}/${pkgname}-${pkgver}-${_target}"
    make DESTDIR="${pkgdir}" install
  done
  install -Dm644 "${srcdir}/grub.default" "${pkgdir}/etc/default/grub"
  echo "===> Configuring /etc/grub.d/10_linux...."
  sed -i 's|GNU/Linux|Linux|g' "${pkgdir}/etc/grub.d/10_linux"
  sed -i 's|message="$(gettext_printf "Loading Linux %s ..." ${version})"|message="$(gettext_printf "Loading %s ..." ${os})"|g' "${pkgdir}/etc/grub.d/10_linux"
  sed -i 's|title="$(gettext_printf "%s, with Linux %s (recovery mode)" "${os}" "${version}")"|title="$(gettext_printf "%s (recovery mode)" "${os}")"|g' "${pkgdir}/etc/grub.d/10_linux"
  sed -i 's|title="$(gettext_printf "%s, with Linux %s" "${os}" "${version}")"|title="$(gettext_printf "%s" "${os}")"|g' "${pkgdir}/etc/grub.d/10_linux"
  echo "===> Stripping userland binaries in /usr/bin...."
  find "${pkgdir}/usr/bin" -type f -exec strip --strip-unneeded {} + 2>/devnull || true
  rm -rf "${pkgdir}/usr/share/info"
  find "${pkgdir}" -type f -name "*.log" -exec rm -f {} +
}
