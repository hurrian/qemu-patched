# Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Sébastien "Seblu" Luttringer <seblu@seblu.net>

pkgbase=qemu-patched
pkgname=(qemu-patched qemu-headless-patched qemu-arch-extra-patched qemu-headless-arch-extra-patched
         qemu-block-{iscsi,rbd,gluster}-patched qemu-guest-agent-patched)
pkgdesc="A generic and open source machine emulator and virtualizer"
pkgver=6.0.0
pkgrel=2
arch=(x86_64)
license=(GPL2 LGPL2.1)
url="https://wiki.qemu.org/"
_headlessdeps=(seabios gnutls libpng libaio numactl libnfs
               lzo snappy curl vde2 libcap-ng spice libcacard usbredir libslirp
               libssh zstd liburing ndctl dtc fuse3)
depends=(virglrenderer sdl2 vte3 libpulse libjack.so brltty "${_headlessdeps[@]}")
makedepends=(spice-protocol python ceph libiscsi glusterfs python-sphinx xfsprogs ninja)
source=(https://download.qemu.org/qemu-$pkgver.tar.xz
        build-most-modules-statically-hack.diff
        qemu-guest-agent.service
        65-kvm.rules
        qemu_hide_hv_qmp_disable.patch)
sha512sums=('ee3ff00aebec4d8891d2ff6dabe4e667e510b2a4fe3f6190aa34673a91ea32dcd2db2e9bf94c2f1bf05aa79788f17cfbbedc6027c0988ea08a92587b79ee05e4'
            '8721068fb968dbae62ceff71aa46eb4c2452c7fde95b87396b439f2f927ea84d2ee2c512264a9f28a5ccaf3096aacce052cebf209aaffd62a201b5bafb512002'
            '269c0f0bacbd06a3d817fde02dce26c99d9f55c9e3b74bb710bd7e5cdde7a66b904d2eb794c8a605bf9305e4e3dee261a6e7d4ec9d9134144754914039f176e4'
            'bdf05f99407491e27a03aaf845b7cc8acfa2e0e59968236f10ffc905e5e3d5e8569df496fd71c887da2b5b8d1902494520c7da2d3a8258f7fd93a881dd610c99'
            '5ab63f9679e7ce3484cbb89d81feade85442ef4563454900cc9f26d07aa3b592cf74b23ee50d88d67fe0fd4b63227a0ec4e160c24a1a5018207e0dbdaabc1176')

case $CARCH in
  i?86) _corearch=i386 ;;
  x86_64) _corearch=x86_64 ;;
esac

prepare() {
  mkdir build-{full,headless}
  mkdir -p extra-arch-{full,headless}/usr/{bin,share/qemu}

  cd qemu-$pkgver
  # https://bugs.launchpad.net/qemu/+bug/1910696
  # the patch comes from https://salsa.debian.org/qemu-team/qemu/-/blob/master/debian/patches/build-most-modules-statically-hack.diff
  patch -p1 < ../build-most-modules-statically-hack.diff

  # meme
  patch -p1 < ../qemu_hide_hv_qmp_disable.patch
}

build() {
  _build full \
    --audio-drv-list="pa alsa sdl jack"

  _build headless \
    --audio-drv-list= \
    --disable-sdl \
    --disable-gtk \
    --disable-vte \
    --disable-brlapi \
    --disable-opengl \
    --disable-virglrenderer
}

_build() (
  cd build-$1

  ../${pkgname}-${pkgver}/configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --libexecdir=/usr/lib/qemu \
    --smbd=/usr/bin/smbd \
    --enable-modules \
    --enable-sdl \
    --enable-slirp=system \
    --enable-xfsctl \
    "${@:2}"

  ninja
)

package_qemu-patched() {
  optdepends=('qemu-arch-extra-patched: extra architectures support')
  provides=(qemu-headless)
  conflicts=(qemu-headless)
  replaces=(qemu-kvm)

  _package full
}

package_qemu-headless-patched() {
  pkgdesc="QEMU without GUI"
  depends=("${_headlessdeps[@]}")
  optdepends=('qemu-headless-arch-extra-patched: extra architectures support')

  _package headless
}

