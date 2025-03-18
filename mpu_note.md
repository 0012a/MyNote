
# 我的MPU筆記fanyu

## 目錄
- [MPU安裝環境](#mpu安裝環境)
- [MPU-NXP-BSP筆記](#mpu-nxp-bsp筆記)<!-- 英文需要注意小寫 -->
- [md and html範例](#md-and-html範例)

---

## MPU安裝環境
  <details>
    <summary>TI</summary>

  https://plausible-tangerine-5fd.notion.site/Git-FoxconnGerrit-4a8e4aadf5fc4279abb6e67ab7752344

  http://192.168.3.173:3000/WOI8kIdET0eyq1IEKHegBQ?view

  https://plausible-tangerine-5fd.notion.site/Ti-Yocto-Building-GLA-a783ad490998465f8262bd7cc3b1ffaf

  DFU網址  http://192.168.3.173:3000/fDHYPQWIRzWdWJes5lmmMA?view

  SD卡燒錄EMMC  http://192.168.3.173:3000/b570FV6gRUa6x-E8eTKf0Q

```markdown
查看現在的磁區
---
root@am62xx-evm:~# lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0      179:0    0 14.7G  0 disk
mmcblk0boot0 179:32   0    4M  1 disk
mmcblk0boot1 179:64   0    4M  1 disk
mmcblk1      179:96   0  7.4G  0 disk
|-mmcblk1p1  179:97   0  128M  0 part /run/media/boot-mmcblk1p1
`-mmcblk1p2  179:98   0  7.3G  0 part /
mmcblk0是EMMC
mmcblk1是SD卡
有根目錄/是開機的地點

MMC開機
---
=> printenv mmcdev
mmcdev=0
=> printenv bootpart
bootpart=0

SD卡開機
---
=> printenv mmcdev
mmcdev=1
=> printenv bootpart
bootpart=1:2
1代表SD卡
2代表磁區


DFU燒錄
---
要先準備
rootfs.ext4

dd if=/dev/zero of=tisdk-base.ext4 bs=1M count=1200
mkfs.ext4 -F tisdk-base.ext4
mkdir mnt_fs
sudo mount -t ext4 tisdk-base.ext4 mnt_fs
cd mnt_fs
sudo tar xvf ../tisdk-base-image-am62xx-evm.tar.xz

[USB端]查看是否有bootloader字串
.\dfu-util.exe -l

這三個檔案是Kazi給的,TI原生檔案
.\dfu-util.exe -R -a 0 -D usb_tiboot3.bin
.\dfu-util.exe -R -a 0 -D usb_tispl.bin
.\dfu-util.exe -R -a 1 -D usb_u-boot.img

[Uart端]
要馬上回到這個視窗,在倒數3秒前卡在uboot

setenv uuid_gpt_disk 8f8c2a63-82d2-4693-95e2-fb4d6479aad2
setenv uuid_gpt_rootfs e14aea3b-1f60-447e-b0df-37a749a387e9
setenv uuid_gpt_data 12345678-1234-1234-1234-123456789abc
setenv partitions "uuid_disk=\${uuid_gpt_disk};name=mmcblk0p1,start=1MiB,size=3GiB,uuid=\${uuid_gpt_rootfs};name=mmcblk0p2,start=3GiB,size=3GiB,uuid=${uuid_gpt_data};name=mmcblk0p3,start=6GiB,size=3GiB"
gpt write mmc 0 ${partitions}
mmc part

設定 the dfu environment variables
setenv dfu_alt_info ${dfu_alt_info_emmc}
輸入Input to load emmc commands from PC
dfu 0 mmc 0

[USB端]
.\dfu-util.exe -a tiboot3.bin.raw -D tiboot3.bin --device ,0451:*
.\dfu-util.exe -a tispl.bin.raw -D tispl.bin --device ,0451:*
.\dfu-util.exe -a u-boot.img.raw -D u-boot.img --device ,0451:*
.\dfu-util.exe -R -a rootfs -D tisdk-base.ext4 --device ,0451:*

[Uart端]
mmc partconf 0 1 1 1
mmc bootbus 0 2 0 0

```
  </details>

## MPU clone與build與ota
<!-- clone -->
<details>
  <summary>clone</summary>
 
```markdown
--------------flo--------------
下載
repo init -u ssh://wmpsrv3.empgmdi.com:29418/manifest.git -b NXP/IMX8 -m IMX8_LINUX_BSP_2.2.0_YOCTO_SBD.xml --repo-url=ssh://wmpsrv3.empgmdi.com:29418/git-repo.git --config-name
repo sync
上傳
gitdir=$(git rev-parse --git-dir); scp -p -P 29418 fanyu.fy.che@wmpsrv3.empgmdi.com:hooks/commit-msg ${gitdir}/hooks/
git commit --amend
git push origin HEAD:refs/for/IMX8_LINUX_BSP_2.2.0_YOCTO_SBD
--------------MAP--------------
下載
repo init -u ssh://wmpsrv3.empgmdi.com:29418/manifest.git -b NXP/IMX8 -m IMX8_LINUX_BSP_2.2.0_YOCTO.xml --repo-url=ssh://wmpsrv3.empgmdi.com:29418/git-repo.git --config-name
repo sync
上傳
gitdir=$(git rev-parse --git-dir); scp -p -P 29418 fanyu.fy.chen@wmpsrv3.empgmdi.com:hooks/commit-msg ${gitdir}/hooks/
git commit --amend
git checkout remotes/origin/IMX8_LINUX_BSP_2.2.0_YOCTO
git push origin HEAD:refs/for/IMX8_LINUX_BSP_2.2.0_YOCTO
--------------MAP2--------------
下載
$ repo init -u ssh://wmpsrv3.empgmdi.com:29418/manifest.git -b NXP/IMX8 -m IMX_LINUX_BSP_6.6.36_YOCTO_MAP.xml --repo-url=ssh://wmpsrv3.empgmdi.com:29418/git-repo.git --config-name
$ repo sync
上傳
gitdir=$(git rev-parse --git-dir); scp -p -P 29418 fanyu.fy.che@wmpsrv3.empgmdi.com:hooks/commit-msg ${gitdir}/hooks/
git commit --amend
git push origin HEAD:refs/for/IMX_LINUX_BSP_6.6.36_YOCTO_MAP
--------------GLA--------------
下載
git clone ssh://mark.yl.lin@wmpsrv3.empgmdi.com:29418/TI/AM623/GLA/MPU_P0
cd ~/GLA_gerrit/MPU_P0
git pull
git switch MPU_P0 
上傳
git pull ssh://wmpsrv3.empgmdi.com:29418/TI/AM623/GLA/MPU_P0
--------------BNK--------------
下載
repo init -u ssh://wmpsrv3.empgmdi.com:29418/manifest.git -b NXP/IMX8 -m IMX_LINUX_BSP_6.6.36_YOCTO_BNK.xml \--repo-url=ssh://wmpsrv3.empgmdi.com:29418/git-repo.git --config-name
repo sync
上傳
gitdir=$(git rev-parse --git-dir); scp -p -P 29418 fanyu.fy.che@wmpsrv3.empgmdi.com:hooks/commit-msg ${gitdir}/hooks/
git commit --amend
git push origin HEAD:refs/for/IMX_LINUX_BSP_6.6.36_YOCTO_BNK

```
</details>
<!-- build -->
<details>
  <summary>build</summary>
 
```markdown
--------------flo--------------
source setup-environment bld-sbd
bitbake imx-image-multimedia
--------------MAP--------------
source setup-environment build-8mp
bitbake imx-image-multimedia
--------------MAP2--------------
source setup-environment bld-map
bitbake map2-image-multimedia
--------------GLA--------------
cd ~/GLA_gerrit/MPU_P0/build/
. conf/setenv
MACHINE=am62xx-evm bitbake -k tisdk-base-image
--------------BNK--------------
cd ~/Binoki/BNK_MPU/
source setup-environment bld-bnk
rm -rf bitbake-cookerdaemon.log cache/ tmp/
bitbake bnk-image-multimedia

```
</details>

<!-- ota -->
<details>
  <summary>ota</summary>
 
```markdown
--------------flo--------------

--------------MAP--------------
cd ~/EVSE_AC_new/ 
source setup-environment build-8mp 
rm -rf bitbake-cookerdaemon.log cache/ tmp/
bitbake imx-image-multimedia

cp tmp/deploy/images/imx8mp-lpddr4-evk/imx-image-multimedia-imx8mp-lpddr4-evk*.rootfs.tar.zst ./imx8mp-lpddr4-fox.rootfs.tar.zst
md5sum imx8mp-lpddr4-fox.rootfs.tar.zst > imx8mp-lpddr4-fox.rootfs.md5sum
zip -rP 1234 imx8mp-lpddr4-fox.zip imx8mp-lpddr4-fox.rootfs.tar.zst imx8mp-lpddr4-fox.rootfs.md5sum

透過Filezilla將Pack從PC複製到MPU內
from Path: /home/mark/EVSE_AC_new/build-8mp
to Path: /home/root/evcsdata/fwimage
File: imx8mp-lpddr4-fox.zip

MPU下指令
/home/root/evcs/evcs_shdmem fw_image_name imx8mp-lpddr4-fox.zip
/home/root/evcs/evcs_shdmem fw_update 1
直到回傳值為0
/home/root/evcs/evcs_shdmem fw_update

--------------MAP2--------------

--------------GLA--------------
cd ~/GLA_gerrit/MPU_P0/build/
. conf/setenv
MACHINE=am62xx-evm bitbake -k tisdk-base-image

```
</details>




## MPU-NXP-BSP筆記
<!-- GPIO -->
<details>
  <summary>GPIO</summary>

```markdown
  > <yocto_build_dir>/tmp/work/imx8mpevk-poky-linux/u-boot-imx/<specified_git_folder>/git/arch/arm/dts/imx8mp-pinfunc.h

  > <yocto_build_dir>/tmp/work/imx8mpevk-poky-linux/u-boot-imx/<specified_git_folder>/git/arch/arm/dts/imx8mp-pinfunc.h
  
  >  <yocto_build_dir>/tmp/work/imx8mpevk-poky-linux/u-boot-imx/<specified_git_folder>/git/arch/arm/dts/imx8mp-evk.dts  

  > <yocto_build_dir>/tmp/work-shared/imx8mpevk/kernel_source/arch/arm64/boot/dts/freescale/imx8mp-pinfunc.h    

  > <yocto_build_dir>/tmp/work-shared/imx8mpevk/kernel_source/arch/arm64/boot/dts/freescale/imx8mp-evk.dts  

  > /sys/devices/platform/30200000.gpio   
  > /sys/devices/platform/30210000.gpio   
  > cat /sys/kernel/debug/gpio

  - 可以查看  
  ls /proc/device-tree/   
  cat /proc/device-tree/__symbols__/main_i2c0   
  cat /proc/device-tree/gpio-leds/status/label    
  gpiodetect  
  gpioinfo 0  
  gpioget 0 9  
  gpiomon 0 9  
  要使用IO Expansion,PCA6416AHF  , 要看pca953x_gpio.c

  root@imx8mpevk:~# gpioget -c 1 1
"1"=inactive

root@imx8mpevk:~# gpioget GFCI_ERROR_R
"GFCI_ERROR_R"=inactive

root@imx8mpevk:~# cat /sys/class/leds/NGFCI_SPR_IN_L/brightness
0
root@imx8mpevk:~# echo 1 > /sys/class/leds/NGFCI_SPR_IN_L/brightness
root@imx8mpevk:~# echo 0 > /sys/class/leds/NGFCI_SPR_IN_L/brightness
root@imx8mpevk:~# cat /sys/class/leds/NGFCI_SPR_IN_L/brightness
0

gpioset -c 4 14=1
gpioinfo -c 1

```
</details>

<!-- I2C -->
<details>
  <summary>I2C</summary>

- Build  
gcc -o bbb hello_i2c.c -li2c    
ls -l /dev/i2c*   
ls -l /sys/bus/i2c/devices/

- 路徑    
/sys/class/i2c-dev    
/usr/include/i2c_drv    
/usr/lib/libi2c_drv.so    
#include <i2c/smbus.h>    
#include <linux/i2c.h>    
#include <linux/i2c-dev.h>    

- 列出總共有幾個  
i2cdetect -l    
i2cdetect -F 1      
設備的 I2C Bus1上，有n個Device    
i2cdetect -y 1    

- 使用i2cdump查詢設備內所有暫存器     
i2cdump -y 1 0x50   
i2cdump -y 1 0x50 w (讀取16bit)   
修改位於 i2c-1 上 0x50 的 0x12 暫存器，並將其數值修改為 5   
i2cset -f -y 1 0x50 0x12 5

- 寫多個    
i2ctransfer -f -y 1 w3@0x68 0x00 0x80 0x07    
- 讀多個    
i2ctransfer -f -y 1 w2@0x68 0x00 0x84 r5    

- 察看剛剛所設定的 0x12 暫存器    
i2cget  -y 1 0x50 0x12(讀取16bit)   
i2ctransfer 0 w7@0x50 0x42 0xff-

</details>

<!-- MMC -->
<details>
  <summary>MMC</summary>

ls /dev | grep mmc

mmc --help
mmc extcsd read /dev/mmcblk2  
mmc extcsd read /dev/mmcblk2 | grep BKOPS_EN  
mmc bkops enable /dev/mmcblk2 

fdisk -l /dev/mmcblk2 
fdisk /dev/mmcblk2  
blkdiscard --secure /dev/mmcblk2  

bootloader底下
- bootloader$ mmc rescan  
- bootloader$ mmc list  
- bootloader$ mmc dev 2 
- bootloader$ mmc info  

</details>


<!-- PWM -->
<details>
  <summary>PWM</summary>
相關路徑 meta-emcraft/recipes-kernel/linux/linux-imx/imx8m-som.dts: 

查看有多少PWM ls /sys/class/pwm/  

查看 cat /sys/kernel/debug/pwm  

cd /sys/class/pwm/pwmchip0

合併
- echo 0 > export && echo 1000000 > pwm0/period && echo 500000 > pwm0/duty_cycle && echo 1 > pwm0/enable  


分開打
- echo 0 > /sys/class/pwm/pwmchip2/export
- echo 1000000 > /sys/class/pwm/pwmchip2/pwm0/period
- echo 500000 > /sys/class/pwm/pwmchip2/pwm0/duty_cycle
- echo 1 > /sys/class/pwm/pwmchip2/pwm0/enable

</details>


<!-- RFID -->
<details>
  <summary>RFID</summary>

參考PN7150安裝網址
https://www.wpgdadatong.com/tw/blog/detail/44420

https://community.nxp.com/t5/i-MX-Processors-Knowledge-Base/PN7150-NFC-Controller-on-i-MX8M-mini-evk-running-Yocto/ta-p/1125177

要改的東西如下

[Kconfig]檔案

-----help------ ==> help

pr_warining(xxxxxx)  ==> 將他註解或換pr_err()

[workshare 的 dts]

在i2c3底下修改
    
     pinctrl-0 = <&pinctrl_i2c3>;
     ststus = "okay"; 
    
     pn547: pn547@28 { 
         compatible = "nxp,pn547";
         reg = <0x28>; 
         interrupt-gpios = <&gpio3 19 0>;
         enable-gpios = <&gpio5 13 0>;
    };

※如果要用開發版 , VEN接J21的pin24 , irq接 J21的pin 32

這邊要註解

/*
&pwm4 {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_pwm4>;
	status = "okay";
};
*/

新增gpio3_io19的  GPIO腳

	pinctrl_ecspi2_cs: ecspi2cs {
		fsl,pins = <
			MX8MP_IOMUXC_ECSPI2_SS0__GPIO5_IO13		0x40000
			MX8MP_IOMUXC_SAI5_RXFS__GPIO3_IO19		0x40000
		>;
	};

沒用的測試
- i2ctransfer 4 w4@0x50 0x20 0x00 0x01 0x01 r6

</details>

<!-- RTC -->
<details>
  <summary>RTC</summary>

- cat /sys/class/rtc/rtc*/name
- rtc-rv3028 0-0052
- snvs_rtc 30370000.snvs:snvs-rtc-lp
</details>


<!-- SPI -->
<details>
  <summary>SPI</summary>
  
```markdown
kernel/linux-4.14/drivers/spi/spidev.c

/sys/class/spi_master/spi1/power/control
cat /sys/bus/spi/devices/spi1.0/uevent

看看 /dev/ 裡面有的 spidev 開頭的裝置
ls /dev/ | grep spidev
dmesg | grep spi

測試
spidev_test -D /dev/spidev2.0 -v -p string_to_send
如果要新增一組chip select 剛好是pwm4的腳
要注意避免
MX8MP_IOMUXC_SAI5_RXFS__PWM4_OUT	0x116
跟
MX8MP_IOMUXC_SAI5_RXFS__GPIO3_IO19		0x40000
衝突


這邊要註解
/*
&pwm4 {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_pwm4>;
	status = "okay";
};
*/


下面要新增
&ecspi2 {
	#address-cells = <1>;
	#size-cells = <0>;
	fsl,spi-num-chipselects = <2>;
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_ecspi2 &pinctrl_ecspi2_cs>;
	cs-gpios = <&gpio5 13 GPIO_ACTIVE_LOW>,<&gpio3 19 0>;
	status = "okay";

	spidev1: spi@0 {
		reg = <0>;
		compatible = "rohm,dh2228fv";
		spi-max-frequency = <500000>;
	};
	spidev2: spi@1 {
		reg = <1>;
		compatible = "rohm,dh2228fv";
		spi-max-frequency = <400000>;
	};
};


	pinctrl_ecspi2_cs: ecspi2cs {
		fsl,pins = <
			MX8MP_IOMUXC_ECSPI2_SS0__GPIO5_IO13		0x40000
			MX8MP_IOMUXC_SAI5_RXFS__GPIO3_IO19		0x40000
		>;
	};


```
</details>

<!-- SPI NOR flash -->
<details>
  <summary>SPI NOR flash</summary>
mtdinfo --all
</details>

<!-- Thermal -->
<details>
  <summary>Thermal</summary>
cat /sys/class/thermal/thermal_zone0/temp
</details>

<!-- Timer -->
<details>
  <summary>Timer</summary>
 
```markdown
cat /sys/kernel/debug/gc/clk
cat /sys/kernel/debug/gc/info
cat /sys/kernel/debug/gc/meminfo
cat /sys/kernel/debug/gc/idle

ls /proc/device-tree/timer
```
</details>

<!-- Uart -->
<details>
  <summary>Uart</summary>
 
```markdown
相關路徑 arch/arm64/boot/dts/freescale/fsl-imx8mq.dtsi
相關路徑 meta-emcraft/recipes-kernel/linux/linux-imx/imx8m-som.dts

設定baud
stty -echo raw speed 115200 < /dev/ttymxc2

監聽
cat /dev/ttymxc2

傳送
echo abcdef > /dev/ttymxc
```
</details>


<!-- Linux 中查看 eMMC 分區 -->
<details>
  <summary>Linux 中查看 eMMC 分區</summary>
  
```markdown
lsblk

```
</details>


<!-- GLA的MPU用USB從頭燒錄 -->
<details>
  <summary>GLA的MPU用USB從頭燒錄</summary>
  
```markdown
可以先查看磁區
linux底下
lsblk
uboot底下
mmc part
---------------------------
刪除linux image的方法
mmc dev 0 1
mmc erase 0 0x2000

刪除uboot的方法
mmc dev 0 0
mmc erase 0 0x2000
---------------------------
Prepare the files
rootfs.ext4

dd if=/dev/zero of=tisdk-base.ext4 bs=1M count=1200
mkfs.ext4 -F tisdk-base.ext4
mkdir mnt_fs
sudo mount -t ext4 tisdk-base.ext4 mnt_fs
cd mnt_fs
sudo tar xvf ../tisdk-base-image-am62xx-evm.tar.xz

---------------------------

參考網址
http://192.168.3.173:3000/fDHYPQWIRzWdWJes5lmmMA?view

指撥到USB開機模式,從開機

[PC]查看是否連線,到dfu-util-0.9-win64資料夾
.\dfu-util.exe -l

[PC]燒到RAM
.\dfu-util.exe -R -a 0 -D usb_tiboot3.bin
.\dfu-util.exe -R -a 0 -D usb_tispl.bin
.\dfu-util.exe -R -a 1 -D usb_u-boot.img   , *這邊uart端需要按任何鍵,進入uboot*

[UART]Partitioning eMMC for new board 
setenv uuid_gpt_disk 8f8c2a63-82d2-4693-95e2-fb4d6479aad2
setenv uuid_gpt_rootfs e14aea3b-1f60-447e-b0df-37a749a387e9
setenv uuid_gpt_data 12345678-1234-1234-1234-123456789abc
setenv partitions "uuid_disk=\${uuid_gpt_disk};name=mmcblk0p1,start=1MiB,size=3GiB,uuid=\${uuid_gpt_rootfs};name=mmcblk0p2,start=3GiB,size=3GiB,uuid=${uuid_gpt_data};name=mmcblk0p3,start=6GiB,size=3GiB"
gpt write mmc 0 ${partitions}
mmc par

[UART]設定環境變數
Setting the dfu environment variables

[UART]在 U-Boot 中使用 DFU（Device Firmware Upgrade）模式來更新 eMMC 的命令
dfu 0 mmc 0

dfu: 表示進入 DFU 模式。
0: 表示使用的 DFU 端點號，通常是 0。
mmc: 表示目標設備是 eMMC。
0: 表示 eMMC 的設備號，通常是 0。

[PC]再次查看
.\dfu-util.exe -l

[PC]to MPU eMMC.
.\dfu-util.exe -a tiboot3.bin.raw -D tiboot3.bin --device ,0451:*
.\dfu-util.exe -a tispl.bin.raw -D tispl.bin --device ,0451:*
.\dfu-util.exe -a u-boot.img.raw -D u-boot.img --device ,0451:*
.\dfu-util.exe -R -a rootfs -D tisdk-base.ext4 --device ,0451:*

[UART]Firmware update completed
mmc partconf 0 1 1 1
mmc bootbus 0 2 0 0

然後把指撥還原,需要從開機

lsblk

```
</details>


<!-- Uboot -->
<details>
  <summary>Uboot</summary>
 
```markdown
bdinfo
printenv bootpart
printenv mmcdev
version 

mmc part

setenv bootdelay 5
setenv bootpart 1:2
setenv mmcdev 1
saveenv

Uboot的倒數可以參考tmp/work/imx8mpddr4evk-poky-linux/u-boot-imx/2022.04-r0/git/common/autoboot.c
裡的bootdelay,改CONFIG_BOOTDELAY試試看

```
</details>

<!-- watchdog -->
<details>
  <summary>watchdog</summary>
 
```markdown
 the file system.conf in /etc/systemd/ 
```
</details>

<!-- wifi_wpa_cliwifi -->
<details>
  <summary>wifi_wpa_cliwifi</summary>
 
```markdown

modprobe moal mod_para=nxp/wifi_mod_para.conf
檔案路徑在/lib/firmware/nxp

启动wpa_supplicant应用
$ wpa_supplicant -D nl80211 -i mlan0 -c /etc/wpa_supplicant.conf -B


启动wpa_cli应用
$ wpa_cli -i mlan0 scan             // 搜索附近wifi网络
$ wpa_cli -i mlan0 scan_result      // 打印搜索wifi网络结果
$ wpa_cli -i mlan0 add_network      // 添加一个网络连接
如果要连接加密方式是[WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][ESS] (wpa加密)，wifi名称是name，wifi密码是：psk。

$ wpa_cli -i mlan0 set_network 0 ssid '"TP-Link_CCBD"'
$ wpa_cli -i mlan0 set_network 0 psk '"63504149"'
$ wpa_cli -i mlan0 enable_network 0

分配ip/netmask/gateway/dns
$ udhcpc -i mlan0 -s /etc/udhcpc.script -q

 ifconfig mlan0 up
 ifconfig mlan0 192.168.3.160

连接已有的连接
$ wpa_cli -i mlan0 list_network             列举所有保存的连接
$ wpa_cli -i mlan0 select_network 0         连接第1个保存的连接
$ wpa_cli -i mlan0 enable_network 0         使能第1个保存的连接

断开wifi
$ ifconfig mlan0 down
$ killall udhcpc
$ killall wpa_supplicant

對外網路
route add default gw 192.168.3.150

在/etc/resolv.conf新增
nameserver 8.8.8.8 或者
nameserver 192.168.3.150
```
</details>



<!-- wifi -->
<details>
  <summary>wifi</summary>
 
```markdown
-------------------最新
modprobe moal mod_para=nxp/wifi_mod_para.conf
wpa_supplicant -B -i mlan0 -c /etc/wpa_supplicant.conf
udhcpc -i mlan0
-------------------

路徑/lib/firmware/nxp/wifi_mod_para.conf

第一步開啟wifi-driver
modprobe moal mod_para=nxp/wifi_mod_para.conf

ifconfig mlan0 up

掃描可用的無線 AP 站
iwlist mlan0 scan | grep -i essid



wpa_supplicant -D nl80211 -i mlan0 -c /etc/wpa_supplicant.conf -B

iwconfig mlan0 essid TP-Link_CCBD key 63504149
iwconfig mlan0 essid "TP-Link_2.4g_CCBD" key "63504149"
iwconfig mlan0 essid fanyu key 123456789


分配ip/netmask/gateway/dns
$ udhcpc -i mlan0 -s /etc/udhcpc.script -q


檢查 ESSID 設定：
iwconfig mlan0

查log
sudo cat /var/log/syslog | grep etwork | tail -n25

interface=wifi
ssid=imx8mp-evk

不知名的指令
wpa_passphrase TP-Link_CCBD 63504149
----------/etc/hostapd.conf-------------------
interface=mlan0
ssid=<name_of_AP>
country_code=US
hw_mode=a
channel=0
ieee80211n=1
--------------/etc/wpa_supplicant.conf--------------
network={
scan_ssid=1
ssid="Wifi_ssid_NAME"
psk="password"
}
```
</details>

<!-- uuu -->
<details>
  <summary>uuu</summary>
 
```markdown
進入kernel後可以用指令刪掉
echo 0 > /sys/block/mmcblk2boot0/force_ro
dd if=/dev/zero of=/dev/mmcblk2boot0
進入u-boot刪除
mmc dev 2 1
mmc erase 0 0x2000
--------------SBD--------------
要先只播開關切換到usb download mode,要另外接USB線,再開機
* List connected know devices
	uuu -lsusb
 
* Burn image by uuu
	uuu -b emmc_all imx-boot-imx8mpevk-sd.bin-flash_evk imx-image-multimedia-imx8mpevk.rootfs.wic.zst 
```
</details>



<!-- nfs command -->
<details>
  <summary>nfs command</summary>
 
```markdown
--------------SBD--------------
參考網址
https://developer.ridgerun.com/wiki/index.php/IMX8/iMX8MEVK/Yocto/Alternative_image_loading

直接用原來的image跟dtb開機
run loadfdt
fdt addr ${fdt_addr_r}
run mmcargs
run loadimage
booti ${loadaddr} - ${fdt_addr_r}

方法1: rootfs(不使用)
-----------------------------------------------------------
setenv bootargs console=ttymxc1,115200 earlycon root=/dev/nfs \
nfsroot=192.168.5.100:/srv/rootfs,nfsvers=3 rw debug \
ip=192.168.5.1::192.168.5.254:255.255.255.0:root:eth0:on
setenv bootcmd "run loadfdt;run loadimage; booti ${loadaddr} - ${fdt_addr}"
_________________

方法2: rootfs+dtb
-----------------------------------------------------------
setenv ipaddr 192.168.5.1
setenv serverip 192.168.5.100 
setenv ip_dyn no
setenv image Image; setenv fdt_file imx8mp-sbd-PreEVT.dtb
setenv bootargs console=ttymxc1,115200 earlycon root=/dev/nfs \
nfsroot=192.168.5.100:/srv/rootfs,nfsvers=3 rw debug \
ip=192.168.5.1::192.168.5.254:255.255.255.0:root:eth0:on \
vt.global_cursor_default=0
setenv bootcmd "tftpboot ${loadaddr} ${image}; tftpboot ${fdt_addr} ${fdt_file}; booti ${loadaddr} - ${fdt_addr}"
run bootcmd
_________________

方法3: rootfs+dtb ,需要建立boot資料夾,把dtb放進boot裡(不使用)
-----------------------------------------------------------
setenv ipaddr 192.168.5.1 
setenv serverip 192.168.5.100 
setenv ip_dyn no
setenv image Image; setenv fdt_file imx8mp-sbd-PreEVT.dtb
setenv nfsroot /srv/rootfs
setenv netargs 'setenv bootargs console=ttymxc1,115200 ${smp} root=/dev/nfs ip=192.168.5.1::192.168.5.254:255.255.255.0:root:eth0:on nfsroot=${serverip}:${nfsroot},v3,tcp'
run netboot
_________________
可以選擇要不要存檔
saveenv

專門用在flo的舊版本uboot,需要uimage
-----------------------------------------------------------
方法4: rootfs+dtb 專門用在flo的舊版本uboot,需要uimage
setenv ipaddr 192.168.5.1 
setenv serverip 192.168.5.100 
setenv ip_dyn no
setenv image uImage; setenv fdt_file imx8mp-sbd-PreEVT.dtb
setenv bootargs console=ttymxc1,115200 earlycon root=/dev/nfs nfsroot=${serverip}:/srv/rootfs,nfsvers=3 rw debug ip=${ipaddr}::::root:eth0:on
setenv bootcmd "tftpboot 0x40400000 ${image}; tftpboot 0x43000000 ${fdt_file}; bootm 0x40400000 - 0x43000000"
run bootcmd


需要先將Image換成uImage
sudo mkimage -A arm64 -O linux -T kernel -C none -a 0x40400000 -e 0x40400000 -n "Linux Kernel" -d Image uImage

-----------------------------------------------------------

## pc端
sudo tar -xvf imx-image-multimedia-imx8mpevk.rootfs.tar.zst -C /home/fanyu/fanyu/imx8-evk-dummy/rootfs/
cp Image /srv/tftp
cp imx8mp-sbd-PreEVT.dtb /srv/tftp 或 cp imx8mp-sbd-PreEVT.dtb /srv/tftp/boot

## 下面是DEBUG專用
mii info
mii device

setenv ethact FEC0
setenv ethaddr 4E:8D:BC:5D:7F:EB  # 設定 MAC 地址
setenv ipaddr 192.168.3.1
setenv serverip 192.168.3.100
## 查看錯誤
dmesg -l warn
dmesg -l err

## 強制使用網路接口
setenv ethact ethernet@30be0000



## SBD Linux device tree
EVSE_SBD/imx-6.6.3-1/modify/sources/linux-imx_lf-6.6.y/arch/arm64/boot/dts/freescale/imx8mp-sbd-PreEVT.dts
 
## SBD u-boot device tree
EVSE_SBD/imx-6.6.3-1/modify/sources/uboot-imx_lf-6.6.3-1.0.0/arch/arm/dts/imx8mp-evk.dts

## Build uboot
 
  * bitbake u-boot-imx -c cleanall
  * bitbake u-boot-imx -c menuconfig -vDD

u-boot=> mdio list
FEC0:
1 - RealTek RTL8201F 10/100Mbps Ethernet <--> ethernet@30be0000
ethernet@30bf0000:

u-boot=> mii device


還原不要開機自動
env default -a
saveenv

可以讀看看
md 0x43000000 10
```
</details>


<!-- nfs and tftp install -->
<details>
  <summary>nfs and tftp install</summary>
 
```markdown
--------------SBD--------------
## nfs
參考網址
https://developer.ridgerun.com/wiki/index.php/IMX8/iMX8MEVK/Yocto/Alternative_image_loading
-----------------------------------------------------------------------------------
安裝 NFS 伺服器
sudo apt install nfs-kernel-server

編輯 /etc/exports，加入共享設定
sudo nano /etc/exports
添加以下內容：
/home/fanyu/fanyu/imx8-evk-dummy/rootfs *(rw,sync,insecure,no_root_squash,no_subtree_check)

把檔案解壓縮並複製
sudo tar -xvf imx-image-multimedia-imx8mpevk.rootfs.tar.zst -C /home/fanyu/fanyu/imx8-evk-dummy/rootfs/

重新啟動 NFS 服務
sudo exportfs -a
sudo systemctl restart nfs-kernel-server

確認 NFS 共享是否生效
sudo exportfs -v

在 i.MX8MP 開發板上掛載 NFS
sudo mount -t nfs 192.168.5.100:/home/fanyu/fanyu/imx8-evk-dummy/rootfs /home/root/nfs
sudo mount -o nfsvers=3 192.168.5.100:/home/fanyu/fanyu/imx8-evk-dummy/rootfs /mnt

------------------------------------------------------------------------------------
## tftp
sudo apt-get install xinetd tftpd tftp
sudo apt-get install tftp-hpa tftpd-hpa

創建共享資料夾
sudo mkdir -p /srv/tftp

複製檔案
cp Image /home/fanyu/fanyu/imx8-evk-dummy/tftp
cp imx8mp-foxconn.dtb /home/fanyu/fanyu/imx8-evk-dummy/tftp/



• 修改 /etc/default/tftpd-hpa 文件：
# /etc/default/tftpd-hpa
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/home/fanyu/fanyu/imx8-evk-dummy/tftp"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="-l -c -s --secure"

sudo /etc/init.d/tftpd-hpa start
下面不要用
新增/etc/xinetd.d/tftp
service tftp
{
    protocol        = udp
    port            = 69
    socket_type     = dgram
    wait            = yes
    user            = nobody
    server          = /usr/sbin/in.tftpd
    server_args     = /srv/tftp
    disable         = no
}

sudo /etc/init.d/xinetd restart


```
</details>

<!-- menuconfig -->
<details>
  <summary>menuconfig</summary>
 
```markdown
--------------kernel--------------
bitbake virtual/kernel -c menuconfig


--------------u-boot--------------
要打開u-boot menuconfig


bitbake u-boot-imx -c devshell

可以到這個路徑
/EVSE_AC_NEW/modify/sources/u-boot-imx
make menuconfig

如果有問題

bitbake u-boot-imx -c cleanall
bitbake u-boot-imx -c menuconfig -vDD
bitbake u-boot-imx

到 /EVSE_AC_NEW/modify/sources/u-boot-imx
make mrproper


```
</details>














<!-- Fan -->
<details>
  <summary>Fan</summary>
 
```markdown
--------------flo--------------
root@imx8mpevk:/# cat /sys/class/hwmon/hwmon*/name
ads7830
ads7830
cpu_thermal
soc_thermal
lm96163

root@imx8mpevk:/# echo 128 > /sys/class/hwmon/hwmon4/pwm1
[ 4090.785327] client->addr=76,command=76,data.byte =23
root@imx8mpevk:/# echo 0 > /sys/class/hwmon/hwmon4/pwm1
[ 4022.315438] client->addr=76,command=76,data.byte =0

```
</details>
















## md-and-html範例
<details>
  <!-- <summary>GPIO</summary> -->

  ### 刪除縣
  1. ~~這是markdown範例~~  
  2. 這是 <del>html刪除線</del> 的範例。
  ### 分隔線
  --- 
  ### 連至索引
  1. [markdown範例](#連至索引)
  2. <a href="#連至索引"  style="font-size: 16px;">html範例</a>
  <!-- <ul><li><a href="#連至索引">html範例</a></li></ul> -->
</details>

---

