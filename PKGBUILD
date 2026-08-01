# PKGBUILD For Liska Linux GRUB (Grand Unified Bootloader)

# Contributor: Janorovic Volkov <janorovicvolkov@gmail.com>
# Maintainer: Janorovic Volkov <janorovicvolkov@gmail.com>

pkgname=grub
pkgver=2.14
pkgrel=1
pkgdesc="GNU GRand Unified Bootloader (Liska Linux Edition - BIOS, EFI32, EFI64)"
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
_targets=(i386-pc i386-efi x86_64-efi)

prepare() {
  cd "${srcdir}/${pkgname}-${pkgver}"
  echo "===> Patching configure script to force -Ttext over --image-base...."
  sed -i 's/grub_cv_target_cc_ld_image_base=yes/grub_cv_target_cc_ld_image_base=no/g' configure
  sed -i 's/--image-base/-Ttext/g' configure
  local _10_linux="${srcdir}/${pkgname}-${pkgver}"
  echo "===> Configuring /etc/grub.d/10_linux...."
  sed -i 's|GNU/Linux|Linux|g' "${_10_linux}/util/grub.d/10_linux.in"
  sed -i 's|message="$(gettext_printf "Loading Linux %s ..." ${version})"|message="$(gettext_printf "Loading %s ...." ${os})"|g' "${_10_linux}/util/grub.d/10_linux.in"
  sed -i 's|title="$(gettext_printf "%s, with Linux %s (recovery mode)" "${os}" "${version}")"|title="$(gettext_printf "%s (recovery mode)" "${os}")"|g' "${_10_linux}/util/grub.d/10_linux.in"
  sed -i 's|title="$(gettext_printf "%s, with Linux %s" "${os}" "${version}")"|title="$(gettext_printf "%s" "${os}")"|g' "${_10_linux}/util/grub.d/10_linux.in"
}

build() {
  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS CC CXX LD
  unset TARGET_CFLAGS TARGET_CCASFLAGS TARGET_LDFLAGS TARGET_CPPFLAGS
  unset EXTRA_TARGET_CFLAGS EXTRA_TARGET_CCASFLAGS EXTRA_TARGET_LDFLAGS
  local _common_flags="-fcf-protection=none -Wa,-mx86-used-note=no"
  for _target in "${_targets[@]}"
  do
    echo "===> Preparing In-Tree Build Directory for: ${_target}"
    rm -rf "${srcdir}/grub-${_target}"
    cp -a "${srcdir}/${pkgname}-${pkgver}" "${srcdir}/grub-${_target}"
    cd "${srcdir}/grub-${_target}"
    local _opts=()
    local _ldflags="-Wl,--build-id=none -Wl,-z,noseparate-code -Wl,-z,ibt=off -Wl,-z,shstk=off"
    case "${_target}" in
      i386-pc)
        _opts+=(--enable-efiemu --with-platform="pc" --target="i386")
        _ldflags="-Wl,-Ttext,0x9000 ${_ldflags}"
        ;;
      i386-efi)
        _opts+=(--disable-efiemu --with-platform="efi" --target="i386")
        ;;
      x86_64-efi)
        _opts+=(--with-platform="efi" --target="x86_64")
        ;;
    esac
    echo "===> Running ./configure for ${_target}...."
    grub_cv_target_cc_ld_image_base=no ./configure \
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
        TARGET_LDFLAGS="${_ldflags}" \
        EXTRA_TARGET_CFLAGS="${_common_flags}" \
        EXTRA_TARGET_CCASFLAGS="${_common_flags}" \
        EXTRA_TARGET_LDFLAGS="-Wl,-z,ibt=off -Wl,-z,shstk=off"
    echo "===> Compiling GRUB for ${_target}...."
    make
  done
}

check() {
  echo "===> Running Validation Tests with grub-mkimage"
  echo "===> [TEST 1/3] Testing i386-pc Core Image Generation...."
  "${srcdir}/grub-i386-pc/grub-mkimage" \
    -d "${srcdir}/grub-i386-pc/grub-core" \
    -O i386-pc \
    -o /tmp/liska_test_bios.img \
    -p /boot/grub \
    biosdisk part_msdos ext2
  echo "===> [TEST 2/3] Testing i386-efi Image Generation...."
  "${srcdir}/grub-i386-efi/grub-mkimage" \
    -d "${srcdir}/grub-i386-efi/grub-core" \
    -O i386-efi \
    -o /tmp/liska_test_efi32.efi \
    -p /boot/grub \
    part_gpt fat ext2
  echo "===> [TEST 3/3] Testing x86_64-efi Image Generation...."
  "${srcdir}/grub-x86_64-efi/grub-mkimage" \
    -d "${srcdir}/grub-x86_64-efi/grub-core" \
    -O x86_64-efi \
    -o /tmp/liska_test_efi64.efi \
    -p /boot/grub \
    part_gpt fat ext2
  rm -f /tmp/liska_test_bios.img /tmp/liska_test_efi32.efi /tmp/liska_test_efi64.efi
  echo "===> [+] SUCCESS: All 3 Target Platforms Passed Validation!"
}

package() {
  for _target in "${_targets[@]}"
  do
    echo "===> Installing target platform: ${_target}"
    cd "${srcdir}/grub-${_target}"
    make DESTDIR="${pkgdir}" install
  done
  install -Dm644 "${srcdir}/grub.default" "${pkgdir}/etc/default/grub"
  echo "===> Stripping userland binaries in /usr/bin...."
  find "${pkgdir}/usr/bin" -type f -exec strip --strip-unneeded {} + 2>/devnull || true
  rm -rf "${pkgdir}/usr/share/info"
  find "${pkgdir}" -type f -name "*.log" -exec rm -f {} +
}
