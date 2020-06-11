PCIE는 PCI을 대체하기위해 직렬 프로토콜을 지원한다



PCIE는 구성영역 256바이트에 추가 확장영역 4KB를 지원한다.



PCIe 주소지정(0000:XX:YY:Z)



lspci 명령을 통해 연결된 PCI 디바이스의 구성을 알 수있다.

bash>lspci
00:00.0 Host bridge: Intel Corporation 82852/82855 GM/GME/PM/GMV Processor to I/O
Controller (rev 02)
...
02:00.0 CardBus bridge: Texas Instruments PCI4510 PC card Cardbus Controller (rev 03)
...
03:00.0 Ethernet controller: Xircom Cardbus Ethernet 10/100 (rev 03)
03:00.1 Serial controller: Xircom Cardbus Ethernet + 56k Modem (rev 03)



0000-> 도메인, XX: PCI 버스번호, YY: PCI 디바이스 번호, Z: 기능

한도메인 256개 버스를 지원할 수있고 한 버스는 32개 디바이스를 지원할 수있다. 기능은 8개 까지 구현된다.



bash> lspci –t
-[0000:00]-+-00.0
+-00.1
+-00.3
+-02.0
+-02.1
+-1d.0
+-1d.1
+-1d.2
+-1d.7
+-1e.0-[0000:02-05]--+-[0000:03]-+-00.0
| | \-00.1
| \-[0000:02]-+-00.0
| +-00.1
| +-01.0
| \-02.0
+-1f.0

아래는 03:00:0 이 도메인으로 부터 어떤 경로로 연결되어 있는지 알려준다.

bash> ls /sys/devices/pci0000:00/0000:00:1e.0/0000:02:00.0/0000:03:00.0/
...
net:eth2 Ethernet
...
bash> ls /sys/devices/pci0000:00/0000:00:1e.0/0000:02:00.0/0000:03:00.1/
...
tty:ttyS1 Modem
...



구성영역(256 바이트)은 제조사와 기능을 식별하는 열쇠이다.

lspci-x 명령으로 구성영역을 확인 할 수있다.



bash> lspci –x

30: 00 00 00 00 dc 00 00 00 00 00 00 00 0b 01 00 00
Controller (rev 02)
00: 86 80 80 35 06 01 90 20 02 00 00 06 00 00 80 00
10: 08 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
20: 00 00 00 00 00 00 00 00 00 00 00 00 14 10 5c 05
30: 00 00 00 00 40 00 00 00 00 00 00 00 00 00 00 00
...
02:00.0 CardBus bridge: Texas Instruments PCI4510 PC card Cardbus Controller (rev 03)
00: 4c 10 44 ac 07 00 10 02 03 00 07 06 20 a8 82 00
10: 00 00 00 b0 a0 00 00 22 02 03 04 b0 00 00 00 f0
20: 00 f0 ff f1 00 00 00 d2 00 f0 ff d3 00 30 00 00
30: fc 30 00 00 00 34 00 00 fc 34 00 00 0b 01 00 05
...
03:00.0 Ethernet controller: Xircom Cardbus Ethernet 10/100 (rev 03)
00: 5d 11 03 00 07 00 10 02 03 00 00 02 00 40 80 00
10: 01 30 00 00 00 00 00 d2 00 08 00 d2 00 00 00 00
20: 00 00 00 00 00 00 00 00 07 01 00 00 5d 11 81 11
30: 00 00 00 00 dc 00 00 00 00 00 00 00 0b 01 14 28
03:00.1 Serial controller: Xircom Cardbus Ethernet + 56k Modem (rev 03)
00: 5d 11 03 01 03 00 10 02 03 02 00 07 00 00 80 00
10: 81 30 00 00 00 10 00 d2 00 18 00 d2 00 00 00 00
20: 00 00 00 00 00 00 00 00 07 02 00 00 5d 11 81 11
30: 00 00 00 00 dc 00 00 00 00 00 00 00 0b 01 00 00



아래와 같이 sysfs를 통해 PCI 구성영역을 덤프할 수도 있다.

bash> od -x /sys/devices/pci0000:00/0000:00:1e.0/0000:02:00.0/0000:03:00.1/config
0000000 115d 0003 0007 0210 0003 0200 4000 0080
0000020 3001 0000 0000 d200 0800 d200 0000 0000
0000040 0000 0000 0000 0000 0107 0000 115d 1181



구성영역은 아래와 같다



arch/arm64/boot/dts/freescale/fsl-ls1043a.dtsi

pcie@3400000 {
                        compatible = "fsl,ls1043a-pcie", "snps,dw-pcie";
                        reg = <0x00 0x03400000 0x0 0x00100000   /* controller registers */
                               0x40 0x00000000 0x0 0x00002000>; /* configuration space */
                        reg-names = "regs", "config";
                        interrupts = <0 117 0x4>, /* PME interrupt */
                                         <0 118 0x4>; /* aer interrupt */
                        interrupt-names = "pme", "aer";
                        #address-cells = <3>;
                        #size-cells = <2>;
                        device_type = "pci";
                        dma-coherent;
                        num-lanes = <4>;
                        bus-range = <0x0 0xff>;
                        ranges = <0x81000000 0x0 0x00000000 0x40 0x00010000 0x0 0x00010000   /* downstream I/O */
                                  0x82000000 0x0 0x40000000 0x40 0x40000000 0x0 0x40000000>; /* non-prefetchable memory */
                        msi-parent = <&msi>;
                        #interrupt-cells = <1>;
                        interrupt-map-mask = <0 0 0 7>;
                        interrupt-map = <0000 0 0 1 &gic 0 110 0x4>,
                                        <0000 0 0 2 &gic 0 110 0x4>,
                                        <0000 0 0 3 &gic 0 110 0x4>,
                                        <0000 0 0 4 &gic 0 110 0x4>;
                };




drivers/pci/host/pci-layerscape-ep.c

