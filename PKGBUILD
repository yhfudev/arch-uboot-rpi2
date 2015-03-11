# Maintainer: Yunhui Fu <yhfudev at gmail dot com>

pkgname=uboot-rpi2
pkgver=1.0.0
pkgrel=1
pkgdesc="Raspberry Pi 2 U-Boot"
arch=('i686' 'x86_64' 'arm')
url="https://github.com/yhfudev/arch-uboot-rpi2.git"
license=('GPL')

makedepends=(
    'git' 'bc' 'gcc-libs' 'bash'
    'ncurses' 'lzop' 'uboot-tools' # for kernel
    #'lib32-libstdc++5' 'lib32-zlib' # for 32 bit compiler
    'base-devel' 'abs' 'fakeroot'
    # 'kernel-package' # debian packages, include make-kpkg
    dtc
    )
#install="$pkgname.install"
#PKGEXT=.pkg.tar.xz

provides=('uboot-rpi2-git')
conflicts=('uboot-rpi2')

if [ 0 = 1 ]; then
# config for Raspberry Pi 1
ARCHITECTURE="armel"
FN_RPI_KERNEL=kernel.img
CONFIG_UBOOT="u-boot-rpi.config"
MAKE_UBOOT_CONFIG=rpi_defconfig
MAKE_UBOOT_DEVTREE=bcm2708-rpi-b-plus
else
# config for Raspberry Pi 2
ARCHITECTURE="armhf"
FN_RPI_KERNEL=kernel7.img
CONFIG_UBOOT="u-boot-rpi2.config"
PATCH_CONFIG_KERNEL="rpi-kernel-config-3.19.patch"
MAKE_UBOOT_CONFIG=rpi_2_defconfig
MAKE_UBOOT_DEVTREE=bcm2709-rpi-2-b
fi


DNSRC_UBOOT_HARDKERNEL=uboot-hardkernel-git
GITCOMMIT_UBOOT_HARDKERNEL=e7d4447d551ccba5d60be8b11697aa0ab49086c4

GITCOMMIT_LINUX=780e68130fba82a525b89e85f051c91b7a508e52
GITCOMMIT_UBOOT=xxxxx8130fba82a525b89e85f051c91b7a508e52
DNSRC_LINUX=linux-${GITCOMMIT_LINUX}
DNSRC_UBOOT=u-boot-${GITCOMMIT_UBOOT}
USE_GIT_REPO=1
DNSRC_UBOOT=u-boot-git
DNSRC_LINUX=linux-raspberrypi-git

source=(
        "kali-arm-build-scripts-git::git+https://github.com/yhfudev/kali-arm-build-scripts.git"
        "${DNSRC_UBOOT}::git+http://git.denx.de/u-boot.git" # "http://git.denx.de/u-boot/archive/${GITCOMMIT_UBOOT}.tar.gz"
        #"${DNSRC_UBOOT_HARDKERNEL}::git+https://github.com/hardkernel/u-boot.git" #"https://github.com/hardkernel/u-boot/archive/${GITCOMMIT_UBOOT_HARDKERNEL}.tar.gz"
        "tools-raspberrypi-git::git+https://github.com/raspberrypi/tools.git"
        )

md5sums=(
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         )
sha1sums=(
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         )

pkgver() {
    cd "${srcdir}/kali-arm-build-scripts-git"
    local ver="$(git show | grep commit | awk '{print $2}'  )"
    #printf "r%s" "${ver//[[:alpha:]]}"
    echo ${ver:0:7}
}

prepare_rpi2_toolchains () {
    echo "[DBG] in prepare_rpi2_toolchains() ..."
    # 32bit compiler
}

prepare_rpi2_uboot () {
    echo "[DBG] cd ${srcdir}/${DNSRC_UBOOT} ..."
    cd "${srcdir}/${DNSRC_UBOOT}"
    if [ "${USE_GIT_REPO}" = "1" ]; then
        git branch -a | grep "rpi_dev"
        if [ ! "$?" = "0" ]; then
            git remote add rpi-swarren https://github.com/swarren/u-boot.git
            git fetch --all
            git branch -a
            echo "for branch in \$(git branch -a | grep remotes | grep -v HEAD | grep -v master); do git branch --track \${branch##*/} \$branch ; done" | bash
            git fetch --all
            git checkout master
            if [ ! "$?" = "0" ]; then
                echo "Error in git master"
                exit 1
            fi
            git merge -s recursive -X theirs rpi_dev
            if [ ! "$?" = "0" ]; then
                echo "Error in git merge rpi_dev"
                exit 1
            fi
        fi
        git fetch --all
        git pull --all
    fi

    #make mrproper
    #make ${MAKE_CONFIG} # generate .config
    #cp ${srcdir}/${CONFIG_KERNEL} .config # or use ours
    #patch -p0 --no-backup-if-mismatch < ${srcdir}/${PATCH_CONFIG_KERNEL}
    #if [ ! "$?" = "0" ]; then
        #echo "error in patch ${PATCH_CONFIG_KERNEL}"
        #exit 1
    #fi
}

