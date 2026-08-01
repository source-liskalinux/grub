# PKGBUILD For Liska Linux GRUB (Grand Unified Bootloader)

# Contributor: Janorovic Volkov <janorovicvolkov@gmail.com>
# Maintainer: Janorovic Volkov <janorovicvolkov@gmail.com>

pkgname=grub
pkgver=2.14
pkgrel=1
pkgdesc="GNU GRand Unified Bootloader (Liska Linux Edition - EFI32, EFI64)"
arch=('x86_64')
url="https://www.gnu.org/software/grub/"
license=('GPL-3.0-or-later')
depends=('bash' 'xz' 'gettext' 'device-mapper')
makedepends=('git' 'python' 'flex' 'bison' 'autogen' 'texinfo' 'help2man' 'freetype2' 'fuse3' 'gcc-multilib')
optdepends=('dosfstools' 'efibootmgr' 'freetype2' 'fuse3' 'mtools' 'os-prober')
provides=('grub')
conflicts=('grub')
backup=('etc/default/grub')
options=('!strip' '!debug' '!buildflags')
source=("https://ftp.gnu.org/gnu/grub/grub-${pkgver}.tar.xz" "grub.default")
sha256sums=('SKIP' 'SKIP')
_targets=(i386-efi x86_64-efi)

prepare() {
  cd "${srcdir}/${pkgname}-${pkgver}"
  echo "--> [PREPARE] Fix DejaVu font path directly in configure script...."
  sed -i 's|/usr/share/fonts/dejavu|/usr/share/fonts/dejavu /usr/share/fonts/TTF|g' configure
  local _10_linux="${srcdir}/${pkgname}-${pkgver}"
  echo "--> [PREPARE] Configuring /util/grub.d/10_linux.in...."
  sed -i 's|GNU/Linux|Linux|g' "${_10_linux}/util/grub.d/10_linux.in"
  sed -i 's|message="$(gettext_printf "Loading Linux %s ..." ${version})"|message="$(gettext_printf "Loading %s...." ${os})"|g' "${_10_linux}/util/grub.d/10_linux.in"
  sed -i 's|title="$(gettext_printf "%s, with Linux %s (recovery mode)" "${os}" "${version}")"|title="$(gettext_printf "%s (recovery mode)" "${os}")"|g' "${_10_linux}/util/grub.d/10_linux.in"
  sed -i 's|title="$(gettext_printf "%s, with Linux %s" "${os}" "${version}")"|title="$(gettext_printf "%s" "${os}")"|g' "${_10_linux}/util/grub.d/10_linux.in"
  echo "--> [PREPARE] Patch to detect Liska Linux initramfs images by grub-mkconfig...."
  sed -i 's|"initramfs-${version}.img"|"initramfs-liska.img" "initramfs-${version}.img"|g' "${_10_linux}/util/grub.d/10_linux.in"
  echo "--> [PREPARE] Make translations reproducible...."
  sed -i '1i /^PO-Revision-Date:/ d' po/*.sed
}

build() {
  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS CC CXX LD
  unset TARGET_CFLAGS TARGET_CCASFLAGS TARGET_LDFLAGS TARGET_CPPFLAGS
  unset EXTRA_TARGET_CFLAGS EXTRA_TARGET_CCASFLAGS EXTRA_TARGET_LDFLAGS
  local _common_flags="-fcf-protection=none -Wa,-mx86-used-note=no"
  for _target in "${_targets[@]}"
  do
    echo "--> [BUILD] Preparing In-Tree Build Directory for UEFI target: ${_target}"
    rm -rf "${srcdir}/grub-${_target}"
    cp -a "${srcdir}/${pkgname}-${pkgver}" "${srcdir}/grub-${_target}"
    cd "${srcdir}/grub-${_target}"
    local _opts=()
    case "${_target}" in
      i386-efi)
        _opts+=(--disable-efiemu --with-platform="efi" --target="i386")
        ;;
      x86_64-efi)
        _opts+=(--with-platform="efi" --target="x86_64")
        ;;
    esac
    echo "--> [BUILD] Running ./configure for ${_target}...."
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
        --enable-stack-protector=no \
        "${_opts[@]}" \
        TARGET_CFLAGS="-O2 -pipe -fno-stack-protector ${_common_flags} -Wno-error=discarded-qualifiers -Wno-error=maybe-uninitialized -Wno-error=attributes" \
        TARGET_CCASFLAGS="${_common_flags}" \
        TARGET_LDFLAGS="-Wl,--build-id=none -Wl,-z,noseparate-code" \
        EXTRA_TARGET_CFLAGS="${_common_flags}" \
        EXTRA_TARGET_CCASFLAGS="${_common_flags}"
    echo "--> [BUILD] Compiling GRUB for ${_target}...."
    make
  done
}

check() {
  echo "--> [CHECK] Running Validation Tests with grub-mkimage (2/2)"
  echo "--> [CHECK 1/2] Testing i386-efi Image Generation...."
  "${srcdir}/grub-i386-efi/grub-mkimage" \
    -d "${srcdir}/grub-i386-efi/grub-core" \
    -O i386-efi \
    -o /tmp/liska_test_efi32.efi \
    -p /boot/grub \
    part_gpt fat ext2
  echo "--> [CHECK 2/2] Testing x86_64-efi Image Generation...."
  "${srcdir}/grub-x86_64-efi/grub-mkimage" \
    -d "${srcdir}/grub-x86_64-efi/grub-core" \
    -O x86_64-efi \
    -o /tmp/liska_test_efi64.efi \
    -p /boot/grub \
    part_gpt fat ext2
  echo "--> [CHECK 2/2] Test completed, no errors found!"
  rm -f /tmp/liska_test_efi32.efi /tmp/liska_test_efi64.efi
  echo "--> [CHECK] All Target Platforms Passed Validation!"
}

package() {
  for _target in "${_targets[@]}"
  do
    echo "--> [PACKAGE] Installing target platform: ${_target}"
    cd "${srcdir}/grub-${_target}"
    make DESTDIR="${pkgdir}" install
  done
  install -Dm644 "${srcdir}/grub.default" "${pkgdir}/etc/default/grub"
  echo "--> [PACKAGE] Stripping userland binaries in /usr/bin...."
  find "${pkgdir}/usr/bin" -type f -exec strip --strip-unneeded {} + 2>/devnull || true
  rm -rf "${pkgdir}/usr/share/info"
  find "${pkgdir}" -type f -name "*.log" -exec rm -f {} +
}
