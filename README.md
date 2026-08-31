# WhorlKit

**设备指纹公开挑战靶场**

> Whorl /wɜːrl/ —— 指纹的"涡纹"。WhorlKit 为每台设备产出一组完全离线计算的指纹串。
> 我们把它公开在这里，**欢迎大家来验证、攻击、修改、测试它的稳定性与唯一性**。

## 挑战目标

WhorlKit 声称以下两条性质成立，欢迎用任何手段推翻：

| # | 声明 | 攻破条件 |
|---|---|---|
| 1 | **稳定性**：同一台物理设备，无论时间流逝、重启、重装 App、网络环境变化，指纹集合保持不变 | 让**同一台设备**在未发生真实硬件变化的前提下，产出与之前**完全不同**的指纹集合（全部项都变） |
| 2 | **唯一性**：不同物理设备（含模拟器/云手机/克隆机）产出的指纹集合互不相同 | 让**两台不同设备**产出相同的指纹集合 |

判定契约（对使用者公开的全部语义）：

- 每台设备产出 **N 项指纹串**（App 首页逐项展示，`#` 编号为固定顺序，编号相同即同一项）；
- **只要任意一项没有变化，即认定设备指纹未变**；全部项变化，才视为设备变化；
- 采集与计算**完全在本机完成**：不联网、不上传、无云端记录——因此不存在"服务端兜底"，一切强度都在端上。

## 怎么开始

1. 取产物并安装（见下方签名说明）；
2. 打开 App，首页自动展示全部指纹项；「一键复制」存档，或用日志持续采集（见下方）；
3. 记录基线 → 执行你的攻击 → 重新采集 → 对比。

## 怎么看指纹

**App 界面**：首页卡片逐项展示，「一键复制」可将全部指纹（含编号）复制到剪贴板。

**日志**（每次采集自动输出，`tag = fkdxjz`）：

```bash
# Android
adb logcat -s fkdxjz

# iOS 模拟器
xcrun simctl launch booted com.sjc.JcDfpIosApp
xcrun simctl spawn booted log show --last 5m --predicate 'eventMessage CONTAINS "fkdxjz"'
```

输出形如：

```
fkdxjz: device fingerprints: N items
fkdxjz: #1: 6tSvbDdvakkracUY8DEY6DVG5cqw8ddr...
fkdxjz: #2: fOej8XWEIFrfhW6G/OIzi0MzdExcegkZ...
```

## 攻击建议（不限于此）

- **Hook / 注入**：frida、substrate、xposed 等，hook 采集函数、篡改返回值；
- **环境伪造**：修改系统属性、伪装机型、篡改文件时间/挂载信息、伪造标识符；
- **设备克隆**：模拟器快照复制、云手机镜像、整机数据迁移后对比指纹集合；
- **常规扰动**：重启、恢复出厂、卸载重装、刷机、换卡换网、时间回拨后观察指纹是否稳定；
- **静态分析**：产物经过 native 层加固，欢迎逆向——能定位并伪造全部指纹因子同样视为攻破。

任何一项攻破，请提交 Issue，附：设备/系统版本、复现步骤、攻击前后两份指纹日志（fkdxjz tag 完整输出）。

## 产物清单与安装

| 文件 | 平台 | 说明 |
|---|---|---|
| `WhorlKit-1.0.0.apk` | Android | Release 加固版，**未签名**，签名后安装 |
| `WhorlKit-1.0.0.ipa` | iOS | Release 加固版，模拟器切片，**无签名**，模拟器可直接安装 |

### Android 签名安装

```bash
# 生成密钥（已有可跳过）
keytool -genkeypair -v -keystore whorlkit.jks -alias whorlkit \
    -keyalg RSA -keysize 2048 -validity 10000

# 对齐 + 签名（apksigner/zipalign 在 Android SDK build-tools 目录下）
zipalign -f -p 4 WhorlKit-1.0.0.apk WhorlKit-aligned.apk
apksigner sign --ks whorlkit.jks --out WhorlKit-signed.apk WhorlKit-aligned.apk
apksigner verify WhorlKit-signed.apk

adb install WhorlKit-signed.apk
```

### iOS 安装

```bash
# 模拟器免签直装：解压 IPA 后
xcrun simctl install booted Payload/MpsDfpIosApp.app
```

真机使用需拥有 Apple 开发者资格者用自有证书与描述文件重签。

## 版本记录

| 版本 | 日期 | 说明 |
|---|---|---|
| 1.0.0 | 2026-08-31 | 首个挑战靶场：Android APK + iOS IPA（未签名），展示/一键复制/日志输出（tag=fkdxjz） |
