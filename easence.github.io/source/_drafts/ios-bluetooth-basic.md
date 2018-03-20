---
title: iOS开发之蓝牙（Bluetooth）
---

## 开始之前
首先得在Info.plist中配置一个`NSBluetoothPeripheralUsageDescription`，否则将会crash。

## Central and Peripheral Devices 
两个角色：

- Peripheral（相当于server），提供数据
- Central（相当于client），使用数据

![](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBDevices1_2x.png)

## Centrals Discover and Connect to Peripherals That Are Advertising
1. Peripherals广播一些`advertising packet`
	- advertising packet : 包含`Peripherals`的一些有用信息，比如：名称、主要功能等。
2. `Centrals`扫描并监听`Peripherals`广播出来的信息。


## How Centrals, Peripherals, and Peripheral Data Are Represented

### Local Centrals and Remote Peripherals

![](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBObjects_CentralSide_2x.png)

 A Remote Peripheral’s Data Are Represented by CBService and CBCharacteristic Objects

![](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/TreeOfServicesAndCharacteristics_Remote_2x.png)


### Local Peripherals and Remote Centrals
![](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBObjects_PeripheralSide_2x.png)

A Local Peripheral’s Data Are Represented by CBMutableService and CBMutableCharacteristic Objects

![](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/TreeOfServicesAndCharacteristics_Local_2x.png)


 `UIBackgroundModes` key to your Info.plist file and setting the key’s value to an array containing one of the following strings:

`bluetooth-central` The app communicates with Bluetooth low energy peripherals using the Core Bluetooth framework.
`bluetooth-peripheral` The app shares data using the Core Bluetooth framework.

---

[Core Bluetooth Overview](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1)