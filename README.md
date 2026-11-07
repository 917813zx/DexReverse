# DexReverse

**DexReverse** 是一款运行在 Android 手机上的 APK 分析与反编译工具。它将 JADX 反编译能力、源码浏览、ZIP 导出和一个基于 Android 系统 Shell 的内置终端整合在同一个应用中，尽量让 APK 分析流程可以直接在设备本地完成。

> 当前项目面向 Android 设备使用。项目仍在持续开发中，正式发布前请先在目标设备上完成构建与功能测试。

- **项目仓库：** <https://github.com/917813zx/DexReverse>
- **应用名称：** DexReverse
- **当前工程版本：** `1.0.0`
- **Android 最低版本：** Android 13（API 33）
- **主要语言：** Kotlin

## 功能特性

### APK 反编译与文件浏览

- 通过 Android 系统文件选择器导入 APK。
- 集成 JADX `1.5.6`，在设备本地执行反编译任务。
- 展示反编译进度和运行日志。
- 浏览生成的 Java 源码及其他输出文件，并提供文本预览。
- 将反编译结果保存到公共存储的 `Download/ApkStudio/` 目录。
- 将结果打包为 ZIP，方便备份或转移到其他设备。

### 内置终端

- 集成 Termux Terminal View / Terminal Emulator 组件和 PTY 支持。
- 使用 Android 系统自带的 `/system/bin/sh` 启动交互式 Shell。
- 支持软键盘输入、终端文本复制与粘贴。
- 在切换应用内页面时保留当前终端会话，减少重复初始化。
- 终端颜色会跟随应用当前主题。

**终端能力说明：** 内置终端运行的是应用自身 UID 下的 Android 系统 Shell，不是完整的 Termux Linux 环境，也不会自动获得 Root 权限。项目没有默认集成 Termux Bootstrap、`apt` 或 `pkg` 软件包环境。设备实际可用的命令取决于 Android 系统及其自带工具。

### 界面与设置

- 提供 Miuix 和 Material 两种界面样式。
- 支持浅色、深色和跟随系统等主题设置。
- 支持主题颜色、动态颜色及部分界面效果调整。
- 提供 Bug 日志页面，便于查看应用异常和反编译失败信息。
- 可从 GitHub 检查项目更新。

## 权限说明

DexReverse 尽可能使用 Android 系统提供的文件访问接口。不同功能所需权限如下：

| 权限 / 接口 | 用途 | 是否为 APK 反编译必需 |
| --- | --- | --- |
| Android 系统文件选择器（Storage Access Framework） | 由用户主动选择待分析的 APK | 是，用于导入文件 |
| `MANAGE_EXTERNAL_STORAGE`（所有文件访问权限） | 让内置终端 Shell 访问设备共享存储中的文件 | 否，仅终端功能需要 |
| `INTERNET` | 访问 GitHub API 检查版本更新 | 否，不影响本地反编译 |

首次进入终端页面时，如果尚未获得“所有文件访问权限”，应用会跳转到 Android 系统的应用专属设置页面。需要用户手动启用“允许管理所有文件”后返回应用，终端才会启动。该权限是 Android 的特殊应用访问权限，不是普通运行时权限。

**隐私与安全提示：** “所有文件访问权限”范围较大，请只在信任该应用并确实需要终端访问共享存储时启用。APK 选择和反编译使用系统文件选择器及应用自身工作目录，不应将该权限理解为运行反编译功能的前置条件。

## 技术栈

- **语言：** Kotlin
- **UI：** Jetpack Compose、Material 3、Miuix
- **APK 反编译：** JADX `1.5.6`
- **终端：** Termux Terminal View / Terminal Emulator `0.118.0`
- **并发：** Kotlin Coroutines
- **构建：** Gradle Wrapper、Android Gradle Plugin、Kotlin Gradle Plugin

主要工程参数：

| 项目 | 配置 |
| --- | --- |
| `namespace` | `com.example.apkstudio` |
| `applicationId` | `com.example.apkstudio` |
| `versionName` | `1.0.0` |
| `minSdk` | `33` |
| `targetSdk` | `37` |
| `compileSdk` | `37` |
| JDK | `21` |
| Gradle Wrapper | `9.7.1` |

## 构建项目

### 环境要求

- Android Studio（支持当前项目所使用的 Android Gradle Plugin）
- JDK 21
- Android SDK Platform 37
- 可联网访问 Google Maven、Maven Central 和 JitPack，以下载 Gradle 插件及依赖