build_rpi2_uboot () {
    cd "${srcdir}/${DNSRC_UBOOT}"

    export ARCH=arm
    if [ "${ISCROSS}" = "1" ]; then
        # 32bit compiler
        export PATH=${DN_TOOLCHAIN_UBOOT}/gcc-linaro-arm-none-eabi-4.8-2014.04_linux/bin/:$PATH
        export CROSS_COMPILE=arm-none-eabi-
    else
        export CROSS_COMPILE=
        unset CROSS_COMPILE
    fi

    echo "[DBG] PATH=${PATH}"
    make -j $MACHINECORES ${MAKE_UBOOT_CONFIG}
    make -j $MACHINECORES

    #bcm2708-rpi-b-plus.dtb
    #bcm2708-rpi-b.dtb
    #bcm2709-rpi-2-b.dtb
    cp "${srcdir}/${DNSRC_LINUX}/arch/arm/boot/dts/"bcm270* "${srcdir}/${DNSRC_UBOOT}/arch/arm/dts/"
    make -j $MACHINECORES -s DEVICE_TREE=${MAKE_UBOOT_DEVTREE}
}

install_rpi2_uboot () {
    echo "install_rpi2_uboot"
    cd "${srcdir}/${DNSRC_UBOOT}"

    # replace the kernel.img
    mkdir -p "${pkgdir}/boot/"
    cp u-boot.bin ${pkgdir}/boot/${FN_RPI_KERNEL}
}

my0_getpath () {
  PARAM_DN="$1"
  shift
  #readlink -f
  DN="${PARAM_DN}"
  FN=
  if [ ! -d "${DN}" ]; then
    FN=$(basename "${DN}")
    DN=$(dirname "${DN}")
  fi
  cd "${DN}" > /dev/null 2>&1
  DN=$(pwd)
  cd - > /dev/null 2>&1
  echo "${DN}/${FN}"
}

my0_check_valid_path() {
    V=$(my0_getpath "$1")
    if [[ "${V}" = "" || "${V}" = "/" ]]; then
        echo "Error: not set path variable: $1"
        exit 1
    fi
}

my_setevn() {
    # setup environments
    MACHINE=${ARCHITECTURE}
    ISCROSS=1
    HW=$(uname -m)
    case ${HW} in
    armv5el)
        # Pi 1
        ISCROSS=0
        MACHINE=armel
        ;;
    armv7l)
        # Pi 2
        ISCROSS=0
        MACHINE=armhf
        ;;
    x86_64)
        ;;
    esac
    export MACHINEARCH="${MACHINE}"

    MACHINECORES=$(grep -c processor /proc/cpuinfo)
    if [ "$MACHINECORES" = "" ]; then
        MACHINECORES=2
    fi

    export FN_IMAGE="${srcdir}/${pkgname}-${pkgver}-${MACHINEARCH}.img"

    #export DN_TOOLCHAIN_UBOOT="${srcdir}/toolchains-uboot-${MACHINEARCH}"
    #export DN_TOOLCHAIN_KERNEL="${srcdir}/toolchains-kernel-${MACHINEARCH}"
    export DN_TOOLCHAIN_UBOOT="${srcdir}/"
    export DN_TOOLCHAIN_KERNEL="${srcdir}/"

    export ARCH=arm
    if [ "${ISCROSS}" = "1" ]; then
        if [ $(uname -m) = x86_64 ]; then
            export DN_TOOLCHAIN_KERNEL="${srcdir}/tools-raspberrypi-git/"
            export PATH=${DN_TOOLCHAIN_KERNEL}/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/:$PATH
            export CROSS_COMPILE=arm-linux-gnueabihf-
        else
            export DN_TOOLCHAIN_KERNEL="${srcdir}/tools-raspberrypi-git/"
            export PATH=${DN_TOOLCHAIN_KERNEL}/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/bin/:$PATH
            export CROSS_COMPILE=arm-linux-gnueabihf-
        fi
    else
        export CROSS_COMPILE=
        unset CROSS_COMPILE
    fi

    # http://elinux.org/RPi_U-Boot
    # contain a non-multilib libgcc that is compiled for a newer CPU architecture
    export USE_PRIVATE_LIBGCC=yes
}

prepare() {
    my_setevn

    prepare_rpi2_uboot

    echo "Build rootfs ..."
    cd ${srcdir}
    # create rootfs
    kali_rootfs_debootstrap
    echo "Build rootfs DONE!"
}

build() {
    my_setevn

    echo "Build U-Boot ..."
    build_rpi2_uboot
    echo "Build U-Boot DONE!"
}

package() {
    my_setevn

    cd ${srcdir}
    #make DESTDIR="$pkgdir/" install
    install_rpi2_uboot
}
