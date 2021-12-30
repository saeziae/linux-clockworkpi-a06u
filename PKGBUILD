# ClockworkPI A06 (based on linux-aarch64 from Manjaro ARM)
# Maintainer (upstream): Dan Johansen <strit@manjaro.org>
# Maintainer (this port): Max Fierke <max@maxfierke.com>
# Contributor (upstream): Kevin Mihelich <kevin@archlinuxarm.org>

pkgbase=linux-clockworkpi-a06
_srcname=linux-5.15
_kernelname=${pkgbase#linux}
_desc="Kernel for ClockworkPI A06"
pkgver=5.15.12
pkgrel=3
arch=('aarch64')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'dtc')
options=('!strip')
source=("http://www.kernel.org/pub/linux/kernel/v5.x/${_srcname}.tar.xz"
        "http://www.kernel.org/pub/linux/kernel/v5.x/patch-${pkgver}.xz"
        '0001-net-smsc95xx-Allow-mac-address-to-be-set-as-a-parame.patch'
        '0023-drm-rockchip-support-gamma-control-on-RK3399.patch' # From list: https://patchwork.kernel.org/project/linux-arm-kernel/cover/20211019215843.42718-1-sigmaris@gmail.com/
        '0024-Bluetooth-btsdio-Do-not-bind-to-non-removable-BCM4345-and-BCM43455.patch' # From list: https://patchwork.kernel.org/project/bluetooth/patch/20211020130023.196651-1-kmcopper@danwin1210.me/
        '0001-arm64-dts-clockworkpi-a06-dts.patch' # Potentially upstreamable, needs cleanup
        '0002-mfd-axp20x-add-clockworkpi-a06-power-support.patch' # Looks potentially incorrect. Probably not upstreamable
        '0004-gpu-drm-panel-add-cwd686-driver.patch' # Potentially upstreamable, needs cleanup
        '0005-video-backlight-add-ocp8178-driver.patch' # Potentially upstreamable, needs cleanup
        '0006-fix-rockchip-mipi-dsi-display-init-timeouts.patch' # From list: https://patchwork.kernel.org/project/linux-rockchip/list/?series=554547
        'config'
        'linux.preset'
        '60-linux.hook'
        '90-linux.hook')
md5sums=('071d49ff4e020d58c04f9f3f76d3b594'
         '3d03f763d15593dc2cc47222a0a2da13'
         '9e6b7f44db105fef525d715213dce7cf'
         'e2f08e3bc6d1b36e7000233abab1bfc7'
         'a897b51be2d05ddb5b7b1a7a7f5a5205'
         '0568bb52d70c17508766676ce896b7d1'
         'fc826c917102f2f2d16690fe9322464f'
         'f547a9ebf80ab8c76b64b523d2c66db8'
         '3203d018422505068fc22b909df871aa'
         '873658be357da087e5bc4f8d3a1e9c8c'
         'ecfafed2184ce310f9c0f219d501fec9'
         '86d4a35722b5410e3b29fc92dae15d4b'
         'ce6c81ad1ad1f8b333fd6077d47abdaf'
         '3dc88030a8f2f5a5f97266d99b149f77')

prepare() {
  cd ${_srcname}

  # add upstream patch
  patch -Np1 -i "${srcdir}/patch-${pkgver}"

  # ALARM patches
  patch -Np1 -i "${srcdir}/0001-net-smsc95xx-Allow-mac-address-to-be-set-as-a-parame.patch"             #All
  
  # Manjaro ARM Patches
  patch -Np1 -i "${srcdir}/0023-drm-rockchip-support-gamma-control-on-RK3399.patch"                     #RK3399
  patch -Np1 -i "${srcdir}/0024-Bluetooth-btsdio-Do-not-bind-to-non-removable-BCM4345-and-BCM43455.patch" #Bluetooth
  
  # ClockworkPI DevTerm A06 patches
  patch -Np1 -i "${srcdir}/0001-arm64-dts-clockworkpi-a06-dts.patch"                    # DTS
  patch -Np1 -i "${srcdir}/0002-mfd-axp20x-add-clockworkpi-a06-power-support.patch"     # Battery/Charger
  #patch -Np1 -i "${srcdir}/0003-snd-codecs-add-es8323-driver.patch"                     # Audio
  patch -Np1 -i "${srcdir}/0004-gpu-drm-panel-add-cwd686-driver.patch"                  # LCD
  patch -Np1 -i "${srcdir}/0005-video-backlight-add-ocp8178-driver.patch"               # Backlight
  patch -Np1 -i "${srcdir}/0006-fix-rockchip-mipi-dsi-display-init-timeouts.patch"      # Fix display suspend/resume

  cat "${srcdir}/config" > ./.config

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh
}

