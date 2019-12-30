i.MX 8M Quad에서는 아래 요소가 필요하다.

• imx-boot (built by imx-mkimage), which includes SPL, U-Boot, Arm Trusted Firmware, DDR firmware, and HDMI
firmware  
• Linux kernel image  
• A device tree file (.dtb) for the board being used.  
• A root file system (rootfs) for the particular Linux image  

imx-boot  
i.MX8의 부트로더로 Uboot, ARM Trusted Firmware, DCD file(8/8X), system controller firmware(8/8X), SPL(8M), DDR Firmware(8M), HDMI firmware(8M), SECO firmware(8/8x)  

SPL(Second Program loader)는 U-boot에 활성화 되며 TCML에서 실행되는 첫번째 레벨부트로더로 실행된다. DDR를 초기화하고 U-BOOT, U-BOOT DTB, Arm trusted firmware, TEE OS (optional) 를 메모리로 로딩한다.  

로딩후에는 Arm trusted firmware BL31 로 바로 점프하며 BL31은 BL32 (TEE OS) , BL33 (u-boot) 을 선택적으로 시작하여 커널 부팅으로 진행된다.  
imx-boot의 SPL에는  DDR 펌웨어가 포함되어 있어 Boot 롬에서 Arm Cortex-M4 TCML으로 SPL를 로딩할 수가 있다. 
U-Boot, U-Boot DTB, Arm Trusted firmware,  TEE OS(선택적)은 FIT이미지로 합쳐지고 imx-boot 이미지에 빌드된다. 


<Linux 커널이미지와 디바이스 트리> 

