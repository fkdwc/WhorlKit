# WhorlKit：写了个设备指纹的小东西，欢迎大家来测试、交流

> Whorl /wɜːrl/，指纹的"涡纹"。
> 个人写的设备指纹库，为每台设备产出一组完全离线计算的指纹串。
> 放出来是想请大家帮忙测一测、对比对比，也欢迎攻击修改它的设备指纹，欢迎交流。
>
> 仓库地址：**https://github.com/fkdwc/WhorlKit**

## 这是什么

就是个个人项目，平时做设备指纹相关的工作，顺手写了这么个东西。

自己手头设备有限，能测的场景不多，索性放出来，请大家帮忙看看：

- **测稳定性**：重启、重装、刷机、换网之后，指纹变不变；
- **测重复**：不同设备、模拟器、云手机上，指纹会不会撞；
- **随便折腾**：hook、伪造环境、trace 看看它采集了什么，都欢迎。

下面几条是它的设计目标，也是我自己自测时的观察——一个人测不了太多机型和场景，放出来就是想看看它在大家手里表现如何，测出问题很正常，欢迎反馈。

## 设计目标

| # | 目标 | 什么情况说明目标没达成 |
|---|---|---|
| 1 | **稳定性**：同一台物理设备，无论时间流逝、重启、重装 App、网络环境变化，指纹集合保持不变 | **同一台设备**在未发生真实硬件变化的前提下，产出与之前**完全不同**的指纹集合（全部项都变） |
| 2 | **唯一性**：不同物理设备（含模拟器 / 云手机 / 克隆机）产出的指纹集合互不相同 | **两台不同设备**产出相同的指纹集合 |

为了对比和讨论方便，先约定一个口径（免得各说各话）：

1. 每台设备产出 **N 项指纹串**，App 首页逐项展示，`#` 编号为固定顺序——**编号相同即同一项**；
2. **只要任意一项没有变化，即认定设备指纹未变**；全部项都变化，才视为设备变化；
3. 一切计算在本机完成，无云端参与。

按这个口径：改指纹要让**所有项**同时变才算改掉；两台设备要**所有项完全一致**才算重复。单项波动、部分变化都不算。

## 公开版本的一些说明

- **完全离线**：不联网、不上传、无云端记录——测试时你的任何数据都不会离开设备，可以放心随便测；
- 输出是**定长哈希串**，不直接暴露设备的原始信息；
- 没有云端参与，测的就是端上这部分，表现好不好当场就能看到；
- 采集行为可以 trace——**欢迎大家跟一遍看它到底采集了哪些信息**。

## 怎么使用

### 1. 取产物并安装

