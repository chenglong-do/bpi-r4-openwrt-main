# Banana Pi BPI-R4 OpenWRT自动构建

[![LICENSE](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square&label=LICENSE)](https://github.com/IncubusRK/openwrt-ax6s/blob/master/LICENSE)

- [BPI-R4入门指南](https://docs.banana-pi.org/en/BPI-R4/GettingStarted_BPI-R4)
- [4PDA刷机教程](https://4pda.to/forum/index.php?showtopic=1093476&view=findpost&p=133806031)

## 无需RS-232串口连接的全新刷机流程

由于该设备的eMMC与SD卡无法同时工作，刷机需分阶段完成:  
- 首先将固件写入SD卡，再从SD卡写入NAND，最后从NAND写入eMMC。
- 后续更新可通过Web界面使用文件[openwrt-mediatek-filogic-bananapi_bpi-r4-squashfs-sysupgrade.itb](openwrt-mediatek-filogic-bananapi_bpi-r4-squashfs-sysupgrade.itb)完成。

1. 下载[openwrt-mediatek-filogic-bananapi_bpi-r4-sdcard.img.gz](releases/latest/download/openwrt-mediatek-filogic-bananapi_bpi-r4-sdcard.img.gz)。注意解压后的镜像文件需小于128MB，否则无法写入NAND。
2. 使用[Rufus](https://rufus.ie/)或[balenaEtcher](https://etcher.balena.io/)将镜像写入SD卡。
3. 将启动开关SW3设置为1:1（两个开关均处于下方位置），以从SD卡启动。
4. 插入SD卡并启动设备。
5. 启动后通过SSH连接设备（默认地址192.168.1.1，用户名root，无密码）。
6. 输入以下命令重启设备并自动将固件从SD卡复制到NAND：
   ```sh
   fw_setenv bootcmd "env default bootcmd ; saveenv ; run ubi_init ; bootmenu 0"
   reboot
   ```
7. 等待固件复制完成并设备重启。可通过 SSH 连接确认是否成功（可能需使用ssh-keygen -R 192.168.1.1清除旧密钥）。
8. 关闭设备并移除 SD 卡。
9. 将 SW3 设置为 0:1（左开关向上，右开关向下）以从 NAND 启动。
10. 通过 SSH 连接并输入以下命令：
    ```sh
    fw_setenv bootcmd "env default bootcmd ; saveenv ; run emmc_init ; bootmenu 0"
    reboot
    ```
    设备重启后会自动将固件写入 eMMC。
11. 等待启动完成后关闭设备，将 SW3 设置为 1:0（左开关向下，右开关向上）以从 eMMC 启动。
    
## 调整系统分区大小

从 eMMC 启动后，可调整系统分区大小（默认仅占用部分空间）：  
    
    通过 SSH 连接并执行：cfdisk /dev/mmcblk0  
    使用方向键选择/dev/mmcblk0p5分区，选择Resize调整大小  
    输入目标尺寸后执行Write保存变更  

注意：建议保留部分未分配空间以避免备份文件过大。也可使用内置工具parted操作。   
若调整失败，需从 NAND 启动（SW3 设为 0:1）执行操作，完成后恢复 SW3 至 eMMC 启动状态。  
最简单的生效方式是通过 Web 界面（System -> Backup / Flash Firmware）刷写.itb升级包。  

## 启用 RTC 实时时钟

安装 CR2032 电池（需焊接 2pin 1.25mm 接口）   
执行命令：  
```sh
fw_setenv bootconf_extra mt7988a-bananapi-bpi-r4-rtc
```
重启后应出现/dev/rtc0设备，可通过hwclock命令验证功能。

## 特别感谢

- [IncubusRK](https://github.com/IncubusRK/openwrt-bpi-r4)
