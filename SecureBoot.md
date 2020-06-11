
http://www.nxp.com/products/automotive-products/microcontrollers-and-processors/arm-mcus-and-mpus/i.mx-application-processors/i.mx-6-processors/i.mx-6solox-processors-heterogeneous-processing-with-arm-cortex-a9-and-cortex-m4-cores:i.MX6SX?tab=Documentation_Tab

·         Secure Boot on i.MX50, i.MX53, and i.MX 6 Series using HABv4

Configuring Secure JTAG for the i.MX 6 Series Family of Applications Processors (


sudo apt-get install openssl

cd cst-2.3.1

        chmod u+x linux64/* keys/*


 


 


        cd keys


 


        echo -n "12345674" > serial


        echo -en "cst_test\ncst_test" > key_pass.txt


 


        ./hab4_pki_tree.sh


        "n", "2048", "10", "4", "y"


 


        cd ../crts/


        ../linux64/srktool -h 4 -t SRK_1_2_3_4_table.bin -e SRK_1_2_3_4_fuse.bin -d sha256 -c ./SRK1_sha256_2048_65537_v3_ca_crt.pem,./SRK2_sha256_2048_65537_v3_ca_crt.pem,./SRK3_sha256_2048_65537_v3_ca_crt.pem,./SRK4_sha256_2048_65537_v3_ca_crt.pem -f 1


 


        hexdump -e '/4 "0x"' -e '/4 "%X""\n"' SRK_1_2_3_4_fuse.bin


        0xD2FB3CC8


        0x48DE41A4


        0xB2E0769F


        0xFA6B3B8C


        0x3D0D80AD


        0xAE9FBB9E


        0x22D0F3E9


        0x2C5ECA20


 


 


        fuse prog 3 0 0xD2FB3CC8

        fuse prog 3 1 0x48DE41A4

        fuse prog 3 2 0xB2E0769F

        fuse prog 3 3 0xFA6B3B8C










        fuse prog 3 4 0x3D0D80AD


        fuse prog 3 5 0xAE9FBB9E


        fuse prog 3 6 0x22D0F3E9


        fuse prog 3 7 0x2C5ECA20


        //RNG Trim fuses




        //fuse


        fuse prog 1 0 0x00100000   


 


        or


        //csf


        [Unlock]


        Engine = CAAM


        Features = RNG


 


        //u-boot include/configs/mx6q_sabresd.h


        #define CONFIG_SECURE_BOOT


 


        mkdir ../u-boot


        cp -p ../../../work/u-boot-imx/2015.04-r0/git/u-boot.imx  ../u-boot/





//parameter definition


ivt_self=$(hexdump -n 4 -s 20 -e '/4 "0x%08x\t" "\n"' u-boot.imx)


ivt_csf=$(hexdump -n 4 -s 24 -e '/4 "0x%08x\t" "\n"' u-boot.imx)


 


auth_len = ivt_csf - ivt_self


sig_len = auth_len + 0x2000


 


  


 


vi ../u-boot/u-boot.csf


[Header]


    Version = 4.1


    Hash Algorithm = sha256


    Engine Configuration = 0


    Certificate Format = X509


    Signature Format = CMS


 


[Install SRK]


    File = "../crts/SRK_1_2_3_4_table.bin"


    Source index = 0


 


[Install CSFK]


    File = "../crts/CSF1_1_sha256_2048_65537_v3_usr_crt.pem"


 


[Authenticate CSF]


 


[Unlock]


    Engine = CAAM


    Features = RNG


 


[Install Key]


    Verification index = 0


    Target index = 2


    File = "../crts/IMG1_1_sha256_2048_65537_v3_usr_crt.pem"


 


# Sign padded u-boot starting at the IVT through to the end with


# length = 0x4B000


# This covers the essential parts: IVT, boot data and DCD.


# Blocks have the following definition:


#    Image block start address on i.MX, Offset from start of image file,


#    Length of block in bytes, image data file


[Authenticate Data]


    Verification index = 2


    Blocks = ivt_self 0x0 auth_len "u-boot-pad.imx"


 

 

vi ../u-boot/habimagegen

 

------------- file content begin -------------

#! /bin/bash

echo "extend u-boot to auth_len..."

objcopy -I binary -O binary --pad-to auth_len --gap-fill=0x5A u-boot.imx u-boot-pad.imx

echo "generate csf data..."

../linux64/cst --o u-boot_csf.bin < u-boot.csf

echo "merge image and csf data..."

cat u-boot-pad.imx u-boot_csf.bin > u-boot-signed.imx

echo "extend final image to sig_len..."

 

objcopy -I binary -O binary --pad-to sig_len --gap-fill=0x5A u-boot-signed.imx u-boot-signed-pad.imx

 

echo "u-boot-signed-pad.imx is ready"

 

 

 

 

 

 

 

U-Boot> hab_status

Secure boot disabled

 

HAB Configuration: 0xf0, HAB State: 0x66

No HAB Events Found!

 

 

U-Boot> fuse prog 0 6 0x2    - SEC_CONFIG flag set




 


폐설정모드(fusing상태)에서도 mfgtool를 사용할 수 있습니다.


Signed DCD Table은 내부 램(OCRAM)영역인 0x00910000 에 SDP 명령과 DCD_WRITE  으로 로드하고


IVT의 DCD Table pointer는 중복설정을 피하기 위해 clear 됩니다.


boot_data 에 코드는 로드됩니다.


 


 


Csf 파일 수정


[Authenticate Data]


Verification index = 2


Blocks = 0x10800400 0x400 0x2BC00 "u-boot-pad.bin"


 


[Authenticate Data]


Verification index = 2


Blocks = 0x00910000 0x430 0x2E0 "u-boot-pad.bin"


 


 


 


Make Script 작성


My_code를  이미지 이름으로 대체하시기 바랍니다.


 


#!/bin/bash


PROG_NAME=my_code


# ${PROG_NAME} padded up to 0x2C000 where the CSF will be added later


objcopy -I binary -O binary --pad-to 0x2C000 --gap-fill=0xff ${PROG_NAME}.bin ${PROG_NAME}_padded.bin


# DCD address must be cleared for signature, as mfgtool will clear it.


./mod_4_mfgtool.sh clear_dcd_addr ${PROG_NAME}_padded.bin


# generate the signatures, certificates, … in the CSF binary


../linux/cst --o ${PROG_NAME}_csf.bin < ${PROG_NAME}.csf


# DCD address must be set for mfgtool to localize the DCD table.


./mod_4_mfgtool.sh set_dcd_addr ${PROG_NAME}_padded.bin


# gather ${PROG_NAME} + its CSF


cat ${PROG_NAME}_padded.bin ${PROG_NAME}_csf.bin > ${PROG_NAME}_tmp.bin


# padding to get a file with size like specified in the IVT


objcopy -I binary -O binary --pad-to 0x22000 --gap-fill=0xff ${PROG_NAME}_tmp.bin ${PROG_NAME}_signed.bin


# remove temporary file


rm ${PROG_NAME}_tmp.bin


 


 


mod_4_mfgtool.sh


#!/bin/bash


# DCD address must be cleared for signature, as mfgtool will clear it.


if [ "$1" == "clear_dcd_addr" ]; then


# store the DCD address


dd if=$2 of=dcd_addr.bin bs=1 count=4 skip=1036


# generate a NULL address for the DCD


dd if=/dev/zero of=zero.bin bs=1 count=4


# replace the DCD address with the NULL address


dd if=zero.bin of=$2 seek=1036 bs=1 conv=notrunc


fi


# DCD address must be set for mfgtool to localize the DCD table.


if [ "$1" == "set_dcd_addr" ]; then


# restore the DCD address with the original address


dd if=dcd_addr.bin of=$2 seek=1036 bs=1 conv=notrunc


rm zero.bin


fi


 


SRK 및 SEC_CONFIG이 Fusing이 안되었다면 UCL.XML에서 아래와 같이 적용해 보시기 바랍니다.


 


<LIST name="MX6Q Sabre-lite SRK Hash" desc="SRK hash fuse programming">


<CMD type="find" body="Recovery" timeout="180"/>


<CMD type="boot" body="Recovery" file ="u-boot-mx6q-sabrelite.bin" >Loading uboot.</CMD>


<CMD type="load" file="uImage" address="0x10800000"


loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" >Doing Kernel.</CMD>


<CMD type="load" file="initramfs.cpio.gz.uboot" address="0x10C00000"


loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" >Doing Initramfs.</CMD>


<CMD type="jump" > Jumping to OS image. </CMD>


<CMD type="find" body="Updater" timeout="180"/>


<!-- ***** Caution - running this xml script with the fuse burning commands uncommented


***** in the Mfg tool permanently burns fuses. once completed this operation cannot


***** be undone!


-->


<CMD type="push" body="$ echo 0xfdf28547 > /sys/fsl_otp/HW_OCOTP_SRK0">Burn Word 0 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ echo 0x270d6ac6 > /sys/fsl_otp/HW_OCOTP_SRK1">Burn Word 1 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ echo 0xee44ad7b > /sys/fsl_otp/HW_OCOTP_SRK2">Burn Word 2 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ echo 0x058b0724 > /sys/fsl_otp/HW_OCOTP_SRK3">Burn Word 3 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ echo 0x49da1948 > /sys/fsl_otp/HW_OCOTP_SRK4">Burn Word 4 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ echo 0xb4374a3f > /sys/fsl_otp/HW_OCOTP_SRK5">Burn Word 5 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ echo 0xffefed48 > /sys/fsl_otp/HW_OCOTP_SRK6">Burn Word 6 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ echo 0x4247c04f > /sys/fsl_otp/HW_OCOTP_SRK7">Burn Word 7 of SRK hash field in OTP </CMD>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK0"/>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK1"/>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK2"/>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK3"/>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK4"/>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK5"/>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK6"/>


<CMD type="push" body="$ cat /sys/fsl_otp/HW_OCOTP_SRK7"/>


</LIST>


</UCL>


 


/////////////////////////////////////////////////////////////////////////////////


Mfgtool용 u-boot.imx 파일을 이용하여 작업들을 하셔야 합니다.


Mk_secure_u-boot 스크립트는 동일하게 사용하시기 바랍니다.


 


u-boot.imx의 사이즈에 맞는 u-boot.csf 이 생성이 되며 생성된 csf 파일에 아래


부분 아래에 ( 아래 값들은  생성된 u-boot.csf의  block address/offset/length 값을 유지합니다)


[Authenticate Data]


Verification index = 2


Blocks = 0x10800400 0x400 0x2BC00 "u-boot-pad.bin",


 


아래 부분을 추가해 주시기 바랍니다. syntax 에러가 발생하지 않습니다. 값들은 아래 붉은색을 유지하시기 바랍니다.


[Authenticate Data]


Verification index = 2


Blocks = 0x00910000 0x430 0x2E0 " u-boot-pad.bin "


Signed DCD Table를  내부 램(OCRAM)영역인 0x00910000 에 load하기위함입니다.


 


 


어플리케이션에서 가이드하는 아래 스크립트는 Mk_secure_u-boot로 생성한 Habimagegen 에


mod_4_mfgtool.sh를 스크립트를 활용하여 IVT의 DCD Table pointer를 clear 하기위해 추가된 스크립트입니다.


 


#!/bin/bash


PROG_NAME=my_code


# ${PROG_NAME} padded up to 0x2C000 where the CSF will be added later


objcopy -I binary -O binary --pad-to 0x2C000 --gap-fill=0xff ${PROG_NAME}.bin ${PROG_NAME}_padded.bin


# DCD address must be cleared for signature, as mfgtool will clear it.


./mod_4_mfgtool.sh clear_dcd_addr ${PROG_NAME}_padded.bin


# generate the signatures, certificates, … in the CSF binary


../linux/cst --o ${PROG_NAME}_csf.bin < ${PROG_NAME}.csf


# DCD address must be set for mfgtool to localize the DCD table.


./mod_4_mfgtool.sh set_dcd_addr ${PROG_NAME}_padded.bin


# gather ${PROG_NAME} + its CSF


cat ${PROG_NAME}_padded.bin ${PROG_NAME}_csf.bin > ${PROG_NAME}_tmp.bin


# padding to get a file with size like specified in the IVT


objcopy -I binary -O binary --pad-to 0x22000 --gap-fill=0xff ${PROG_NAME}_tmp.bin ${PROG_NAME}_signed.bin


# remove temporary file


rm ${PROG_NAME}_tmp.bin


 


 


mod_4_mfgtool.sh


#!/bin/bash


# DCD address must be cleared for signature, as mfgtool will clear it.


if [ "$1" == "clear_dcd_addr" ]; then


# store the DCD address


dd if=$2 of=dcd_addr.bin bs=1 count=4 skip=1036


# generate a NULL address for the DCD


dd if=/dev/zero of=zero.bin bs=1 count=4


# replace the DCD address with the NULL address


dd if=zero.bin of=$2 seek=1036 bs=1 conv=notrunc


fi


# DCD address must be set for mfgtool to localize the DCD table.


if [ "$1" == "set_dcd_addr" ]; then


# restore the DCD address with the original address


dd if=dcd_addr.bin of=$2 seek=1036 bs=1 conv=notrunc


rm zero.bin


fi


Habimagegen의 주소와 사이즈값들은 그대로 유지하시고 Mod_4_mfgtool.sh 를 사용하여 dcd table pointer를 clear하시기 바랍니다.