产物在仓库 [WhorlKit](https://github.com/fkdwc/WhorlKit)下，克隆或直接下载即可：

| 文件 | 平台 | 说明 |
|---|---|---|
| `WhorlKit-1.0.0.apk` | Android | Release 加固版（arm64-v8a），**未签名**，签名后安装 |
| `WhorlKit-1.0.0.ipa` | iOS | Release 加固版，模拟器切片（arm64），**无签名**，模拟器可直接安装 |

**Android 签名安装**（APK 未签名，需用自有密钥签一下）：

```bash
# zipalign/apksigner 一般不在 PATH 里，先加一下（版本号按本机 build-tools 实际调整）
export PATH="$HOME/Library/Android/sdk/build-tools/35.0.0:$PATH"   # macOS
# export PATH="$HOME/Android/Sdk/build-tools/35.0.0:$PATH"         # Linux

# 生成密钥（已有可跳过，交互式按提示填即可）
keytool -genkeypair -v -keystore whorlkit.jks -alias whorlkit \
    -keyalg RSA -keysize 2048 -validity 10000

# 对齐 + 签名 + 校验
zipalign -f -p 4 WhorlKit-1.0.0.apk WhorlKit-aligned.apk
apksigner sign --ks whorlkit.jks --out WhorlKit-signed.apk WhorlKit-aligned.apk
apksigner verify WhorlKit-signed.apk

# 如果设备上装过别的签名的旧版本，先卸载，否则会报 INSTALL_FAILED_UPDATE_INCOMPATIBLE
adb uninstall com.jb.whorlkit

adb install WhorlKit-signed.apk

# 启动 App
adb shell am start -n com.jb.whorlkit/.MainActivity
```

**iOS 安装**（模拟器免签直装，解压 IPA 后）：

```bash
xcrun simctl install booted Payload/MpsDfpIosApp.app
```

真机使用需拥有 Apple 开发者资格者用自有证书与描述文件重签。

### 2. 看指纹

**App 界面**：打开 App，首页自动展示全部指纹项（`#1`、`#2`……编号固定）。两个按钮：

- **重新采集**：强制重跑整条采集链，用来看"同设备反复采集是不是同结果"；
- **一键复制**：将全部指纹（含编号）复制到剪贴板，方便存档和贴 Issue。

**日志**（每次采集自动输出，`tag = fkdxjz`）：

```bash
# Android
adb logcat -s fkdxjz

# iOS 模拟器
xcrun simctl launch booted com.jb.whorlkit
xcrun simctl spawn booted log show --last 5m --predicate 'eventMessage CONTAINS "fkdxjz"'
```

输出形如（每项为定长哈希串，Android 与 iOS 的前缀格式略有差异）：

```
# Android（logcat）
fkdxjz: device fingerprints: 11 items (tag=fkdxjz)
fkdxjz: #1: ir1oFfox3yEnLiNAnGLiVuxqRS23KSTe0VU1bHoiiId
fkdxjz: #2: 6s66xVjK5TL+g5f6LZ2ECuGLV79NsZsLolMtcaB5wpA

# iOS（模拟器 unified log）
[fkdxjz] device fingerprints: 9 items
[fkdxjz] #1: EJuXLKMajtdgS2wr5F1JhB3WkzCypNqX9mOvEoR8Znk
```

两个实测小提醒（我在模拟器上跑出来的现象，供参考）：

- 卸载重装后，其中一项（预埋的随机标识）会重新生成，其余项保持不变——按上面的判定口径指纹未变，属正常现象；
- 不同设备/系统上个别因子可能取不到，项数会有差异（我这边 Android 模拟器 11 项、iOS 模拟器 9 项）；同一台设备的项数是稳定的，两边项数不一致时，先点「重新采集」再对比。

### 3. 建议的测试流程

```
先重启一两次 App，等指纹项数稳定
    ↓
记录基线（一键复制 / 日志存档）
    ↓
执行你的测试（hook / 环境伪造 / 克隆 / 刷机 ……）
    ↓
点「重新采集」重新取指纹
    ↓
对比：哪些编号的项变了？是否全部项都变了？
```

做多设备**对比测试**时同理：两台设备各取一份指纹，逐编号比对是否完全一致。

## 一些可以试的方向（不限于此）

- **Hook / 注入**：frida、substrate、xposed 等，hook 采集函数、篡改返回值；
- **行为审计 / Trace**：用 strace、frida-trace 等工具跟一遍采集行为，看它到底读取了设备的哪些信息——trace 出完整采集清单并逐项分析，也挺有意思；
- **环境伪造**：修改系统属性、伪装机型、篡改文件时间 / 挂载信息、伪造各类标识符；
- **设备克隆**：模拟器快照复制、云手机镜像、整机数据迁移，之后对比指纹集合；
- **常规扰动**：重启、恢复出厂、卸载重装、刷机、换卡换网、时间回拨，看指纹是否稳定；
- **静态分析**：产物经过 native 层加固，感兴趣可以逆向看看。

如果测出了问题，或者有想法想聊，欢迎到 [github.com/fkdwc/WhorlKit/issues](https://github.com/fkdwc/WhorlKit/issues) 提 Issue，最好附上：

1. 设备 / 系统版本；
2. 复现步骤（越细越好）；
3. 测试前后两份指纹日志（`fkdxjz` tag 完整输出）。

## 欢迎交流

如果你也在做设备指纹相关的工作，欢迎找我聊聊（个人行为，非官方）：

- **风控 / 反欺诈从业者**：想在业务里做指纹相关测试，或对比自研与其他方案；
- **安全研究者**：想测攻防、云手机 / 模拟器识别、克隆检测之类的场景；
- **厂商与团队**：需要测试环境、评估用例或联合测试。

补充说明：仓库公开的是**离线测试版**；**完整版 SDK 是需要联网的**，包含降级预案等工程稳定性手段，适合真实业务接入，欢迎联系了解。

**联系方式**：`私信，留言`

也欢迎直接在仓库 [WhorlKit](https://github.com/fkdwc/WhorlKit) 提 Issue 交流——测出的问题、不稳的场景、对比数据，都欢迎。

## FAQ

**Q：为什么不联网？**
A：主要是为了数据安全——这个版本不联网、不上传，测试的时候不用担心你的任何数据被收集，可以放心随便折腾。另外完整版是联网的，带降级预案等工程稳定性手段；公开的离线版去掉了联网部分，只保留端上采集计算这块。

**Q：指纹串能看到我设备的什么信息？**
A：输出是定长哈希串，不直接暴露设备原始信息。你只需要关心它"变没变"、"和别的设备是否相同"。

**Q：我怎么知道它到底采集了我设备的哪些信息？**
A：欢迎 trace。这个版本完全离线，用各种手段和工具跟一遍系统调用和 API 行为，就能看到它读了哪些信息——采了什么可以自己看，不用猜。

**Q：改了一部分指纹项，算改掉了吗？**
A：按上面的口径不算：任一项没变，即认定指纹未变；全部项变化，才视为设备变化。反向同理，两台设备要完全一致才算重复。

**Q：我可以拿它和其他方案对比吗？**
A：欢迎——同机安装、同样扰动流程、各自记录指纹变化情况，对比数据如果愿意分享，那就更好了。

## 版本记录

| 版本 | 日期 | 说明 |
|---|---|---|
| 1.0.0 | 2026-08-31 | 首个公开测试版：Android APK + iOS IPA（未签名），展示 / 一键复制 / 日志输出（tag=fkdxjz） |
