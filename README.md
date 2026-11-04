# 知食 NutriTrack

离线优先的食物营养成分查询与个人饮食跟踪工具。一套 C# / Avalonia 代码同时跑 **Windows 桌面** 与 **Android**，查成分、记三餐、看建议都不需要联网。

![首页](docs/shots/android-home.png)

## 它能做什么

| 功能 | 说明 |
| --- | --- |
| 食物搜索 | 关键字模糊匹配，支持拼音首字母；本地库覆盖上千条常见食物与菜品 |
| 拍照识别 | 拍一张餐盘照片，交给在线模型猜出是什么菜，再落到成分查询 |
| 分类浏览 | 按谷物、肉类、蔬菜、水果等大类逐层翻，每件食物都有完整的营养明细 |
| 个性化建议 | 每份食物按当前档案给出建议用量与注意事项：GI/GL、嘌呤、草酸、钠、蛋白……切换成员建议跟着变 |
| 多成员档案 | 性别、年龄、身高体重、活动量、特殊病症分开存，一家人共用一个程序 |
| 用餐记录 | 早中晚加餐逐条记，当日热量与三大营养素实时汇总 |
| 每日建议 | 对照《中国居民膳食指南》给出当天缺什么、下一餐怎么补，还能推荐食谱 |
| 收藏与组合 | 常吃的东西收进收藏，一餐的搭配存成组合，下次一键复原 |
| 备份与导出 | 档案、收藏、记录、设置打成一个包，换设备或重装前一键导出 |
| 在线补充 | 内置 USDA、中国食物成分表、香港食物安全中心三个数据源，本地查不到时可以联网补 |
| 自动更新 | 启动后静默检查新版本；Windows 直接覆盖当前安装目录，Android 交给系统安装器 |

## 下载与安装

