# Wifi - Marvell 88W8997 2*2 ac
###### tags: `wifi` `88w8997`

[TOC]

```
From iwconfig:

mode
    Set the operating mode of the device, which depends on the network
    topology. The mode can be Ad-Hoc (network composed of only one cell
    and without Access Point), Managed (node connects to a network
    composed of many Access Points, with roaming), Master (the node is
    the synchronisation master or acts as an Access Point), Repeater
    (the node forwards packets between other wireless nodes), Secondary
    (the node acts as a backup master/repeater), Monitor (the node is not
    associated with any cell and passively monitor all packets on the
    frequency) or Auto.
    Example :
        iwconfig eth0 mode Managed
        iwconfig eth0 mode Ad-Hoc
```

## Buildroot patch

- Firmware file
    - Kconfig http://buildroot-busybox.2317881.n4.nabble.com/PATCH-v1-package-linux-firmware-add-option-for-Marvell-8997-wifi-cards-td279120.html#a279293
    - file source 
        - http://sources.buildroot.net/linux-firmware/git/mrvl/

## Software Test

### 除錯可能會用到 command

- wireless_tools
    - iwconfig
    - iwpriv
    - iwxxx
    - iwspy
    - ifrename


### Hardware Test

- Normal Mode (測試 Client/AP mode 連線)
    - Normal mode firmware
    - mftutil
- MFG Mode (測試 RF)
    - MFG firmware
    - PC - Ethernet to DUT
        - Labtool (Labtool Client)
    - DUT - Ethernet to PC
        - mfbridge (Labtool Server)


### Mtest

- 一臺 AP mode, 一臺 client mode


### 測試流程

- 確認 firmware in ```/lib/firmware/mrvl```
- 確認 kernel module
    - ```cfg80211``` , ```rfkill```, ```mwifiex```, ```mwifiex_pcie```

- 確認 user space tool
    - dhclient
    - wpa_supplicant
    - rfkill (非必要)

- 準備 wpa_supplicant

```
ctrl_interface=/var/run/wpa_supplicant
ap_scan=1

network={
        ssid="ShaunPixel4XL"
        psk="04150415"
}
```

- 執行 wpa_supplicant
    - ```wpa_supplicant -D nl80211 -i wlp1s0 -c /etc/wpa_supplicant.conf```
- 要 IP (default wpa_supplicant 應該就要會要)
    - ```dhclient wlp1s0```

### 開機的 message