_package() {
  optdepends+=('samba: SMB/CIFS server support'
               'qemu-block-iscsi-patched: iSCSI block support'
               'qemu-block-rbd-patched: RBD block support'
               'qemu-block-gluster-patched: glusterfs block support')
  install=qemu.install
  options=(!strip !emptydirs)

  DESTDIR="$pkgdir" ninja -C build-$1 install "${@:2}"

  # systemd stuff
  install -Dm644 65-kvm.rules "$pkgdir/usr/lib/udev/rules.d/65-kvm.rules"

  # remove conflicting /var/run directory
  cd "$pkgdir"
  rm -r var

  cd usr/lib

  # bridge_helper needs suid
  # https://bugs.archlinux.org/task/32565
  chmod u+s qemu/qemu-bridge-helper

  # remove split block modules
  rm qemu/block-{iscsi,rbd,gluster}.so

  cd ../bin

  # remove extra arch
  for _bin in qemu-*; do
    [[ -f $_bin ]] || continue

    case ${_bin#qemu-} in
      # guest agent
      ga) rm "$_bin"; continue ;;

      # tools
      edid|img|io|keymap|nbd|pr-helper|storage-daemon) continue ;;

      # core emu
      system-${_corearch}) continue ;;
    esac

    mv "$_bin" "$srcdir/extra-arch-$1/usr/bin"
  done

  cd ../share/qemu
  for _blob in *; do
    [[ -f $_blob ]] || continue

    case $_blob in
      # provided by seabios package
      bios.bin|bios-256k.bin|vgabios-cirrus.bin|vgabios-qxl.bin|\
      vgabios-stdvga.bin|vgabios-vmware.bin|vgabios-virtio.bin|vgabios-bochs-display.bin|\
      vgabios-ramfb.bin) rm "$_blob"; continue ;;

      # provided by edk2-ovmf package
      edk2-*) rm "$_blob"; continue ;;

      # iPXE ROMs
      efi-*|pxe-*) continue ;;

      # core blobs
      bios-microvm.bin|kvmvapic.bin|linuxboot*|multiboot.bin|sgabios.bin|vgabios*) continue ;;

      # Trace events definitions
      trace-events*) continue ;;
    esac

    mv "$_blob" "$srcdir/extra-arch-$1/usr/share/qemu"
  done

  # provided by edk2-ovmf package
  rm -r firmware

  cd ..
  if [ "$1" = headless ]; then rm -r {applications,icons}; fi
}

package_qemu-arch-extra-patched() {
  pkgdesc="QEMU for foreign architectures"
  depends=(qemu)
  provides=(qemu-headless-arch-extra)
  conflicts=(qemu-headless-arch-extra)
  options=(!strip)

  mv extra-arch-full/usr "$pkgdir"
}

package_qemu-headless-arch-extra-patched() {
  pkgdesc="QEMU without GUI, for foreign architectures"
  depends=(qemu-headless)
  options=(!strip)

  mv extra-arch-headless/usr "$pkgdir"
}

package_qemu-block-iscsi-patched() {
  pkgdesc="QEMU iSCSI block module"
  depends=(glib2 libiscsi)

  install -D build-full/block-iscsi.so "$pkgdir/usr/lib/qemu/block-iscsi.so"
}

package_qemu-block-rbd-patched() {
  pkgdesc="QEMU RBD block module"
  depends=(glib2 ceph-libs)

  install -D build-full/block-rbd.so "$pkgdir/usr/lib/qemu/block-rbd.so"
}

package_qemu-block-gluster-patched() {
  pkgdesc="QEMU GlusterFS block module"
  depends=(glib2 glusterfs)

  install -D build-full/block-gluster.so "$pkgdir/usr/lib/qemu/block-gluster.so"
}

package_qemu-guest-agent-patched() {
  pkgdesc="QEMU Guest Agent"
  depends=(gcc-libs glib2 libudev.so liburing)
  install=qemu-guest-agent.install

  install -D build-full/qga/qemu-ga "$pkgdir/usr/bin/qemu-ga"
  install -Dm644 qemu-guest-agent.service "$pkgdir/usr/lib/systemd/system/qemu-guest-agent.service"
  install -Dm755 "$srcdir/qemu-$pkgver/scripts/qemu-guest-agent/fsfreeze-hook" "$pkgdir/etc/qemu/fsfreeze-hook"
}

# vim:set ts=2 sw=2 et:
