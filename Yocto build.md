
FSL Yocto Project Community BSP - freescale.github.io 
i.MX 8M Quad Evaluation Kit Quick Start Guide (IMX8MQUADEVKQSG)- https://www.nxp.com/doc/IMX8MQUADEVKQSG

1. Host Setup

120GB �̻� �ϵ��ũ
Ubuntu 16.04 �̻�
SDL ��ġ ���� ������ �߻��� local.conf ���Ͽ� �Ʒ� �κ� �߰��� �� 
#PACKAGECONFIG_append_pn-qemu-native = " sdl"
#PACKAGECONFIG_append_pn-nativesdk-qemu = " sdl"
1-1.	Host Setup
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat libsdl1.2-dev

i.MX layers host packages for a Ubuntu 12.04 or 14.04 host setup are:

$sudo apt-get install libsdl1.2-dev xterm sed cvs subversion coreutils texi2html docbook-utils python-pysqlite2 help2man make gcc g++ desktop-file-utils libgl1-mesa-dev libglu1-mesa-dev mercurial autoconf automake groff curl lzop asciidoc

$sudo apt-get install u-boot-tools

1-2.	repo utility ��ġ

Repo�� ���� ����Ұ� ���� �� ������Ʈ������ ���� ���� �� ���ִ� Git ��� �����Դϴ�.
������ ������ ���� �ʿ�� �����ϴ�.
Repo�� Yocto ������Ʈ�� ����ȭ �� Ư���� �ſ� �� �����ϹǷ� ����ڰ� �ڽ��� ������ BSP�� ���� �߰� �� �� �ֽ��ϴ�.

$mkdir ~/bin
$curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$chmod a+x ~/bin/repo

.bashrc�� ~/bin�� �߰��϶� 
$vi ~/.bashrc  
export PATH=~/bin:$PATH




2. YOCTO ������Ʈ ��ġ

GIT ����
$git config --global user.name "Your Name"
$git config --global user.email "Your Email"
$git config ?list

i.MX Yocto Project Community BSP recipe layers �ٿ�ε��ϱ� 

$mkdir imx-yocto-bsp
$cd imx-yocto-bsp
$repo init -u https://source.codeaurora.org/external/imx/imx-manifest -b imx-linux-sumo -mimx-4.14.98-2.0.0_ga.xml
$repo sync

�Ʒ� ��ũ���� �� ������� �����Ǵ� ��� �޴��佺Ʈ�� Ȯ��

https://source.codeaurora.org/external/imx/imx-manifest/tree/?h=imx-linux-sumo

repo sync�� ���� �ֽ� �ڵ带 ������Ʈ �� �� �ִ�. ������ �߻��ϸ� .repo�� �����ϰ� �ٽ� repo �ʱ�ȭ�� ���� 

3. �̹��� ����

fsl-setup-release.sh ��ũ��Ʈ�� ���� �����

meta-freescale/conf/machine �� �����Ǵ� �ӽ�(����)����Ʈ�� Ȯ���� �� �ִ�. 

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
Distro�� �� �׷��� �鿣�� �����ӿ�ũ���� Frame Buffer, Wayland, XWayland, and X11 ���� �ִ�. 


<distro> 
? fsl-imx-x11 - X11 graphics are not supported on i.MX 8.
? fsl-imx-wayland - Wayland weston graphics.
? fsl-imx-xwayland - Wayland graphics and X11. X11 applications using EGL are not supported.
? fsl-imx-fb - Frame Buffer graphics - no X11 or Wayland. Frame Buffer is not supported on i.MX 8.

MACHINE�� ������ meta-freescale or meta-fsl-bsp-release/conf/machine �� ����Ʈ���� �����̴�. 

-b build-wayland �� ������丮 �̸��̴�.
���� ���丮�ȿ� conf �����ȿ� bblayers.conf �� local.conf ������ �ִµ� 
/conf/bblayers.conf �� ��Ÿ���̾���� ���Եȴ�. 
/conf/local.conf�� machine, distro ������ ���Եȴ�. ACCEPT_FSL_EULA�� ���ԵǴµ��� �̴� EULA�� �������� �ǹ��մϴ�. 

�̹��� �����Ƿ� ��ü �̹��� ���� 
$ bitbake core-image-minimal or $ bitbake fsl-image-qt5-validation-imx

<poky> core-image-minimal, core-image-base, core-image-sato, 
<meta-freescale-distro>fsl-image-machine-test,
<meta-fsl-bsp-release/imx/meta-sdk> fsl-image-validation-imx , fsl-image-qt5-validation-imx


U-BOOT ����
local.conf �� UBOOT_CONFIG �� ���������� U-BOOT Boot Ÿ���� ������ �� �ִ�. 

$ echo "UBOOT_CONFIG = \"emmc\"" >> conf/local.conf (U-boot EMMC)
$ MACHINE= imx8qmmek  bitbake -c deploy u-boot-imx

����ȯ�� �� ���۹��
source setup-environment build-wayland

To integrate Chromium into your rootfs and enable hardware accelerated rendering of WebGL 
Local.conf�� �Ʒ� �κ��� �߰��߽��ϴ�.
CORE_IMAGE_EXTRA_INSTALL += "chromium-ozone-wayland"


Multilib ���� 

Local.conf ���Ͽ� �Ʒ� �κ��� �����ϱ� �ٶ��ϴ�. 

MACHINE = imx8mqevk
# Define multilib target
require conf/multilib.conf
MULTILIBS = "multilib:lib32"
DEFAULTTUNE_virtclass-multilib-lib32 = "armv7athf-neon"
# Add the multilib packages to the image
IMAGE_INSTALL_append = "lib32-glibc lib32-libgcc lib32-libstdc++"

Jailhouse �� local.conf�� �Ʒ� �κ��� �߰��� 
DISTRO_FEATURES_append = " jailhouse"


run `run jh_netboot` or `jh_mmcboot`.
��� ���� 
#insmod jailhouse.ko
#./jailhouse enable imx8mq.cell


3. �̹��� ����
<build directory>/tmp/deploy/images �� .sdcard, Ext3, tar.bz2 
.sdcard ��  u-boot, Ŀ��, ��Ʈ���Ͻý����� ���Ե�


$ bunzip2 -dk -f <image_name>.sdcard.bz2
$ sudo dd if=<image name>.sdcard of=/dev/sd<partition> bs=1M conv=fsync



Package �� package group �߰� 

CORE_IMAGE_EXTRA_INSTALL += "<package_name1 package_name2>"


CORE_IMAGE_EXTRA_INSTALL += "ethtool��

IMAGE_INSTALL_append = " imx-test"
IMAGE_INSTALL_append += " net-tools"

IMAGE_INSTALL_append�� CORE_IMAGE_EXTRA_INSTALL ��ũ�ο� ������ �߰��ϴ� ��ũ���Դϴ�. 


