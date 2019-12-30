
FSL Yocto Project Community BSP - freescale.github.io 
i.MX 8M Quad Evaluation Kit Quick Start Guide (IMX8MQUADEVKQSG)- https://www.nxp.com/doc/IMX8MQUADEVKQSG

1. Host Setup

120GB 이상 하드디스크
Ubuntu 16.04 이상
SDL 설치 에서 문제가 발생시 local.conf 파일에 아래 부분 추가갈 것 
#PACKAGECONFIG_append_pn-qemu-native = " sdl"
#PACKAGECONFIG_append_pn-nativesdk-qemu = " sdl"

1-1.	Host Setup
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat libsdl1.2-dev

i.MX layers host packages for a Ubuntu 12.04 or 14.04 host setup are:

$sudo apt-get install libsdl1.2-dev xterm sed cvs subversion coreutils texi2html docbook-utils python-pysqlite2 help2man make gcc g++ desktop-file-utils libgl1-mesa-dev libglu1-mesa-dev mercurial autoconf automake groff curl lzop asciidoc

$sudo apt-get install u-boot-tools

1-2.	repo utility 설치

Repo는 여러 저장소가 포함 된 프로젝트를보다 쉽게 관리 할 수있는 Git 기반 도구입니다.
동일한 서버에 있을 필요는 없습니다.
Repo는 Yocto 프로젝트의 계층화 된 특성을 매우 잘 보완하므로 사용자가 자신의 계층을 BSP에 쉽게 추가 할 수 있습니다.

$mkdir ~/bin
$curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$chmod a+x ~/bin/repo

.bashrc에 ~/bin를 추가하라 
$vi ~/.bashrc  
export PATH=~/bin:$PATH




2. YOCTO 프로젝트 설치

GIT 설정
$git config --global user.name "Your Name"
$git config --global user.email "Your Email"
$git config ?list

i.MX Yocto Project Community BSP recipe layers 다운로드하기 

$mkdir imx-yocto-bsp
$cd imx-yocto-bsp
$repo init -u https://source.codeaurora.org/external/imx/imx-manifest -b imx-linux-sumo -mimx-4.14.98-2.0.0_ga.xml
$repo sync

아래 링크에서 이 릴리즈에서 지원되는 모든 메니페스트를 확인

https://source.codeaurora.org/external/imx/imx-manifest/tree/?h=imx-linux-sumo

repo sync를 통해 최신 코드를 업데이트 할 수 있다. 문제가 발생하면 .repo를 제거하고 다시 repo 초기화를 진행 

3. 이미지 빌드

fsl-setup-release.sh 스크립트를 통해 진행됨

meta-freescale/conf/machine 에 지원되는 머신(보드)리스트를 확인할 수 있다. 

? imx6qpsabreauto
? imx6qpsabresd
? imx6ulevk
? imx6ulz14x14evk
? imx6ull14x14evk
? imx6ull9x9evk
? imx6dlsabreauto
? imx6dlsabresd
? imx6qsabreauto
? imx6qsabresd
? imx6slevk
? imx6solosabreauto
? imx6solosabresd
? imx6sxsabresd
? imx6sxsabreauto
? imx6sllevk
? imx7dsabresd
? imx7ulpevk
? imx8qmmek
? imx8qxpmek
? imx8mqevk
? imx8mmevk

$ DISTRO= fsl-imx-wayland  MACHINE=imx8qmmek source fsl-setup-release.sh -b build-wayland
Distro는 각 그래픽 백엔드 프레임워크으로 Frame Buffer, Wayland, XWayland, and X11 등이 있다. 


<distro> 
? fsl-imx-x11 - X11 graphics are not supported on i.MX 8.
? fsl-imx-wayland - Wayland weston graphics.
? fsl-imx-xwayland - Wayland graphics and X11. X11 applications using EGL are not supported.
? fsl-imx-fb - Frame Buffer graphics - no X11 or Wayland. Frame Buffer is not supported on i.MX 8.

MACHINE은 위에서 meta-freescale or meta-fsl-bsp-release/conf/machine 에 리스트업된 보드이다. 

-b build-wayland 은 빌드디렉토리 이름이다.
빌드 디렉토리안에 conf 폴더안에 bblayers.conf 와 local.conf 파일이 있는데 
/conf/bblayers.conf 는 메타레이어들이 포함된다. 
/conf/local.conf는 machine, distro 설정이 포함된다. ACCEPT_FSL_EULA도 포함되는데는 이는 EULA를 수용함을 의미합니다. 

이미지 레시피로 전체 이미지 빌드 
$ bitbake core-image-minimal or $ bitbake fsl-image-qt5-validation-imx

<poky> core-image-minimal, core-image-base, core-image-sato, 
<meta-freescale-distro>fsl-image-machine-test,
<meta-fsl-bsp-release/imx/meta-sdk> fsl-image-validation-imx , fsl-image-qt5-validation-imx


U-BOOT 빌드
local.conf 에 UBOOT_CONFIG 를 변경함으로 U-BOOT Boot 타입을 지정할 수 있다. 

$ echo "UBOOT_CONFIG = \"emmc\"" >> conf/local.conf (U-boot EMMC)
$ MACHINE= imx8qmmek  bitbake -c deploy u-boot-imx

빌드환경 재 시작방법
source setup-environment build-wayland

To integrate Chromium into your rootfs and enable hardware accelerated rendering of WebGL 
Local.conf에 아래 부분을 추가했습니다.
CORE_IMAGE_EXTRA_INSTALL += "chromium-ozone-wayland"


Multilib 지원 

Local.conf 파일에 아래 부분을 적성하기 바랍니다. 

MACHINE = imx8mqevk
# Define multilib target
require conf/multilib.conf
MULTILIBS = "multilib:lib32"
DEFAULTTUNE_virtclass-multilib-lib32 = "armv7athf-neon"
# Add the multilib packages to the image
IMAGE_INSTALL_append = "lib32-glibc lib32-libgcc lib32-libstdc++"

Jailhouse 는 local.conf에 아래 부분을 추가후 
DISTRO_FEATURES_append = " jailhouse"


run `run jh_netboot` or `jh_mmcboot`.
모듈 적재 
#insmod jailhouse.ko
#./jailhouse enable imx8mq.cell


3. 이미지 전개
<build directory>/tmp/deploy/images 에 .sdcard, Ext3, tar.bz2 
  
.sdcard 은  u-boot, 커널, 루트파일시스템이 포함됨


$ bunzip2 -dk -f <image_name>.sdcard.bz2
$ sudo dd if=<image name>.sdcard of=/dev/sd<partition> bs=1M conv=fsync



Package 및 package group 추가 

CORE_IMAGE_EXTRA_INSTALL += "<package_name1 package_name2>"


CORE_IMAGE_EXTRA_INSTALL += "ethtool”

IMAGE_INSTALL_append = " imx-test"
IMAGE_INSTALL_append += " net-tools"

IMAGE_INSTALL_append는 CORE_IMAGE_EXTRA_INSTALL 매크로에 내용을 추가하는 매크로입니다. 


