$ sudo apt-get update
$ sudo apt-get upgrade
$sudo apt-get install ssh vim cscope ctags
$sudo apt-get install tftpd-hpa tftp-hpa
$sudo apt-get install nfs-kernel-server nfs-common portmap
$sudo apt-get install samba cifs-utils
 
 
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat libsdl1.2-dev
 
$ sudo apt-get install libsdl1.2-dev xterm sed cvs subversion coreutils texi2html docbook-utils python-pysqlite2 help2man make gcc g++ desktop-file-utils libgl1-mesa-dev libglu1-mesa-dev mercurial autoconf automake groff curl lzop asciidoc
 
$ sudo apt-get install u-boot-tools
 
//$sudo apt-get install tftpd-hpa tftp-hpa
$sudo vi /etc/default/tftpd-hpa
 
RUN_DAEMON=”yes”
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/home/samuel/fsl-release-bsp-4.1.15/build-imx7dsabresd/tmp/deploy/images/imx6ulevk"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="—secure —create"
 
 
$sudo chmod 777 /home/samuel/fsl-release-bsp-4.1.15/build-imx7dsabresd/tmp/deploy/images/imx6ulevk
$sudo /etc/init.d/tftpd-hpa restart
 
 
//$sudo apt-get install nfs-kernel-server nfs-common portmap
$sudo vi /etc/exports
/opt/nfsroot  192.168.1.*(rw,no_root_squash,no_all_squash,async,no_subtree_check)
$ sudo /etc/init.d/nfs-kernel-server restart
 
#mount -t nfs -o nolock 192.168.1.98:/opt/nfsroot /mnt/nfs
 
 
//$sudo apt-get install samba smbfs
$sudo vi /etc/samba/smb.conf 
[samuel]
comment = samuel samba
path = /home/samuel
writeable = yes
guest ok = no
create mask = 0644
directory mask = 0755
 
$ sudo smbpasswd -a samuel
$ sudo /etc/init.d/smbd restart
 
$sudo  ifconfig eth1 192.168.56.101 up
\\192.168.56.101\samuel
 
ctags -R or make tags
ctags -R --language-force=c++ --extra=+q --fields=+i *.cpp *.h
 
find ./ -name *.[chS] -print > cscope.files
find ./ -type f \( -name "*.[chS]" -o -name "*.dts" -o -name "Makefile*" \) -print > cscope.files
cs add cscope.out
 
sudo mv ~/tool/taglist_44/doc/taglist.txt /usr/share/vim/vim73/doc/
sudo mv ~/tool/taglist_44/plugin/taglist.vim /usr/share/vim/vim73/plugin/
 
 
 
 
$ mkdir ~/bin
$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
$ vi ~/.bashrc
export PATH=~/bin:$PATH
$ source ~/.bashrc
 
$ git config --global user.name "samuel park"
$ git config --global user.email "samuel.park@avnet.com"
 
$ git config --list
 
$ mkdir fsl-release-bsp
cd fsl-release-bsp
//$ repo init -u git://git.freescale.com/imx/fsl-arm-yocto-bsp.git -b imx-3.14.52-1.1.0_ga
$ repo init -u git://git.freescale.com/imx/fsl-arm-yocto-bsp.git -b imx-morty
//$ repo init -u git://git.freescale.com/imx/fsl-arm-yocto-bsp.git -b imx-morty -m imx-4.9.11-1.0.0_ga.xml
$ repo sync
 
 
1. Bitbake
 
$ DISTRO=fsl-imx-fb MACHINE=imx6ull14x14evk source fsl-setup-release.sh -b build-fb
 
//meta-fsl-bsp-release/imx/meta-sdk/conf/distro
//conf/machine <-meta-fsl-arm , meta-fsl-bsp-release
 
$ bitbake core-image-minimal
 
 
 
2. HOB
$ DISTRO=fsl-imx-fb MACHINE=imx6ull14x14evk source fsl-setup-release.sh -b build-fb
 
 
find ./ -type f \( -name "*.[chS]" -o -name "*.dts" -o -name "*.dtsi" -o -name "Makefile*" -o -name "makefile*" -o -name "*.mak" -o -name "*.mk" \) -print > cscope.files
 
 
bitbake-layers show-recipes "*-image-*"
 
bitbake -g core-image-minimal
cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' | awk '{print $1}' | sort | uniq > core-image-minimal.txt
 
 
bitbake –g hdparm
cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' | awk '{print $1}' | sort | uniq > hdparm-dep.txt
 
bitbake –g diffstat
cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' | awk '{print $1}' | sort | uniq >  diffstat.txt
 
bitbake –g fsl-image-gui
cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' | awk '{print $1}' | sort | uniq > fsl-image-gui.txt
 
diff -y --suppress-common-lines fsl-image-gui.txt diffstat.txt | grep -v \< | gawk '{print $NF}'
 
 
bitbake –g udisks
cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' | awk '{print $1}' | sort | uniq > udisk.txt
bitbake –g fsl-image-gui
cat pn-depends.dot | grep -v -e '-native' | grep -v digraph | grep -v -e '-image' |  awk '{print $1}' | sort | uniq > fsl-image-gui.txt
 
diff -y --suppress-common-lines fsl-image-gui.txt udisks.txt | grep -v \< |grep -v \> | gawk '{print $NF}'
 
bitbake core-image-base -c populate-sdk
