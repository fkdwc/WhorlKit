# WhorlKit

**完全离线的设备指纹（Offline Device Fingerprint）**

> Whorl /wɜːrl/ —— 指纹的"涡纹"。每一台设备的指纹涡纹独一无二，正如每一项 WhorlKit 指纹串对应一台真实设备。

## 这是什么

WhorlKit 在设备本机采集多项稳定特征，为每台设备产出一组**指纹串**。它**完全离线**：

- 采集与计算全部在本机完成，**不联网、不上传、无云端记录**；
- 指纹的记录、调试与观察由使用方自行完成（App 内可一键复制，日志可持续输出）；
- native 层经过高强度代码加固，不可剥离、不可独立复用。

## 产物清单

| 文件 | 平台 | 说明 |
|---|---|---|
| `WhorlKit-1.0.0.apk` | Android | 演示应用，Release 加固版，**未签名**（安装前需自行签名） |
| `WhorlKit-1.0.0.ipa` | iOS | 演示应用，Release 加固版，模拟器切片，**无签名**（模拟器可直接安装） |

## 快速上手

安装并打开 App，首页自动采集并展示全部指纹项，无需任何操作。

### 怎么用（判定规则）

- 列表中的**每一项都是该设备的设备指纹**；
- **只要其中任意一项没有变化，即认定设备指纹未变**；全部变化，才视为设备发生了变化；
- 各项**顺序固定**，`#` 编号相同的永远是同一项指纹，跨次对比请按编号对位；
- 底部「重新采集」可随时重跑一次采集，验证多次结果的一致性。

### 怎么看指纹

**方式一：App 界面**——首页卡片逐项展示，点「一键复制」可将全部指纹（含编号）复制到剪贴板，方便存档对比。

**方式二：日志**——每次采集都会输出日志，`tag = fkdxjz`：

```bash
# Android（adb 连接设备后）
adb logcat -s fkdxjz

# iOS 模拟器
xcrun simctl launch booted com.sjc.JcDfpIosApp
xcrun simctl spawn booted log show --last 5m --predicate 'eventMessage CONTAINS "fkdxjz"'
```

日志输出形如：

```
fkdxjz: device fingerprints: N items
fkdxjz: #1: 6tSvbDdvakkracUY8DEY6DVG5cqw8ddr...
fkdxjz: #2: fOej8XWEIFrfhW6G/OIzi0MzdExcegkZ...
...
```

## 怎么签名

### Android（APK 必须签名后才能安装）

产物未签名，接收方用自己的 keystore 重签：

```bash
# 1. 生成签名密钥（已有可跳过）
keytool -genkeypair -v -keystore whorlkit.jks -alias whorlkit \
    -keyalg RSA -keysize 2048 -validity 10000

# 2. 对齐（zip 级处理后的 APK 必须重新对齐）
zipalign -f -p 4 WhorlKit-1.0.0.apk WhorlKit-aligned.apk

# 3. 签名（SDK build-tools 目录下）
apksigner sign --ks whorlkit.jks --out WhorlKit-signed.apk WhorlKit-aligned.apk

# 4. 校验并安装
apksigner verify WhorlKit-signed.apk
adb install WhorlKit-signed.apk
```

### iOS

- **模拟器**：无需签名。解压 IPA 取出 `Payload/MpsDfpIosApp.app`，执行
  `xcrun simctl install booted Payload/MpsDfpIosApp.app` 即可；
- **真机**：本 IPA 为模拟器切片。真机使用需要拥有 Apple 开发者资格的接收方，用自有证书与描述文件对 App 重签后再安装分发。

## 集成说明

本仓库分发演示产物。指纹能力本身以**源码内嵌**形态集成进宿主 App（与宿主共同编译，无独立 SDK 产物，不可剥离直接使用），集成方式请联系维护者。

## 版本记录

| 版本 | 日期 | 说明 |
|---|---|---|
| 1.0.0 | 2026-08-31 | 首个公开产物：Android APK + iOS IPA（未签名），App 内展示 / 一键复制 / 日志输出（tag=fkdxjz） |