![](https://i.imgur.com/6PNWUqq.png)

- 卡如果沒插好就不會 rename 

### 連上的 message

![](https://i.imgur.com/EI3zlIq.png)

### device id

![](https://i.imgur.com/lIB0EgZ.png)

## TODO 

- Add Kconfig to default
- Add mfbridge to buildroot



## LS10XX BSP

- 需要先選用某些 kernel config 後才會看到 wifiex dirver 選項

```
1.
Suggest LSDK 1906 kernel 4.14
Follow: https://docs.nxp.com/bundle/GUID-87AD3497-0BD4-4492-8040-3F3BE0F2B087/page/GUID-2C30B1D2-2E1A-49BC-A3F7-D481D76B00BE.html
to create a boot SD card.
and 
create LSDK1906 ¿Ùπ“

2. Modify and replace Kernel

How to replace the default Linux kernel with custom kernel on target board:

Step1: Optionally, run 'flex-builder -i repo-fetch -B linux' to download linux source, then modify Linux kernel source code in <flexbuild_dir>/packages/linux/<kernel-repo>
(LSDK ¿Ùπ“ build ¶nßY¶≥)

Step2: Optionaly, customize kernel options in interactive menu by command "flex-builder -c linux:custom -a arm64"
https://www.linuxquestions.org/questions/linux-kernel-70/linux-2-6-37-rc5-brcm80211-staging-driver-wlan0-no-wireless-extension-850277/

[*] Networking support  --->
  -*-   Wireless  --->
    <M>   cfg80211 - wireless configuration API
    ...
    [*] cfg80211 wireless extensions compatibility

Device Drivers  ---> Network device support  ---> Wireless LAN
<*>IEEE 802.11 for Host AP (Prism2/2.5/3 and WEP/TKIP/CCMP)
§U™∫®‚≠”•ÿø˝≥£≠nøÔ <*>
•i•H≈˝:
CONFIG_WIRELESS_EXT=y CONFIG_WEXT_PRIV=y

Kernel over 4.8 has no old API for ioctl, iwpriv command will fail, so need merge old code into 4.14
Ref: http://blog.lyjiot.cn:8088/2020/07/09/iwpriv.html

ª›¶bnet/wireless/wext-core.c wireless_process_ioctl()•[§J¬¬api
// add begin
        /* Old driver API : call driver ioctl handler */
        if (dev->netdev_ops->ndo_do_ioctl)
                return dev->netdev_ops->ndo_do_ioctl(dev, (struct ifreq *)iwr, cmd);
// add end 

Step3: Build new kernel by command "flex-builder -c linux -a arm64"

Step4: Generate new linux tarball by command "flex-builder -i mkbootpartition -a arm64"

The new kernel tarball <flexbuild-dir>/build/images/linux_4.x_LS_arm64_<timestamp>.tgz will be generated.

Step5: Login LSDK Linux system on target board and replace the existing kernel as below:
root@localhost:/# tar xfmv linux_4.14_LS_arm64_<timestamp>.tgz -C /
root@localhost:/# reboot

3. Wireless Modules and NXP WiFi modules
Default LS platform does not bring up rfkill and cfg80211
so need manually build rfkill.ko and cfg80211.ko

Go into LSDK package/linux/linux/
For 64-bit Arm:
$ sudo apt-get install gcc-aarch64-linux-gnu  
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ export ARCH=arm64

$ make defconfig lsdk.config
$ make menuconfig
≥]©w¶p "2. Modify Kernel" Step2 ≥£≠n∞µ
$ make
$ make modules

build¶b
net/rfkill/rfkill.ko
net/wireless/cfg80211.ko

-NXP WiFi Modules.
Download WiFi 8997 driver package

in wlan_src/
#sudo bash
#KERNELDIR="LSDK path package/linux/linux" LDFLAGS="" make
build mlan.ko and pcie8xxx.ko

4. Build mfgbridge

Copy MFG-W8997-MF-WIFI-BT-BRG-FC-VS2013-1.0.0.173-16.80.205.p193
æ„•]®Ï LS10xx ARDB ∏Ã≠±
¶b device§Wbuild

ª›•˝≠◊ßÔ Makefile
CC=aarch64-linux-gnu-gcc

BT=n
hot_plug=n
#make
•i•H±o®Ï mfgbridge

5. Bring up wifi normal mode:
copy FW pcie8997_wlan_v4.bin to /lib/firmware/nxp/
#insmod rfkill.ko
#insmod cfg80211.ko
#insmod mlan.ko
#insmod pcie8xxx.ko drv_mode=1 fw_name=nxp/pcie8997_wlan_v4.bin cfg80211_wext=0xf  cal_data_cfg=none
rename interface:
#ip link set wlP1p1s0 name mlan0

install wpa_supplicant
wpa_supplicant -i mlan0 -Dnl80211 -c 1.conf -B 

1.conf:
network={
ssid="xxx"
psk="xxx"
}
#dhclient mlan0

6. Bring up wifi mfg mode: (for labtool usage)
copy mfg FW pcie8997_uart_combo.bin to /lib/firmware/nxp/
#insmod rfkill.ko
#insmod cfg80211.ko
#insmod mlan.ko
#insmod pcie8xxx.ko drv_mode=1 mfg_mode=1 fw_name=nxp/pcie8997_uart_combo.bin cfg80211_wext=0xf  cal_data_cfg=none

rename interface:
#ip link set wlP1p1s0 name mlan0

connect Wire line to switch or router
$ ifconfig fm1-mac4 up
$ ifconfig fm1-mac4 192.168.xx.xx

#./mfgbridge

-> follow labtool SOP

```
## 特殊 feature

- Precision indoor location capability, including IEEE 802.11mc as well as Bluetooth Low Energy direction finding

## Driver

- mwlwifi(https://openwrt.org/docs/techref/driver.wlan/mwlwifi)
    - Firmware : http://sources.buildroot.net/linux-firmware/git/mwlwifi/

- [mwifiex](https://wireless.wiki.kernel.org/en/users/drivers/mwifiex)
    - Firmware
        - Buildroot package : linux-firmware
    - Driver
        - https://cateee.net/lkddb/web-lkddb/MWIFIEX_USB.html

## Installation

- install driver
- install fwr


## RF測試 (Marvell 提供的是 Labtool Setup)

### Bridge PC Setup

![](https://i.imgur.com/1pF2f5o.png)

- 目前先以 DUT = Linux PC + Marvell 做 RF 測試
- 之後 FAE 會再 support 到將 DUT 替換為 NXP + Marvell 的 RF 測試
    - FAE 已提供 LSDK 的 BSP
    - 目前尚缺 for OpenWRT 的

- Topology 方式
    - 1. LAB Tool 可以直接裝在 Windows 上對 VM 去打
    - 2. 接 topology with router/switch 

```bash=
# Vagrant box
vagrant init ubuntu/xenial64
vagrant up


tar -zxvf Ubuntu_14.tgz >cd Ubuntu_14
sudo bash install.sh
reboot

# Check Kernel version 3.18 and install packages
uname -r
apt-get install vim
apt-get install gawk
apt-get install openssh-server
apt-get install libnl-dev
apt-get install libreadline6-dev
apt-get install libssl-dev
apt-get install libbluetooth-dev
apt-get install libncurses5-dev
apt-get install libc6-dev-i386

# Check gcc version 4.84
gcc -v

# Disable firewall
service iptables stop

# Install driver package

```



## Open Source

- hostapd
- wpa-supplicant
- wpad


## Trace hostapd

![](https://i.imgur.com/T0YrO3i.png)


## Kernel Config

## FAE note

- kernel
    - Suggest LSDK 1906 kernel 4.14
```
Follow: https://docs.nxp.com/bundle/GUID-87AD3497-0BD4-4492-8040-3F3BE0F2B087/page/GUID-2C30B1D2-2E1A-49BC-A3F7-D481D76B00BE.html
```
to create a boot SD card.

and 

create LSDK1906 環境



2. Modify and replace Kernel



How to replace the default Linux kernel with custom kernel on target board:


```
Step1: Optionally, run 'flex-builder -i repo-fetch -B linux' to download linux source, then modify Linux kernel source code in <flexbuild_dir>/packages/linux/<kernel-repo>

(LSDK 環境 build 好即有)



Step2: Optionaly, customize kernel options in interactive menu by command "flex-builder -c linux:custom -a arm64"

https://www.linuxquestions.org/questions/linux-kernel-70/linux-2-6-37-rc5-brcm80211-staging-driver-wlan0-no-wireless-extension-850277/



[*] Networking support  --->

  -*-   Wireless  --->

    <M>   cfg80211 - wireless configuration API

    ...

    [*] cfg80211 wireless extensions compatibility



Device Drivers  ---> Network device support  ---> Wireless LAN

<*>IEEE 802.11 for Host AP (Prism2/2.5/3 and WEP/TKIP/CCMP)

下的兩個目錄都要選 <*>

可以讓:

CONFIG_WIRELESS_EXT=y CONFIG_WEXT_PRIV=y



Kernel over 4.8 has no old API for ioctl, iwpriv command will fail, so need merge old code into 4.14

Ref: http://blog.lyjiot.cn:8088/2020/07/09/iwpriv.html



需在net/wireless/wext-core.c wireless_process_ioctl()加入舊api

// add begin

        /* Old driver API : call driver ioctl handler */

        if (dev->netdev_ops->ndo_do_ioctl)

                return dev->netdev_ops->ndo_do_ioctl(dev, (struct ifreq *)iwr, cmd);

// add end 



Step3: Build new kernel by command "flex-builder -c linux -a arm64"



Step4: Generate new linux tarball by command "flex-builder -i mkbootpartition -a arm64"



The new kernel tarball <flexbuild-dir>/build/images/linux_4.x_LS_arm64_<timestamp>.tgz will be generated.



Step5: Login LSDK Linux system on target board and replace the existing kernel as below:

root@localhost:/# tar xfmv linux_4.14_LS_arm64_<timestamp>.tgz -C /

root@localhost:/# reboot



3. Wireless Modules and NXP WiFi modules

Default LS platform does not bring up rfkill and cfg80211

so need manually build rfkill.ko and cfg80211.ko



Go into LSDK package/linux/linux/

For 64-bit Arm:

$ sudo apt-get install gcc-aarch64-linux-gnu  

$ export CROSS_COMPILE=aarch64-linux-gnu-

$ export ARCH=arm64



$ make defconfig lsdk.config

$ make menuconfig

設定如 "2. Modify Kernel" Step2 都要做

$ make

$ make modules



build在

net/rfkill/rfkill.ko

net/wireless/cfg80211.ko



-NXP WiFi Modules.

Download WiFi 8997 driver package



in wlan_src/

#sudo bash

#KERNELDIR="LSDK path package/linux/linux" LDFLAGS="" make

build mlan.ko and pcie8xxx.ko



4. Build mfgbridge



Copy MFG-W8997-MF-WIFI-BT-BRG-FC-VS2013-1.0.0.173-16.80.205.p193

整包到 LS10xx ARDB 裡面

在 device上build



需先修改 Makefile

CC=aarch64-linux-gnu-gcc



BT=n

hot_plug=n

#make

可以得到 mfgbridge



5. Bring up wifi normal mode:

copy FW pcie8997_wlan_v4.bin to /lib/firmware/nxp/

#insmod rfkill.ko

#insmod cfg80211.ko

#insmod mlan.ko

#insmod pcie8xxx.ko drv_mode=1 fw_name=nxp/pcie8997_wlan_v4.bin cfg80211_wext=0xf  cal_data_cfg=none

rename interface:

#ip link set wlP1p1s0 name mlan0



install wpa_supplicant

wpa_supplicant -i mlan0 -Dnl80211 -c 1.conf -B 



1.conf:

network={

ssid="xxx"

psk="xxx"

}

#dhclient mlan0



6. Bring up wifi mfg mode: (for labtool usage)

copy mfg FW pcie8997_uart_combo.bin to /lib/firmware/nxp/

#insmod rfkill.ko

#insmod cfg80211.ko

#insmod mlan.ko

#insmod pcie8xxx.ko drv_mode=1 mfg_mode=1 fw_name=nxp/pcie8997_uart_combo.bin cfg80211_wext=0xf  cal_data_cfg=none



rename interface:

#ip link set wlP1p1s0 name mlan0



connect Wire line to switch or router

$ ifconfig fm1-mac4 up

$ ifconfig fm1-mac4 192.168.xx.xx



#./mfgbridge



-> follow labtool SOP


```
## Refs

- [Chip 漏洞](https://update.venuseye.com.cn/reports/1574921129597/2019112601.html)
- http://www.jinglingshu.org/?p=5322
- https://wireless.wiki.kernel.org/en/users/drivers/mwifiex