所有安装包都在 [Releases](https://github.com/VrezenStrijder/NutriTrack/releases) 页面。

| 平台 | 文件 | 说明 |
| --- | --- | --- |
| Windows 10 / 11（64 位） | `NutriTrack-x.y.z-windows10-x64-setup.exe` | 双击安装，装完直接能用，不需要预装 .NET |
| Windows 7 SP1 / 8.1（64 位） | `NutriTrack-x.y.z-windows7-x64-setup.exe` | 需打齐 KB2670838 / KB2999226 / KB2533623，缺一项双击没反应 |
| Android 6.0 及以上 | `NutriTrack-x.y.z-android-arm64.apk` | 适用于 2017 年以后的手机与平板；首次安装需允许「安装未知应用」 |

程序本身**没有代码签名**，Windows 首次运行会被 SmartScreen 拦一下，点「更多信息 → 仍要运行」即可。

> Android 的 x86_64 包只用于模拟器调试，不对外发布。

安装位置默认挑空间最大的非系统盘，装在 `...\Program Files\NutriTrack`；也可以在安装界面里改。程序主体是绿色可运行的，安装包只是把它放到合适的位置并登记卸载项。

## 更新怎么工作

- 启动几秒后静默检查一次 GitHub 上的 [`update.json`](https://raw.githubusercontent.com/VrezenStrijder/NutriTrack/main/update.json)，有新版才提示，没网不打扰。
- 发现新版时，电脑上会在**屏幕右下角**弹一张提示卡（不在程序窗口里），点一下就开始下载。
- 下载完先核对 SHA-256，对不上直接丢弃重来。
- Windows 会**装回你当前程序所在的目录**，不需要再选路径，也不会装出第二份；装完自动重新打开。
- Android 下载完拉起系统安装器，授权后即为新版本。
- 不想自动装就点「打开发布页」自己下。免安装（解压即用）的那一份识别不到安装信息，按钮会变成「前往下载」。

## 数据放在哪

| 内容 | 位置 |
| --- | --- |
| 用户数据（档案、收藏、记录、设置） | Windows：`%LOCALAPPDATA%\NutriTrack`；Android：应用私有目录 |
| 备份包 | 同上目录下的 `backups`，也可以在设置页导出到任意位置 |
| API Key | 与用户数据同库，经系统凭据保护（Windows DPAPI / Android Keystore）加密后存储 |

程序不向任何服务器上报使用数据。在线查询只在你主动搜索或点「刷新在线」时才发请求，用的是你自己填的 Key。

## 从源码构建

前置：

- .NET 8 SDK（`global.json` 已钉住 SDK 版本）
- 打 Android 包还需要 `dotnet workload install android`，以及本机生成的签名配置（`tools/make-keystore.ps1`）

```bash
# 编译全部
dotnet build src/NutriTrack.sln

# 跑起来（Windows）
dotnet run --project src/NutriTrack.Desktop

# 跑测试
dotnet test src/NutriTrack.Tests
```

打发布包用仓库里的发布器（图形界面或命令行都行）：

```bash
# 一次发布：Windows + Android 真机包，并写出 update.json
NutriTrack.Publisher --publish --version 0.3.0 --out artifacts/release \
  --target windows10-x64-setup --target windows7-x64-setup --target android-arm64-apk \
  --name "知食 NutriTrack" --notes "本次更新内容"

# 再加 --upload 就会传到 GitHub Release 并把清单提交回 main
NutriTrack.Publisher --publish ... --upload
```

`--name` 是显示名，可以写中文；**文件名只取其中的英文部分**（上面产出的是 `NutriTrack-0.3.0-…`），因为 GitHub 上传时会把 asset 名里的中文前缀截掉，而 `update.json` 里的下载地址是按文件名拼的。

界面上的「生成一键脚本」会把当前勾选写成 `release-0.3.0.ps1`，参数与上面完全一致，双击即可重跑同一次发布。

版本号只有 `src/Directory.Build.props` 一个来源，Android 的 `versionCode` 由它派生（`主×10000 + 次×100 + 修订`）。

## 项目结构

| 目录 | 内容 |
| --- | --- |
| `src/NutriTrack.Core` | 领域模型、查询与建议算法、更新清单解析。不依赖 UI |
| `src/NutriTrack.Infrastructure` | EF Core + SQLite、仓储、在线数据源、更新下载与校验 |
| `src/NutriTrack.UI` | Avalonia 界面与 ViewModel，Windows 与 Android 共用 |
| `src/NutriTrack.Desktop` | Windows 宿主：托盘、系统通知、静默安装 |
| `src/NutriTrack.Android` | Android 宿主：安装器拉起、分享、闹钟、相机 |
| `src/NutriTrack.Setup` | 安装器外壳，与负载拼成一个 setup.exe |
| `src/NutriTrack.Packaging` | 打包引擎：目标映射、校验、产物归档 |
| `src/NutriTrack.Publisher` | 打包工具的界面与命令行入口 |
| `src/NutriTrack.Tests` | 单元测试（600+ 条） |
| `docs` | 开发计划、数据说明、发布流程、用户手册 |
| `tools` | 数据生成、签名、清单生成等脚本 |

## 文档

| 文档 | 内容 |
| --- | --- |
| [用户手册](docs/14-用户手册.md) | 面向使用者的完整操作说明 |
| [发布打包](docs/10-发布打包.md) | 打包工具怎么用、产物长什么样、更新清单怎么发 |
| [数据说明](docs/02-数据说明.md) | 离线库的来源、表结构与字段含义 |
| [在线数据源](docs/05-在线数据源.md) | 三个外部接口的申请与限速 |
| [开发计划](docs/01-开发计划.md) | 分阶段的任务与验收标准 |
| [Android 冒烟记录](docs/13-Android冒烟记录.md) | 每一轮在真机/模拟器上的实测结果 |

## 已知限制

- 没有代码签名，SmartScreen 会提示；企业分发前需要自备证书并重新签名。
- Win7 / 8.1 不在 .NET 8 的官方支持范围内，实测可启动，但系统补丁缺一不可。
- 升级是整包替换，没有差分更新，每次约 100 MB 以上。
- Android 的 x86_64 包仅供模拟器，真机请装 arm64。
- 拍照识别依赖在线模型，离线时只会走本地关键词匹配。

## 许可

个人项目，暂未附开源许可证。引用或再分发前请先联系作者。