### 使用 Android Studio

1. 克隆仓库：

   ```bash
   git clone https://github.com/917813zx/DexReverse.git
   ```

2. 使用 Android Studio 打开项目根目录。
3. 安装项目要求的 JDK 和 Android SDK Platform 37。
4. 等待 Gradle Sync 完成。
5. 选择 `app` 配置并运行，或通过下方命令生成 Debug APK。

### 命令行构建 Debug APK

macOS / Linux：

```bash
./gradlew assembleDebug
```

Windows：

```bat
gradlew.bat assembleDebug
```

构建产物默认位于：

```text
app/build/outputs/apk/debug/app-debug.apk
```

### 构建 Release APK

Release 构建需要配置自己的签名密钥。请复制模板：

```bash
cp keystore.properties.example keystore.properties
```

然后编辑 `keystore.properties`，填写自己的密钥库路径、密钥库密码、Key Alias 和 Key 密码。完成后执行：

```bash
./gradlew assembleRelease
```

签名信息和密钥库文件属于私密资料。**不要将 `keystore.properties`、`.jks`、`.keystore` 或密码提交到 Git 仓库。** 项目 `.gitignore` 已忽略这些本地签名文件。

> 构建环境提示：当前工程的 `gradle.properties` 中包含一项特定环境使用的 AAPT2 覆盖路径 `android.aapt2FromMavenOverride`。如果在普通 Windows、macOS 或 Linux 开发环境构建时遇到 AAPT2 路径错误，请检查该配置，并按本机环境调整或移除该覆盖项。

## 项目结构

```text
.
├── app/
│   └── src/main/
│       ├── assets/               # 应用内资源及许可证文本
│       ├── java/com/example/apkstudio/
│       │   ├── terminal/         # 内置终端与 PTY 管理
│       │   ├── component/        # 通用界面组件
│       │   ├── material/         # Material 风格组件
│       │   ├── ui/               # 页面、设置及更新检查
│       │   ├── JadxEngine.kt     # JADX 反编译入口
│       │   └── MainActivity.kt   # 应用主界面与页面导航
│       └── AndroidManifest.xml
├── gradle/                       # Gradle Wrapper 与版本目录
├── THIRD_PARTY_NOTICES.md        # 已记录的第三方组件声明
├── keystore.properties.example   # Release 签名配置模板
└── README.md
```

## 参与开发

欢迎通过 Issue 和 Pull Request 参与项目改进。

提交问题时，建议附上：

- Android 版本、设备型号及 ROM 信息；
- 复现步骤和预期结果；
- 相关错误提示或 Bug 日志；
- 修改内容及必要的测试说明。

提交代码前请确认没有包含 APK 样本、个人文件、签名密钥、访问令牌或其他敏感信息。对于反编译相关问题，请仅使用你拥有、获准分析或依法可以分析的 APK。

## 第三方项目与致谢

本项目使用或参考了以下开源项目及技术：

- [JADX](https://github.com/skylot/jadx)：DEX / APK 反编译引擎。
- [Termux](https://github.com/termux/termux-app)：终端模拟器与 PTY 相关组件。
- **Miuix UI**：项目使用 `top.yukonga.miuix.kmp` 相关 UI、图标、导航、偏好设置及模糊效果组件；请以 Gradle 依赖声明和对应上游仓库的许可证为准。
- [KernelSU](https://github.com/tiann/KernelSU)：部分界面设计与组件实现参考了 KernelSU UI。
- [AndroidLiquidGlass](https://github.com/Kyant0/AndroidLiquidGlass)：部分液态玻璃视觉效果实现参考该项目。

部分第三方组件的许可证和声明请查看 [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) 及各上游项目仓库。发布前请维护者逐项核对所有直接、间接依赖和移植代码的许可证要求，并补全必要的版权声明。

## 许可证

**本仓库当前未提供根目录 `LICENSE` 文件，项目自身代码的开源许可证尚待维护者确认。** 在添加明确许可证之前，请不要默认将本项目视为采用 MIT、Apache-2.0、GPL 或其他开源许可证。

准备公开发布时，请先选择适合项目的许可证，在仓库根目录添加对应的 `LICENSE` 文件，并确认第三方代码、资源和依赖的许可证兼容性。第三方组件仍分别受其上游许可证约束。
