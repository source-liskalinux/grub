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
  cd "${pkgname}-${pkgver}"
  unset CFLAGS CPPFLAGS CXXFLAGS LDFLAGS TARGET_CFLAGS TARGET_LDFLAGS TARGET_CPPFLAGS CC CXX LD
  for _target in "${_targets[@]}"
  do
    echo "===> Building target platform: ${_target}"
    local _arch="${_target%%-*}"
    local _platform="${_target#*-}"
    mkdir -p "build-${_target}"
    cd "build-${_target}"
    ../configure \
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
        --enable-stack-protector=no \
        --with-platform="${_platform}" \
        --target="${_arch}" \
        EXTRA_TARGET_CFLAGS="-fno-cf-protection"
    make
    cd "${srcdir}/${pkgname}-${pkgver}"
  done
}

package() {
  cd "${pkgname}-${pkgver}"
  for _target in "${_targets[@]}"
  do
    echo "===> Installing target platform: ${_target}"
    cd "build-${_target}"
    make DESTDIR="${pkgdir}" install
    cd "${srcdir}/${pkgname}-${pkgver}"
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
