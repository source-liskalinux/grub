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

prepare() {
  cd "${pkgname}-${pkgver}"
  if [[ -f bootstrap ]]; then
    ./bootstrap
  fi
}

build() {
  cd "${pkgname}-${pkgver}"
  for _target in "${_targets[@]}"
  do
    echo "===> Building target platform: ${_target}"
    mkdir -p "build-${_target}"
    cd "build-${_target}"
    local _platform _arch _target_cflags _target_ldflags
    case "${_target}" in
      i386-pc)
        _platform=pc
        _arch=i386
        _target_cflags="-m32 -O2 -pipe"
        _target_ldflags="-m32"
        ;;
      i386-efi)
        _platform=efi
        _arch=i386
        _target_cflags="-m32 -O2 -pipe"
        _target_ldflags="-m32"
        ;;
      x86_64-efi)
        _platform=efi
        _arch=x86_64
        _target_cflags="-O2 -pipe"
        _target_ldflags=""
        ;;
    esac
    export CFLAGS="${_target_cflags}"
    export CPPFLAGS=""
    export LDFLAGS="${_target_ldflags}"
    ../configure \
      --prefix=/usr \
      --sysconfdir=/etc \
      --sbindir=/usr/bin \
      --mandir=/usr/share/man \
      --infodir=/usr/share/info \
      --datarootdir=/usr/share \
      --disable-werror \
      --enable-grub-mkfont \
      --enable-grub-mount \
      --enable-device-mapper \
      --with-platform="${_platform}" \
      --target="${_arch}"
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
  echo "===> Stripping userland binaries in /usr/bin..."
  find "${pkgdir}/usr/bin" -type f -exec strip --strip-unneeded {} + 2>/dev/null || true
  rm -rf "${pkgdir}/usr/share/info"
  find "${pkgdir}" -type f -name "*.log" -exec rm -f {} +
}