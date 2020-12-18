# Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Sébastien "Seblu" Luttringer <seblu@seblu.net>

pkgbase=qemu-patched
pkgname=(qemu-patched qemu-headless-patched qemu-arch-extra-patched qemu-headless-arch-extra-patched
         qemu-block-{iscsi,rbd,gluster}-patched qemu-guest-agent-patched)
pkgdesc="A generic and open source machine emulator and virtualizer"
pkgver=5.2.0
pkgrel=2
arch=(x86_64)
license=(GPL2 LGPL2.1)
url="https://wiki.qemu.org/"
_headlessdeps=(seabios gnutls libpng libaio numactl libnfs
               lzo snappy curl vde2 libcap-ng spice libcacard usbredir libslirp
               libssh zstd liburing ndctl dtc)
depends=(virglrenderer sdl2 vte3 libpulse libjack.so brltty "${_headlessdeps[@]}")
makedepends=(spice-protocol python ceph libiscsi glusterfs python-sphinx xfsprogs ninja)
source=(https://download.qemu.org/qemu-$pkgver.tar.xz
        qemu-guest-agent.service
        65-kvm.rules
        Add-hide-hypervisor-QMP-command.patch)
sha512sums=('bddd633ce111471ebc651e03080251515178808556b49a308a724909e55dac0be0cc0c79c536ac12d239678ae94c60100dc124be9b9d9538340c03a2f27177f3'
            '269c0f0bacbd06a3d817fde02dce26c99d9f55c9e3b74bb710bd7e5cdde7a66b904d2eb794c8a605bf9305e4e3dee261a6e7d4ec9d9134144754914039f176e4'
            'bdf05f99407491e27a03aaf845b7cc8acfa2e0e59968236f10ffc905e5e3d5e8569df496fd71c887da2b5b8d1902494520c7da2d3a8258f7fd93a881dd610c99'
            '9515ae7f503395cb9704c51fb015e91025da25d9ff262df1308b95637de98f268b6e12bebced764ce04b0228ad59838876df9fd7a23454b349e3bccbb4aca41c')
validpgpkeys=('CEACC9E15534EBABB82D3FA03353C9CEF108B584') # Michael Roth <flukshun@gmail.com>

case $CARCH in
  i?86) _corearch=i386 ;;
  x86_64) _corearch=x86_64 ;;
esac

prepare() {
  mkdir build-{full,headless}
  mkdir -p extra-arch-{full,headless}/usr/{bin,share/qemu}
  qemu_hd_replacement="RealIO LegitHD"
  qemu_dvd_replacement="REAL DVD-ROM"
  hypervisor_string_replacement="REAL"
  sed -i "s/QEMU HARDDISK/$qemu_hd_replacement/g" qemu-${pkgver}/hw/ide/core.c
  sed -i "s/QEMU HARDDISK/$qemu_hd_replacement/g" qemu-${pkgver}/hw/scsi/scsi-disk.c
  sed -i "s/QEMU DVD-ROM/$qemu_dvd_replacement/g" qemu-${pkgver}/hw/ide/core.c
  sed -i "s/QEMU DVD-ROM/$qemu_dvd_replacement/g" qemu-${pkgver}/hw/ide/atapi.c
  sed -i "s/QEMU PenPartner tablet/REAL PenPartner tablet/g" qemu-${pkgver}/hw/usb/dev-wacom.c
  sed -i 's/s->vendor = g_strdup("QEMU");/s->vendor = g_strdup("REAL");/g' qemu-${pkgver}/hw/scsi/scsi-disk.c
  sed -i "s/QEMU CD-ROM/$qemu_dvd_replacement/g" qemu-${pkgver}/hw/scsi/scsi-disk.c
  sed -i 's/padstr8(buf + 8, 8, "QEMU");/padstr8(buf + 8, 8, "REAL");/g' qemu-${pkgver}/hw/ide/atapi.c
  sed -i 's/QEMU MICRODRIVE/REAL MICRODRIVE/g' qemu-${pkgver}/hw/ide/core.c
  sed -i "s/KVMKVMKVM\\0\\0\\0/$hypervisor_string_replacement/g" qemu-${pkgver}/target/i386/kvm.c
  sed -i 's/"bochs"/"REAL"/g' qemu-${pkgver}/block/bochs.c
  sed -i 's/"BOCHS "/"REAL  "/g' qemu-${pkgver}/include/hw/acpi/aml-build.h
  sed -i 's/"BXPC"/"REAL"/g' qemu-${pkgver}/include/hw/acpi/aml-build.h
  sed -i 's/Microsoft Hv/$hypervisor_string_replacement/g' qemu-${pkgver}/target/i386/kvm.c
  patch -Np1 -F 3 -i ${srcdir}/Add-hide-hypervisor-QMP-command.patch -d qemu-${pkgver}
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

  ../qemu-${pkgver}/configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --libexecdir=/usr/lib/qemu \
    --extra-ldflags="$LDFLAGS" \
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
  provides=('qemu-headless' 'qemu')
  conflicts=('qemu-headless' 'qemu')
  replaces=(qemu-kvm)

  _package full
}

package_qemu-headless-patched() {
  pkgdesc="QEMU without GUI"
  depends=("${_headlessdeps[@]}")
  optdepends=('qemu-headless-arch-extra: extra architectures support')

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
