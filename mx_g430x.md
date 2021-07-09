# CAT4 G430x BSP
###### tags: `G4300` `CAT4`

[TOC]


## RCW 修改

> CPU 參數調整

### Ex1. I2C2 的修改

- Datasheet P239
    - ![](https://i.imgur.com/u5KBomA.png)


- 修改值的檔案 ```rcw_1000_default.rcw```
    - 如果有的話就直接改成 10 進位值
- 定義值的檔案 ```ls1012a.rcwi```
    - ![](https://i.imgur.com/gNGLZ9P.png)
        - 424-425 bit

- uboot device tree 可能需要做修改

### 選擇不同 cpu frequency 的 RCW

- 更改 menuconfig rcw package config
    - BR2_PACKAGE_HOST_QORIQ_RCW_BIN
- 更改 menuconfig Bootloader config for arm-trusted-firmware
    - BR2_TARGET_ARM_TRUSTED_FIRMWARE_ADDITIONAL_VARIABLES


## DDR Tuning

- package 
    - arm-trusted-firmware 

## CPU Test Item

- Reset
    - ```reset``` by uboot

## RAM

- R/W
    - ```mtest``` by uboot

## EMMC

- Boot from EMMC

- Detect
    - ```mmcinfo``` by uboot
    - eMMC和一般硬碟類似，分區資訊位於 mmcblk0 的 0 扇區，內核不負責分區的創建，僅僅是讀0扇區MBR及分區表即來獲得分區資訊。

## Switch

- Test MDIO interface
    - 


## E-test Auto check

- Ethernet
- MCU
- Serial
    - ```modprobe ti_usb_3410_5052```
- Cellular
- Wifi

## Kernel Feature Research

- DSA
- Overlayfs
- ebpf

## Refs

- [Wifi 認證]()
- [Cellular 認證]()
- [Sierra Cellular module - WPXX](https://hackmd.io/N2osOA81RdK0Ecz0dHSwrw)
- [NXP CPU - LS1012aRDB](https://hackmd.io/bN7Ah69uTRmTmSDkABcgeA)
- [Wifi - Marvell 88W8997 2*2 ac](https://hackmd.io/bEjtXlBZSPyLNe8ES-TWgA)
- [MCU - Atmel SAM D10](https://hackmd.io/y2gPkLoVTLKv2RB40co-lA)
- [Switch chip - Realtek RTL 8309, 8364M, 8370](https://hackmd.io/gomZbuHTT6KF_XNlld4A-Q)
    - BSP tool
        - switchdev 
            - https://houmin.cc/posts/1602ea65/
            - https://elinux.org/images/8/88/Belloni-Switchdev-Slides-ELC2018.pdf
            - https://www.hwchiu.com/2016-03-27-switchdev-i.html
        - swconfig 
        - Kernel Config
            - NET_DSA
            - NET_SWITCHDEV
- [USB to UART - TUSB3410](https://hackmd.io/2a9AcC6CTOuWCAqkD-EwcA)
    - DUT 做 Client 送訊號出去 (一般 UART 是 PC 做 Client 送訊號進來)
    - 照舊
- [EEPORM - ]
- [EMMC - ]
    - ```boot_emmc```
    - https://www.itread01.com/content/1549211979.html
- [QSPI - Winbond W25Q64JW](https://hackmd.io/H2SkEwg0TUCo5fOk66cdqA)