build() {
  cd ${_srcname}

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  # ... or manually edit .config

  # Copy back our configuration (use with new kernel version)
  #cp ./.config /var/tmp/${pkgbase}.config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  # build!
  unset LDFLAGS
  make ${MAKEFLAGS} Image modules
  # Generate device tree blobs with symbols to support applying device tree overlays in U-Boot
  make ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${pkgver}")
  conflicts=('kernel26' 'linux', 'linux-armv8' 'linux-aarch64')
  replaces=()
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")
  install=${pkgname}.install

  cd ${_srcname}

  KARCH=arm64

  # get kernel version
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs" dtbs_install
  cp arch/$KARCH/boot/Image "${pkgdir}/boot"

  # make room for external modules
  local _extramodules="extramodules-${_basekernel}${_kernelname}"
  ln -s "../${_extramodules}" "${pkgdir}/usr/lib/modules/${_kernver}/extramodules"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build and source links
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"


  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install mkinitcpio preset file
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "${_subst}" ../90-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers', 'linux-aarch64-headers')
  replaces=()

  cd ${_srcname}
  local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

  install -Dt "${_builddir}" -m644 Makefile .config Module.symvers
  install -Dt "${_builddir}/kernel" -m644 kernel/Makefile

  mkdir "${_builddir}/.tmp_versions"

  cp -t "${_builddir}" -a include scripts

  install -Dt "${_builddir}/arch/${KARCH}" -m644 arch/${KARCH}/Makefile
  install -Dt "${_builddir}/arch/${KARCH}/kernel" -m644 arch/${KARCH}/kernel/asm-offsets.s
  install -Dt "${_builddir}" -m644 vmlinux 

  cp -t "${_builddir}/arch/${KARCH}" -a arch/${KARCH}/include
  mkdir -p "${_builddir}/arch/arm"
  cp -t "${_builddir}/arch/arm" -a arch/arm/include

  install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
  install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

  # http://bugs.archlinux.org/task/13146
  install -Dt "${_builddir}/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # http://bugs.archlinux.org/task/20402
  install -Dt "${_builddir}/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "${_builddir}/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "${_builddir}/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # add xfs and shmem for aufs building
  mkdir -p "${_builddir}"/{fs/xfs,mm}

  # copy in Kconfig files
  find . -name Kconfig\* -exec install -Dm644 {} "${_builddir}/{}" \;

  # remove unneeded architectures
  local _arch
  for _arch in "${_builddir}"/arch/*/; do
    [[ ${_arch} == */${KARCH}/ || ${_arch} == */arm/ ]] && continue
    rm -r "${_arch}"
  done

  # remove documentation files
  rm -r "${_builddir}/Documentation"

  # remove now broken symlinks
  find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

  # strip scripts directory
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        $CHOST-strip $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        $CHOST-strip $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        $CHOST-strip $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        $CHOST-strip $STRIP_SHARED "$file" ;;
    esac
  done < <(find "${_builddir}" -type f -perm -u+x ! -name vmlinux -print0 2>/dev/null)
  $CHOST-strip $STRIP_STATIC "${_builddir}/vmlinux"
  
  # remove unwanted files
  find ${_builddir} -name '*.orig' -delete
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
