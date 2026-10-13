# 华为MatePad11解锁BL+ROOT+刷入GMS

OnePlus13手机实在是太好用了，最主要的原因还是原生谷歌服务，今年春天最早是因为手机上运行挂机小游戏有点耽误事，所以找出来了平板，想刷一个谷歌系统，替手机挂机，后来小游戏不玩了，刷系统却做了下来。

## 硬件基本信息

| 项目 | 信息 |
|---|---|
| 产品 | HUAWEI MatePad 11 |
| 型号 | **DBY-W09** |
| 地区版本 | **中国大陆 C00** |
| 网络版本 | **Wi-Fi 版** |
| EDL | **Qualcomm Emergency Download Mode 9008** |

## 教程及主要路线

主要参考了B站教程，用面具Magisk刷入，发现原来up是高中生，[华为MatePad 11 2021 解BL+Root教程](https://www.bilibili.com/video/BV1HLVa68EUv/)。概括具体流程如下：

原厂 HarmonyOS 4.2
        ↓
备份个人数据 + 关键分区
        ↓
工程线进入 Qualcomm EDL 9008
        ↓
Firehose 获得 UFS 分区读写能力
        ↓
刷入 unlock ABL
        ↓
华为低级格式化 + Factory Reset
        ↓
Bootloader → PHONE Unlocked
        ↓
处理原厂 boot
        ↓
Magisk Root
        ↓
仍然启动原厂 HarmonyOS

到这里教程内容就结束了，接下来在ChatGPT的帮助下，把GooglePlay的模块刷进去了

Magisk systemless 注入 googleplay.zip
        ↓
GSF + GMS + Play Store
        ↓
Google 账号登录、应用下载、自动更新

## 刷机过程中遇到的问题

## USB设备突然不识别

备份过程中想导出数据，一开始打算采用华为手机助手HiSuite，电脑上安装了软件之后，就识别设备了，ADB也进不去，卸载之后依然进不去。

### 排查原因

检查 Windows USB 设备：

```powershell
Get-PnpDevice -PresentOnly | Where-Object {
    $_.InstanceId -like 'USB*'
} | Format-Table Status,Class,FriendlyName,InstanceId -AutoSize
```

发现 MatePad 对应的 USB Composite Device 报错，硬件 ID 是:``USB\VID_12D1&PID_107D``，问题码是 ``Code 39 / CM_PROB_DRIVER_FAILED_LOAD``，最终定位到 HiSuite 留下来的旧 Huawei USB filter，其中xxx是电脑的编号：

```
oemxxx.inf
原始 INF：ew_usbccgpfilter.inf
Provider：Huawei
Version：1.0.9.0
```

### 解决方案

在管理员 PowerShell里删除这个驱动包，其中xxx是电脑的编号：

```powershell
pnputil /delete-driver oemxxx.inf /uninstall
```

重新拔插USB线，Windows就会自己枚举适配驱动了。

## 进入 EDL 9008 后黑屏

工程线成功让 DBY-W09 进入 EDL 后，平板是纯黑屏，而且ADB工具不识别，吓毁了，但是好在Windows能找到这个设备 Qualcomm 9008 设备，原来只是没有驱动。

## 解决方案

从 Microsoft Update Catalog 下载

```
Qualcomm Incorporated
Qualcomm HS-USB QDLoader 9008
Version 2.1.1.0
Hardware ID:
USB\VID_05C6&PID_9008
```

放在路径``D:\Downloads\
20855130_dae427ffe0a2d6268aa209d782d05ea874aad0dc.cab``。

```powershell

# 解压

mkdir C:\Qualcomm9008

expand -F:* `
"D:\Downloads\20855130_dae427ffe0a2d6268aa209d782d05ea874aad0dc.cab" `
"C:\Qualcomm9008"

# 安装

pnputil /add-driver "C:\Qualcomm9008\qcser.inf" /install

```

之后确认设备 ``Qualcomm HS-USB QDLoader 9008 (COM7)``，至此可以被windows识别。

## Root后安装原生 GMS

Magisk直接导入googleplay.zip模块。

## 遗留问题

设备未通过认证，并且自动更新到新版商店之后，就会因为未认证而锁定Play商店，删除缓存之后，会恢复到老版商店（版本26），可以下载软件。商店无法使用不会影响已经安装的软件。