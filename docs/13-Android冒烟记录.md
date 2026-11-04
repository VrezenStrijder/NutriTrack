# 13 · Android 冒烟记录（W9.6）

> 每轮发版按 `docs/12` §11.2 的 18 条逐项过一遍，结果填在本文件末尾新增的一节里。
> 本轮为 **首轮**，跑在模拟器上；凡依赖真机/真 Key 的条目单列，不混入「通过」。

## 一、本轮信息

| 项 | 值 |
|---|---|
| 日期 | 2026-10-30 ～ 10-31 |
| 设备 | Pixel 7 AVD（`pixel_7_-_api_34`），API 34，x86_64 |
| 包 | Release 自签，`-Abi android-x64`，44.2 MB，versionCode 200 / versionName 0.2.0 |
| 签名 | `CN=NutriTrack, OU=FoodInventory, O=FoodInventory, C=CN` |
| 构建 | `tools/build-android.ps1 -Configuration Release -Package apk -Abi android-x64` |
| 数据 | 新建档案「Smoke（男 · 30 岁 · 170 cm / 65 kg）」+ 1 条午餐记录（火腿蛋三文治 100 g） |
| 崩溃 | crash buffer 全程为空（0 次 FATAL / 0 次 ANR），进程 pid 2563 全程未重启 |
| 结论 | 18 条中 **通过 11 / 部分通过 1 / 未测 5 / 未确认 1**；无阻断性问题。后续轮次逐条补验，最终成绩见 §9.4（通过 13 / 部分通过 2 / 未测 3） |

## 二、逐条结果

### 通过（11）

| # | 条目 | 证据 |
|---|---|---|
| 2 | 详情 → 份量 → 加入午餐 → 记录页出现 → 能量变化 | 提示「已加入 10 月 30 日午餐：火腿蛋三文治 100 g，约 240 kcal」；记录页 240 / 2155 kcal、Atwater 236（−2%）、蛋白质 11 g 完成 17%、7 日柱状图 10/30=240。`18-track-after-log.png` |
| 8 | 飞行模式：搜索、记录、汇总、建议、库自检可用 | `airplane_mode_on=1` 下搜索离线库 12 条、三个在线源显示「失败」；库自检通过「离线成分库 7648 条（精简库）· 构建于 2026-10-27T15:17:09」。`26-airplane-search.png`、`30-db-check.png` |
| 9 | 字号放大 | 应用内「大」：首页 / 记录 / 设置均无截断、无重叠；系统 `font_scale=1.3` 对自绘 UI 基本无影响（见观察 3）。`23-font-large-home.png`、`25-system-font-home.png` |
| 10 | 深色 / 跟随系统 | 切深色后首页、设置、记录、详情对比正常，无残留浅色块；已切回「跟随系统 · 标准字号 · 森林绿」。`20-dark-settings.png`、`21-dark-home.png` |
| 11 | 返回键逐级 + 「我的」子页返回 | 收藏 → 返回 → 「我的」；设置 → 返回 → 「我的」；一级页返回 → 桌面且进程存活（pid 7196）。W9.5 的 `PushPage` 修复生效 |
| 16 | SAF：导出单个 `.zip`；导入走完预览与合并 | 导出默认名 `NutriTrack备份-20261030-1550.zip`（**无 `.zip.zip`**），落 `/sdcard/Download` 1797 B；目录里 `...zip.zip` 是 10-29 修复前的遗留。导入：预览显示「导出于 2026-10-30 15:50 · 程序 0.2.0 · 含 1 个档案 / 0 条收藏 / 1 条记录」「当前库已有 1 个档案、1 条记录；同名档案 1 个、重复记录 1 条」，执行后提示「导入完成」。`32-saf-export.png`、`36-import-preview.png`、`49-import-complete.png` |
| 17 | Release 签名与版本号 | 覆盖安装报签名冲突，`adb uninstall` 后安装成功；设置页显示 v0.2.0，`dumpsys package` 为 versionCode=200 / versionName=0.2.0。`19-settings-bottom.png` |
| 18 | 体积与签名 DN | arm64 APK **42.1 MB** / AAB **39.7 MB**（≤ 60 MB）；`apksigner` DN = `CN=NutriTrack` |
| 7 | 后台提醒 → 通知 → 点击 → 「10 分钟后」 | 把餐食提醒改到 12:40 后把应用压到后台（`keyevent 3`）。12:42 通知到达：标题「今天的饭还没记」、正文「记一条就能看到今天还差多少蛋白质、膳食纤维和钠。当前设定 12:40 提醒。」、三项动作「去记录 / 10 分钟后 / 知道了」，`dumpsys` 里 `id=4001 channel=nutritrack.reminder actions=3`。点「10 分钟后」→ 通知立即撤回，闹钟重排到 `12:55:54`（原 12:45:57，正好 +10 min），12:55:54 再次投递同一条。`w97-notify-reminder.png`、`w97-notify-actions.png` |
| 12 | 通知点击（冷 / 热启动） | 点通知主体 → `topResumedActivity=com.nutritrack.app/...MainActivity`，应用内浮出「今天的饭还没记」卡片（带「10 分钟后」与「去记录」）；点卡片「去记录」正确落到「记录」页（底栏「记录」高亮、今天 0 / 2155 kcal、7 天柱图 10/29=240）。`w97-notify-click.png`、`w97-card-goto.png` |
| 5 | 导出 → **卸载重装** → 导入 | 建「BackupA」档案 + 1 条收藏 + 1 条午餐（苹果均值 150 g）+ 1 条体重（68.5 kg / 82 cm / 备注 morning）→ 导出 `NutriTrack备份-20261101-1328.zip`（1845 B）→ `adb uninstall` → 重装（库清空，「尚未创建档案」）→ SAF 选中备份 → 策略「覆盖」→「导入完成 新增档案 1 个，收藏 +1，记录 +1，体重/腰围 +1。」四类数据逐项对得上，体重记的时间戳 `11-01 13:20` 与备注都在。详见 §八 |

### 部分通过（1）

| # | 条目 | 已验 / 未验 |
|---|---|---|
| 1 | 冷启动 → 搜索 → 命中数 → 首屏 1.5 s | 已验：首装冷启 7.06 s（含首次解压 7,648 条成分库）、二次冷启 5.25 s、热启 0.71 s；搜索「fq / fanqie」回 12 / 10 条，「鸡蛋」离线 60 条（共匹配 65）。未验：「番茄」中文命中数——`adb input text` 无法注入中文，直查离线库 `food.name like '%番茄%'` 为 **38 条**（≥ 20 成立）。模拟器耗时不能代表真机首屏 |

### 未测（5，环境限制）

| # | 条目 | 原因 |
|---|---|---|
| 3 | 建议页「糖尿病 + 高血压」条目 | 需先给档案勾选病症；本轮档案为默认健康态，只验到详情页「给「Smoke」的建议 + 建议份量」正常渲染 |
| 4 | 导出建议报告 → 分享 → 打开 | 依赖分享链路（见 #15） |
| 6 | 拍照 → 5 条候选 → 采纳 | 无可用视觉模型 Key。未配置时点「拍菜品」不动作并显示「还没有可用的视觉模型」，守卫正常 `50-camera-launch.png` |
| 13 | 相机 `ACTION_IMAGE_CAPTURE` | 同上，被模型守卫拦在拉起之前 |
| 14 | 相册 `ACTION_PICK` | 同上 |

### 未确认（1）

| # | 条目 | 现象 |
|---|---|---|
| 15 | 分享图片 / 分享文字拉起分享面板 | 10/31 当天无记录时两个按钮正确置灰；切到 10/30（240 kcal）后点击，多次轮询 `topResumedActivity` 始终是 `MainActivity`，未出现 Chooser，logcat 无异常无崩溃。**待复验**：无法确定是点击落点问题、`IShareService` 的 Intent 未被模拟器接收，还是截图/导出耗时超过等待窗口 |

## 三、过程中发现的问题

| 级别 | 现象 | 影响 | 建议 |
|---|---|---|---|
| 低 | 新建档案后，**同一会话内已打开**的食物详情页不会出现「加入今日餐次」，重新进入详情页才出现 | 首次配置档案的用户会以为不能记餐 | `FoodDetailViewModel.CanLog` 是计算属性且无变更通知；改为在 `_session.Changed` 里 `OnPropertyChanged(nameof(CanLog))`。已有档案的路径不受影响。**W9.7 已修**（`Changed` 回调 + 每次 `LoadAsync` 收尾各补一次通知），模拟器复验通过 |
| 低 | 设置页底部按钮（「允许通知」「选择备份文件…」）落在底栏覆盖区，需再滚一点才能点中 | 用户可能点空 | 内容区 `Padding` 底部留出底栏高度（现为 `24,16,24,32`） |
| 观察 | 「在线数据源」里 API Key 输入框在空态下被无障碍树暴露为 `by by by …`（24 个掩码字符） | 屏幕阅读器会读出噪音 | 空态不要给 `PasswordChar` 填充；或给 `AutomationProperties.Name="API Key"` |
| 观察 | 拼音缩写召回少于中文直查（`fq`→12、`fanqie`→10，中文「番茄」直查 38） | 只用拼音找长尾食物会漏 | 属检索策略取舍，暂不改；若要改，给拼音匹配加同义词扩展 |
| 环境 | 三个在线源全部 `Connection failure` | 与本轮无关（宿主无外网）；离线优先路径按设计工作 | 真机复验时确认在线源能出结果 |
| 环境 | 系统 `font_scale=1.3` 对自绘 UI 几乎无影响，仅搜索框占位提示被省略号截断 | Avalonia 自绘特性 | 已在应用内提供「小 / 标准 / 大」三档，够用 |

## 四、仅真机可验的遗留项

1. 杀进程后闹钟到点仍发通知（`AlarmManager` 兜底 + 厂商 ROM 后台限制）
2. ~~通知点击的冷 / 热启动落点，「10 分钟后」二次提醒~~ —— §七 已在模拟器验通
3. 相机、相册（需真实相机 App 与相册）、> 20 MB 备份包的 `content://` 读取
4. ~~分享面板真正弹出并选中具体接收应用~~ —— 面板已验（§9.1 #15），只剩「选中具体应用并真发出去」
5. 首屏 < 1.5 s（模拟器数据不可用）
6. `bundletool` 校验 `aab`（本机未装）

## 五、复跑方法

```powershell
# 1. 构建 x64 测试包（模拟器）或 arm64 发版包（真机）
./tools/build-android.ps1 -Configuration Release -Package apk -Abi android-x64

# 2. 卸载旧签名再装
$adb = "$env:ANDROID_HOME/platform-tools/adb.exe"
& $adb uninstall com.nutritrack.app
& $adb install -r src/NutriTrack.Android/bin/Release/net8.0-android/android-x64/com.nutritrack.app-Signed.apk

# 3. UI 断言辅助（按文本点击 / 校验存在）
python artifacts/smoke/ui.py tap "设置"
python artifacts/smoke/ui.py has "加入今日餐次"

# 4. 崩溃与前台确认
& $adb logcat -d -b crash
& $adb shell dumpsys activity activities | Select-String topResumedActivity
```

截图与 UI 树归档在 `artifacts/smoke/w96/`（`*.png` + 同名 `.xml`），文件名序号对应本文件里的证据引用。

---

## 六、第二轮：首轮真机反馈修复（2026-11-01）

第一轮的结论是「无阻断性问题」，但那只覆盖了模拟器能验的路径。用户装机跑了一轮，
反馈 5 条，其中第 1 条是 Release 特有的硬伤——**Debug 包永远验不出来，因为工具链替它补了权限**。

| 项 | 值 |
|---|---|
| 来源 | 用户真机（Windows 版同一套配置正常，Android Release 全部失败） |
| 设备 | 复验仍在 `pixel_7_-_api_34`（x86_64）；真机为 arm64 |
| 包 | 复验用 Release `-Abi android-x64`（44.2 MB）；归档包换成新的 arm64 42.1 MB / AAB 39.7 MB |
| 崩溃 | 全程无 FATAL / ANR |

### 6.1 逐条对照

| # | 反馈 | 根因 | 修复位置 | 本次验证 |
|---|---|---|---|---|
| 1 | **无法在线查询，配置的 API 测试也「连接失败」（Windows 版同一配置可用）** | `AndroidManifest.xml` 缺 `INTERNET`。Debug 由工具链为调试器注入，Release 不带 → 只在装机后、只在 Release 上失败，socket 被拒，.NET 只报 `Connection failure` | `src/NutriTrack.Android/AndroidManifest.xml` 显式声明；新增 `NetworkErrors.Describe()` 把异常翻成人话 | ①`aapt2 dump permissions`：新包有 `android.permission.INTERNET`，旧归档包确认**没有**；②设备实测**三个源全部连通**（见 6.3）；③失败文案变为可读的「域名解析失败：…」 |
| 2 | 胶囊文字都不垂直居中 | `Button.chip` 模板漏绑 `VerticalContentAlignment`（`ContentPresenter` 默认 `Top`）；另有 20 处药丸靠 `Padding="9,3"` 凑 | `Styles/Controls.axaml` 模板补绑 + `MinHeight`；新增 `Border.badge`，各视图内联药丸换类 | 模拟器 420 dp 截图：筛选胶囊、GI 徽标、离线库药丸上下留白均为 1 px |
| 3 | 体重记录页文字遮挡 | 单行 `104,104,*,Auto` 把备注挤成几十像素宽 | `WeightView.axaml` 改两行，备注跨四列 | 模拟器 + 420 dp 截图：占位文字完整 |
| 4 | 详情页 GI / GL 比例不合理、标签歪斜 | 2 : 1 分栏在窄屏每栏只剩一百多 dp | `FoodDetailView.axaml` 改上下同构（圆形数值 + 右侧两行） | 模拟器截图：说明成段、标题与等级胶囊同行、`21.8` 不再压字 |
| 5 | 餐次修改是居中对话框，左紧右松 | 居中浮层在窄屏必然两侧留白不均 | `TrackerView.axaml` 改 `Border.sheet` 底部抽屉（拖拽条 / 滚动区 / 等宽双按钮），组合浮层同步 | 模拟器截图：通栏、拖拽条、两按钮各占一半 |

### 6.2 顺带修掉的

- **`CanLog` 无变更通知**（§三 那条低级问题）：`FoodDetailViewModel` 在 `_session.Changed` 与每次
  `LoadAsync` 收尾各补一次 `OnPropertyChanged(nameof(CanLog))`。
  模拟器复验：详情页下滑可见「加入今日餐次」绿色主按钮，点开后面板正常展开（份量 `− 50 g +`、
  「半碗 · 110 g / 一碗 · 220 g」、早餐 / 午餐 / 晚餐 / 加餐 四个按钮等宽一行）。
- 药丸垂直居中之外，`CustomDishes / ImportedFoods / Main / Profile / Settings` 里同问题的内联药丸一并换类。
- **把所有网络异常翻成中文**（`Infrastructure/Net/NetworkErrors.cs`）。第一版只按
  `HttpRequestException.HttpRequestError` 分档，在模拟器上一试才发现**Android 上这个属性恒为 `Unknown`**
  ——那台设备走的是 Java 的网络栈（`AndroidMessageHandler`），外层只剩一句 `Connection failure`，
  真正的原因在内层：

  ```
  [net] type=System.Net.Http.HttpRequestException error=Unknown msg=Connection failure
        inner=Java.IO.IOException:Unable to resolve host "api.nal.usda.gov": No address associated with hostname
  ```

  改成从外往里扫异常链，同时认 `HttpRequestError`、`SocketException.SocketErrorCode` 与
  Android 的英文原文（`Unable to resolve host` / `EPERM (Operation not permitted)` /
  `Network is unreachable` / `Connection refused` / `timeout`），认不出来的原样返回。
  实测提示从「请求失败：Connection failure」变成「域名解析失败：设备现在解析不了域名，
  先确认真的联网了（飞行模式、代理也可能造成这个）」，截图 `w97-online-error-text.png`。
  `EPERM` 那一档专门指向「Release 包必须声明 INTERNET」——正是这轮第 1 条的根因。
  这些原文都固化成了 `NetworkErrorsTests`（8 条），真机哪天换了异常类型，这里是第一个红的地方。
  设置页的「测试」不再套 `请求失败：` 前缀（否则变成「请求失败：域名解析失败：…」两层冒号）。

### 6.3 联网在设备上验过了

前几次启动 AVD 的 `wlan0` 一直是 `NO-CARRIER`、`eth0` 没地址（`Active default network: none`），
所以一度断定「模拟器无网、只能上真机验」。这次用 `-no-snapshot-load` 冷启动后 `eth0` 拿到了
`10.0.2.15`，DNS 也能解析——**是之前那几次启动的 DHCP 没起来，不是模拟器不能联网**。
（`ping 8.8.8.8` 仍然 100% 丢包，那是 QEMU 的 slirp 不转发 ICMP，跟 TCP 无关，别拿 ping 当判据。）

装 Release x64 包实测（与 arm64 归档包同源码同配置）：

| 场景 | 结果 | 截图 |
|---|---|---|
| 搜 `apple` | 「在线 11 条 · 3/3 个来源」；USDA 6 条、Open Food Facts 5 条、中国营养学会「无结果」（英文词查中文库，属正常空结果，不是失败）。重复搜同一词会标「缓存」（结果在本地留 7 天） | `w97-search-en.png` |
| 搜「苹果」 | 「离线库 56 条 · 在线 9 条」；中国营养学会 6 条、Open Food Facts 3 条、USDA「无结果」；结果卡带「英文来源」标记与 GI | `w97-search-cn2.png` |
| 设置 → 在线数据源 → USDA「测试」 | 「连通正常：搜「apple」返回 2 条，耗时 1609 ms」 | `w97-test-usda.png` |
| 设置 → 在线数据源 → Open Food Facts「测试」 | 「连通正常：搜「苹果」返回 2 条，耗时 790 ms」 | `w97-test-off3.png` |

到此第 1 条闭环：**补 `INTERNET` 是根因修复，三个源在设备上真的能出数据**。
仍然未验的只剩需要 Key 的视觉模型识别（本机没有可用 Key）。

### 6.4 一条方法上的教训

用 `NUTRITRACK_SHOT_SCROLL=bottom` 截图核对「某个区块在不在」是错的：滚到底只保留页尾一段，
页面中部的改动（比如新加的按钮）在截图上根本不会出现，看起来就像「渲染丢了」。
本次为此白排查了一轮。要验某个区块，用 `NUTRITRACK_SHOT_CLICK=<按钮文案>` + `SCROLL=click`
把它滚进视野；区块在不在也可以直接看日志里的 `[click] … Command=ok`——找不到按钮会打印「未找到」。


---

## 七、第三轮：提醒链路复验（2026-11-01，模拟器 x86_64 Release）

首轮把 #7 / #12 记成「未验」，原因是提醒只在到点那一刻判定——要么等到 19:00，要么改系统时间，
两样在模拟器上都麻烦。这次换个做法：**把提醒时间改到 5 分钟后**，把应用压到后台，等它自己到点。

操作路径：设置 → 记录提醒 → 勾「启用」→「餐食记录」→ 点时间框，在小时/分钟两列里滚到 `12:40`
（列表按滑动位置选中，选完点 ✓）→ 按 Home 键把应用切到后台。

| 时刻 | 发生了什么 | 证据 |
|---|---|---|
| 12:37 | 启用提醒；`dumpsys alarm` 出现两条 `RTC_WAKEUP com.nutritrack.app.action.ALARM`（今天 12:40、明天 07:30）。顺带点「发一条测试通知」也能收到（`id=4002`，渠道 `nutritrack.info`） | `w97-reminder-on.png` |
| 12:42 | 通知到达（`id=4001`，渠道 `nutritrack.reminder`，`actions=3`）：标题「今天的饭还没记」，正文「记一条就能看到今天还差多少蛋白质、膳食纤维和钠。当前设定 12:40 提醒。」 | `w97-notify-reminder.png`、`w97-notify-actions.png` |
| 12:45 | 点「10 分钟后」：通知立刻消失，`id=4001` 计数归零；闹钟从 12:45:57 重排到 **12:55:54**（正好 +10 min），体重那条仍是明天 07:30 | `dumpsys alarm` |
| 12:55 | 到点再次投递同一条通知（`id=4001` 重新出现） | `dumpsys notification` |
| 12:57 | 点通知主体：`topResumedActivity` 为 `MainActivity`，应用内浮出提醒卡片；点卡片「去记录」落到「记录」页（今天 0 / 2155 kcal、7 天柱图正常） | `w97-notify-click.png`、`w97-card-goto.png` |

结论：#7 与 #12 **转通过**。提醒是「到点且确实没记才发」——那天已经记过一餐就不会响，
这点在首轮没体现出差异，复验时特意用了空白的一天。

---

## 八、第四轮：卸载重装后由备份恢复（2026-11-01，模拟器 x86_64 Release）

首轮 #5 只验了「导出 → 同机导入」，没验真场景：**卸载会连应用私有目录一起删掉**，
而用户库是 AES-GCM 加密的（密钥文件也在私有目录）。所以真正要回答的是：
删库删钥匙之后，还能不能从备份包里把数据还原出来。

做法：全新安装 → 造四类数据 → 导出 → `adb uninstall` → 重装 → SAF 选备份 → 覆盖还原。

| 步骤 | 结果 | 证据 |
|---|---|---|
| 造数据 | 新建档案「BackupA（男 · 30 岁 · 170 cm）」；收藏「苹果(均值)」；午餐「苹果(均值) 150 g」；体重「68.5 kg / 腰围 82 cm / 备注 morning」 | `w98-restore-profile.png` 等见下 |
| 导出 | SAF 默认名 `NutriTrack备份-20261101-1328.zip`，落 `/sdcard/Download`，**1845 B** | `w98-backup-save.png` |
| 包内容 | 明文 zip，8 个 JSON：`manifest / users / favorites / meals / templates / weights / foods / settings`。`manifest.counts` = 1 档案 / 1 收藏 / 1 记录 / 1 体重；`includesSecrets=false`（没配 Key） | 见下方「包结构」 |
| 卸载重装 | `adb uninstall` + `adb install`，启动后「尚未创建档案 / 还没有档案，先建一份吧」，库确实清空 | — |
| 导入 | 预览「含 1 个档案 / 1 条收藏 / 1 条记录 / 1 条体重记录」「当前库已有 0 个档案、0 条记录，与备份没有冲突」；选**覆盖**： 「同名档案按备份覆盖字段，并先清空它的收藏与记录再写入」 | `w98-restore-done.png` |
| 结果 | 「导入完成 新增档案 1 个，收藏 +1，记录 +1，体重/腰围 +1。」 | `w98-restore-done.png` |
| 逐项核对 | 「我的」：BackupA（男 · 30 岁）170 cm / 69 kg，日能量目标 2203 kcal（体重回填 68.5 后重算，符合预期）；记录页：BackupA 的每日摄入 78 kcal，午餐「苹果(均值) 150 g」，7 日柱图 11/1=78；体重页：当前 68.5 kg、BMI 23.7、腰围 82 cm，条目时间 `11-01 13:20` 且备注 `morning` 在；收藏页：BackupA 的收藏 · 共 1 条，苹果(均值) | `w98-restore-profile.png`、`w98-restore-meal.png`、`w98-restore-weight.png`、`w98-restore-fav.png` |

**包结构（为什么删库也能还原）**：备份不是 `user.db` 的拷贝，而是把各表序列化成 JSON 再打包。
好处是不受加密密钥影响（卸载后重新生成的密钥也能写进去），代价是包体随数据量增长而不是固定大小——
首轮 1 条记录时 1797 B，本次 4 条数据 1845 B，差 48 B，量级由 `manifest.json`（411 B）和
每条约 130 B 的 JSON 记录决定，压缩后都很小。

结论：#5 **转通过**。仍未验的只剩需要 Key 的 #6 / #13 / #14（拍照与相册识别）与 #3 / #4（病症建议、
报告分享）的完整链路；#15 分享面板仍待复验。

---

## 九、第五轮：功能缺口复查（2026-11-02，模拟器 x86_64 Release）

用户确认联网问题已修复后，要求再查一遍「还有哪些功能缺口」。这一轮不验新功能，
只把首轮记成**未测 5 / 未确认 1** 的六条逐条重新判定，然后沿着 18 条清单往两边各扩一步：
每个入口点进去，看有没有「按钮在、点下去没反应」「页面在、数据永远为空」的地方。
六条里有三条能就地判定，其中一条（#4）暴露出一处真实缺口（见 §9.3 缺口 B）。

### 9.1 三条从「未测 / 未确认」转正

| # | 本轮做法 | 结果 | 证据 |
|---|---|---|---|
| 3 | 档案勾上「糖尿病 + 高血压」→ 回建议页 | **通过**。档案卡下方多出「糖尿病 · 高血压」一行；「今天怎么吃」里出现橙标「注意 糖尿病：先吃菜，再吃饭」，正文说明进餐顺序对餐后血糖的影响与主食选择（杂粮饭 / 荞麦面、避开粥与糊化度高的做法）。原本按普通成年人给的三条原则自动让位 | `w99-advice-conditions.png` |
| 15 | 记录页点「分享图片」「分享文字」 | **通过**。`topResumedActivity=com.android.intentresolver/.ChooserActivity`，两张面板分别是「Sharing image」（Quick Share / Messages / Photos / QRKeeper / Print）与「Sharing text」（预览「# 2026-11-01 周日 饮食记录 / BackupA · 已摄入 78 kcal / 2203 kcal (4%) …」，目标 Chrome / Drive / Bluetooth / Messages）。**未验**：真正选一个接收方发出去（模拟器里没有登录态的微信） | `w99-share-image.png`、`w99-share-text.png` |
| 4 | 建议页顶栏「导出」→ SAF 存盘 →「分享」→「打开」 | **部分通过**。导出：提示「已导出「…」。用浏览器打开它即可打印，或「另存为 PDF」发给医生」；分享：Chooser 收到 `NutriTrack建议-20261102-0902.html`（Drive / Bluetooth / Gmail / Quick Share）。打开：模拟器里没有能处理 `text/html` 的应用。**未验**：真机上 Chrome 接管。原提示是一句干巴巴的「没有可以打开网页的应用。」，改成「设备上没有能打开网页的应用。可以先用「导出」存成文件，再从文件管理器打开」（`AdviceViewModel.OpenReport`），设备上已复验新文案 | `w99-adv-export.png`、`w99-open-no-browser.png` |

三条之外，#1 的中文搜索、#8 的飞行模式、#16/#17/#18 的包与签名本轮未复跑——它们与本次改动无交集。

### 9.2 复查中修掉的五件事

| 现象 | 根因 | 修法 |
|---|---|---|
| 「数据完整性检查」把「菜品配料表」标成**注意**，汇总写「1 项需要注意」 | 精简库本来就不含配料，检查器把空表一律当降级 | `DatabaseHealthService` 读 `meta.scope`：`core` 时该行显示「本版是精简成分库，不含菜谱配料（官方形态，不是文件损坏）」+ 绿色「正常」。补两条测试（精简库空表 → 正常；完整库空表 → 仍提示） |
| 「新建菜品」的菜品名、收藏分类名看起来像一行灰字，不像输入框 | `TextBox.flat` 的 `Background=Transparent`、`BorderThickness=0`，形状全靠外层 | 两处都包一层 `Border.cardFlat`（`CustomDishesView`、`FavoritesView`），配料的克数框同样处理 |
| 用户手册写了「建议页有推荐食谱」，实机根本没有 | 手册照桌面开发版描述；`health.db` 从未进包（§9.3 缺口 B） | 手册该节按实机重写：删掉「推荐食谱」，补「能量与营养素完成度」「针对你的建议」「这些数字是怎么来的」与顶栏四个图标的行为 |
| 「打开」失败后只报「没有可以打开网页的应用」，没有下一步 | 文案停在陈述现象 | 补一句「可以先用「导出」存成文件，再从文件管理器打开」 |
| **抽屉/面板开着时按返回键，整个应用退到桌面** | `IBackHandler` 这套「页面先吃掉返回键」的接口定义在 `Services/IBackHandler.cs`、`MainViewModel.HandleBack` 也按它分派，但**一个页面都没实现**——`HandleBack` 永远落到「退栈」，退不动就交回平台（Android 宿主 `MoveTaskToBack`） | `TrackerViewModel`（编辑抽屉 / 组合抽屉）、`FoodDetailViewModel`（加入餐次面板）、`AdviceViewModel`（饮食原则浮层）实现 `HandleBack()`：有临时层就先关掉并返回 true，否则 false 交回原逻辑 |

### 9.3 当时的两处缺口（已于 §10 关闭）

#### 缺口 A：菜谱菜（家常菜）整库没进包

`food_core.db` 共 7,648 条，其中 `group_key='dish'` 只有 **229 条**，且几乎全是港式点心类
（占酱多、油炸鬼、菠萝包……）。真正的家常菜一条都没有：

| 搜索词 | 精简库命中 | 完整库（`food_full.db`）命中 |
|---|---|---|
| 红烧肉 | 0 | 296 |
| 番茄炒蛋 | 0 | 1 |
| 宫保鸡丁 | 0 | 1 |
| 可乐鸡翅 | 0 | 27 |
| 鱼香肉丝 | 0 | 18 |
| 蛋炒饭 | 0 | 11 |

连带影响：`dish_ingredient` 表 **0 行**，所以详情页那块「配料组成」（用了哪些食材、各多少克）
在离线库里对所有食物都不会出现。用户搜「红烧肉」只能靠在线源，飞行模式下搜不到。

原因是 `tools/fooddb/build_food_db.py:759` 的一行取舍（注释原话：「菜品会把它撑大三倍」）。
实测各方案的体量（deflate 口径，约等于 APK 内的增量）：

| 方案 | 食物条数 | 压缩后 | 相对现在 |
|---|---|---|---|
| 现在（精简库，无菜谱配料） | 7,648 | 2.40 MB | — |
| 常用 3,000 道菜 + 仅概览列 | ~10,600 | 3.60 MB | **+1.2 MB** |
| 常用 3,000 道菜 + 完整营养 | ~10,600 | 4.80 MB | +2.4 MB |
| 全部 2 万道菜 + 仅概览列 | ~27,000 | 9.10 MB | +6.7 MB |
| 全部 2 万道菜 + 完整营养 | ~27,000 | 17.20 MB | +14.8 MB |

（「概览列」= 能量 / 三大营养素 / 钠，够列表与记录用；点进详情时其余营养素显示为「—」。）

#### 缺口 B：食谱库 `health.db` 从未部署到任何平台

- 代码侧齐全：`RecipeDatabaseLocator`、`RecipeMatcher`、建议页的「推荐食谱」卡片、25 条食谱 + 112 条配料的库都在。
- 部署侧缺失：没有任何 `.csproj` / 打包脚本把它带进安装包；Android 只有一个
  `FoodDatabaseInstaller`（只解压 `food_core.db`）。仓库根的 `data/health.db` 是 **0 字节**，
  真库在 `data/GI/health.db`（1.45 MB）。
- 结果：桌面开发时靠「向上找 `data/GI`」的兜底能看见推荐食谱，发布出去的程序和 Android 版都看不见。
  这也解释了 §9.2 里手册与实际不符那条。
- 体量：1.45 MB → deflate **0.30 MB**，是这批选项里最便宜的。
- 修法：Android 加 `AndroidAsset` + 把 installer 泛化成「装两个库」；桌面发布脚本带上 `data/health.db`。

### 9.4 18 条最终状态

| 状态 | 条数 | 条目 |
|---|---|---|
| 通过 | 13 | 2、3、5、7、8、9、10、11、12、15、16、17、18 |
| 部分通过 | 2 | 1（首屏耗时只有模拟器数据）、4（打开报告待真机浏览器） |
| 未测 | 3 | 6、13、14（都要视觉模型 Key：拍照 / 相册 / 候选采纳） |

已无阻断性问题；剩下的三处都不是「坏了」，而是「还没验」或「从来没带数据」。

### 9.5 返回键复验（修完当场验的）

| 场景 | 按返回键之前 | 按之后 | 证据 |
|---|---|---|---|
| 记录页 → 某一餐「＋」→ 组合抽屉 | 抽屉贴着底栏展开（「这一餐还没有条目。先记两样，再回来把它存成组合。」） | 抽屉收起，仍停在记录页；`topResumedActivity` 还是 `MainActivity` | 修之前同一操作直接落到 `NexusLauncherActivity`（桌面）；修后 `w99-sheet-open.png` / `w99-sheet-back.png` |
| 详情页 →「加入今日餐次」→ 份量面板 | 面板展开（`− 200 g +`、`一个 · 200 g / 半个 · 100 g`、四餐等宽） | 面板收起，详情页保持原位（绿色的「加入今日餐次」还在） | `w99-detail-back.png` |

顺带记一条排查教训：`uiautomator dump` 只报**当前可见**的节点，用 `has("加入今日餐次")` 在页面底部查会得到 `NO`，
一度让我以为详情页的主按钮丢了。正确的是按固定步长滚动、每一步都查一次——这个按钮在页面中部，滚过去就没了。

### 9.6 本轮的证据与产物

截图归档 `artifacts/smoke/`：`w99-advice-conditions.png`、`w99-share-image.png`、
`w99-share-text.png`、`w99-adv-export.png`、`w99-open-no-browser.png`、`w99-dish-new-input.png`、
`w99-db-check.png`、`w99-sheet-open.png`、`w99-sheet-back.png`、`w99-detail-back.png`。

## 十、第六轮：把菜谱与食谱补进包（2026-11-03）

§9.3 那两处「要用户决策」的缺口，用户选了补：**菜谱菜取常用 3000 道 + 仅概览列**、
**health.db 部署到 Android 和桌面**。这一轮把两件事都做完，并在模拟器上验到界面。

### 10.1 菜谱菜：`food_core.db` 从 7,648 条涨到 10,648 条

改动 `tools/fooddb/dish_build.py` + `build_food_db.py`：`build_dish_rows` 新增
`max_dishes` / `nutrient_keys` 两个参数，core 传 `CORE_DISH_BUDGET = 3000` 与
`{energy_kcal, protein_g, fat_g, carb_g, sodium_mg}`，full 传 `MAX_DISHES = 20000` 与「全都要」。
之前 core 是整段跳过菜品阶段（注释原话「菜品会把它撑大三倍」），所以 `dish_ingredient` 是空表。

| 指标 | 改前 | 改后 |
|---|---|---|
| 食物条数 | 7,648 | **10,648** |
| `group_key='dish'` | 229（全是港式熟食） | **3,229** |
| `dish_ingredient` 行数 | 0 | 有值，详情页「配料组成」出得来 |
| 库文件 | 9.8 MB / deflate 2.40 MB | 14.2 MB / deflate **3.68 MB**（APK +1.28 MB） |

离线搜索实测（飞行模式下，三个在线源全失败）：

| 搜索词 | 改前离线命中 | 改后离线命中 |
|---|---|---|
| 红烧肉 | 0 | **13** |
| 宫保鸡丁 | 0 | 有（同族多条） |

详情页：红烧肉 →「配料组成（15 项）· 合计约 991 g · 14 项可查看成分」，
五花肉 500 g / 冰糖 50 g / 生抽 17.2 g / 老抽 17.2 g / 姜片 30 g / 桂皮 9 g……

### 10.2 食谱库：`health.db` 进包并落到 `files/data/`

- 新增 `tools/fooddb/build_recipe_db.py`：从 `data/GI/health.db`（1.45 MB，整份应用库）
  只抽 `recipes`(25) + `recipe_ingredients`(112) 两张表，产出 `data/generated/health.db`（**12 KB**）。
  只读代码（`SqliteRecipeRepository`）就查这两张表，其余表跟食谱无关。
- `NutriTrack.Android.csproj` 加 `AndroidAsset`；`FoodDatabaseInstaller` 泛化成
  `Install(asset, revision, stamp)`，新增 `EnsureRecipeInstalled`（自己的版本戳 `health.version`）。
- 桌面发布器默认内容补一条 `data/GI/health.db → data/health.db`，`docs/10` 的打包命令同步。
- 顺手删掉仓库根那个 0 字节的 `data/health.db`——它正是「发布出去的程序看不见推荐食谱」的原因。

`health.db` 只占 12 KB，deflate 后 0.00 MB——体积上等于免费。

实测（模拟器，档案勾了糖尿病 + 高血压）：建议页出现「推荐食谱」卡片，
标题「从 25 条食谱里按你的档案挑出 4 条；能量与三大营养素按「一份」计。」，
条目「凉拌黄瓜 / 小米南瓜粥 / 凉拌木耳 …」，每条带「适合：低 GI · 高血压」标签、
一份的能量与三大营养素、配料清单和「加入记录」按钮。

### 10.3 顺带修掉的两个布局缺陷

| 现象 | 根因 | 修法 |
|---|---|---|
| 配料组成在手机上每行只排一项，右半边整片空白 | `WrapPanel ItemWidth="168"` 不含子项外边距，一项实际占 176；411 dp 机器上卡片内宽约 346，2×176=352 差 6 dp 挤不下，整块退化成单列 | `ItemWidth` 改 150（2×158=316），并在注释里写清这个 6 dp 的来龙去脉 |
| 食谱名压到右边「适合：…」标签上（「小米南瓜粥」最明显） | 名字放在横向 `StackPanel` 里，被按内容无限宽测量，`*` 列约束不到它 | 名字改到内层 Grid 的星号列 + `TextTrimming`；随后发现行内塞不下「名字 + 标签 + 加入记录 + 复制」，名字被挤成 0 宽，于是把标签移到下一行 |

### 10.4 一条排查教训：软键盘会把底栏顶上来

这轮有半小时耗在「点搜索结果没反应、直接跳到记录页」上。真相是：**键盘弹起后，
应用窗口变矮，底部导航条被顶到 y≈1430**，而我一直在按键盘收起时量到的坐标
`input tap 540 1430` 点卡片——正好点中「记录」标签。

正确的做法：先 `KEYCODE_BACK` 或点 IME 的完成键收键盘，再 `uiautomator dump` 取 `bounds`。
两次 dump 的坐标不能混用，尤其是屏幕下半部分的控件。

### 10.5 证据

`artifacts/smoke/`：`w99-search-hsr.png`（离线 13 条）、`w99-dish-detail-android.png`、
`w99-dish-ingredients-android.png`（两列配料）、`w99-advice-recipes-android.png`（推荐食谱）、
`w99-advice-recipes.png`（桌面同一份 `data/generated/health.db` 渲染）、`w99-dish-desktop.png`（宫保鸡丁 20 项配料）。

## 十一、第七轮：更新检测与自动更新（2026-11-04）

### 11.1 这一轮改了什么

| 项 | 内容 |
| --- | --- |
| 清单 | 读仓库根的 `update.json`，兼容扁平的 `Version_Win7` / `Version_Win10`，新增 `Assets`（含 SHA-256、大小、Android 条目） |
| 下载 | 下到 `<FilesDir>/updates`，边下边算 SHA-256，对不上直接删掉 |
| 装机 | Windows：`setup.exe /silent /dir "<当前 exe 目录>" /restart`；Android：FileProvider + 系统安装器 |
| 权限 | 新增 `REQUEST_INSTALL_PACKAGES`；`file_paths.xml` 加 `files/updates` |
| 界面 | 设置页「软件更新」卡片 + 主界面右上角浮卡 + 启动静默检查（可关） |
| 发布 | 发布器支持 Android（`--family android --format apk|aab|both`），`tools/make-update-json.ps1` 生成清单 |

### 11.2 验证结果（模拟器 x86_64 Release）

| 项 | 结果 |
| --- | --- |
| 装包 | `adb install` 通过（需先卸载调试签名版本，签名不同会 `INSTALL_FAILED_UPDATE_INCOMPATIBLE`） |
| 权限 | `aapt2 dump permissions` 含 `REQUEST_INSTALL_PACKAGES` |
| 设置页卡片 | 正常渲染：当前版本 v0.2.0、检查按钮、自动检查开关、免安装提示按条件显隐 |
| 自动检查 | **这台模拟器没有网络**（`ip addr` 只有 lo，`ping` 报 Network is unreachable），卡片如实显示「检查更新失败：域名解析失败…」——错误文案与降级行为正确，但**没能在这台设备上验证「检出→下载→拉起安装器」的完整链路** |
| 待验 | 真机（有网）点一次「检查更新」，确认出现「发现新版本」并能拉起系统安装器 |

### 11.3 桌面侧验证（同一套代码）

| 项 | 结果 |
| --- | --- |
| 检出 | 指向本地桩清单（`NUTRITRACK_UPDATE_URL`），设置页与浮卡同时显示「发现新版本 v0.3.0」 |
| 免安装运行 | 按钮为「前往下载」+ 灰色提示「不是由安装包安装的」 |
| 经安装包装过 | 按钮为「下载并更新」（依据是安装目录里的 `install.json`） |
| 静默安装 | `setup.exe /silent /dir <目标>` 退出码 0，`install.json` 版本 0.2.0 |
| 装完自启 | 追加 `/restart` 后进程从目标目录起来，确认「装完回来」这条链路成立 |
| 哈希校验 | 单测覆盖：正确哈希通过、错误哈希抛 `UpdateIntegrityException` 且不留残文件 |

### 11.4 证据

`artifacts/smoke/w100-update-android.png`（手机设置页卡片与失败文案）、
`w100-update-desktop-installed.png`（已安装副本，按钮为「下载并更新」）、
`w100-update-desktop-portable.png`（免安装运行，按钮为「前往下载」）。


## 十二、第八轮：0.3.0 发布与提示位置调整（2026-11-04）

### 12.1 这一轮改了什么

| 项 | 内容 |
| --- | --- |
| 文件名 | 产物名只留显示名里的英文部分（中文不进文件名）；显示名（开始菜单、卸载列表）保持原样 |
| Android 打包 | 发布器一次可出多个 ABI（`--arch arm64,x64`，界面里是「全部 ABI」）；归档时打印 `versionCode` 与可直接粘进 `update.json` 的条目 |
| 提示位置 | 桌面端的提醒卡与更新卡改由独立窗口显示，贴在**屏幕**工作区右下角；Android 与截图自检仍用主界面内浮卡 |
| 设置页 | 「软件更新」从最下面移到「在线数据源」与「备份与导出」之间 |
| 版本号 | 0.2.0 → 0.3.0（Android `versionCode` 200 → 300） |

### 12.2 提示浮窗位置（Windows，1920×1200，任务栏 48 px）

| 项 | 实测 |
| --- | --- |
| 窗口矩形 | `(1428, 1043) 484×101`，右边缘 `1912 = 1920 − 8`，下边缘 `1144 = 1152 − 8` |
| 样式 | `WS_EX_TOPMOST` + `WS_EX_TOOLWINDOW`，不进任务栏、不抢焦点 |
| 多屏 | 跟随主窗口所在的那块屏（`Screens.ScreenFromWindow`），取不到时退回主屏 |
| 主窗口内 | 内嵌浮卡已隐藏，不会出现两份 |
| 兜底 | 独立窗口建不起来时自动退回内嵌，并往诊断日志写一条——宁可丑，不能「什么都没发生」 |

证据：`artifacts/smoke/w101-toast-bottom-right.png`（浮卡贴在任务栏上方，按钮为「前往下载」，因为这一份是免安装运行）。

### 12.3 0.3.0 产物

| 产物 | 体积 | SHA-256 |
| --- | --- | --- |
| `NutriTrack-0.3.0-windows10-x64-setup.exe` | 145.4 MB | `41a8df0f9a974438bd847174368b25cfb3ea1f4441263b82f68c9982e515f9b6` |
| `NutriTrack-0.3.0-windows7-x64-setup.exe` | 145.4 MB | `f2eefe38882c8404b1d28c0ba6bb696ccfff96e8c95dec3c3fa0bccfe311ae6d` |
| `NutriTrack-0.3.0-android-arm64.apk` | 43.5 MB | `e7b8eb8768259f8aa2e9650770c1941bca1e9cf6da3c05192d585507a7142062` |

**Android x64（x86_64）不打进清单**：它只服务于模拟器，真机装不上，多一条记录只会让清单看起来像发了三个包。需要时用发布器单独出一条即可。

三个包的显示版本一致（0.3.0），Android 的 `versionCode` 为 300。清单由 `tools/make-update-json.ps1` 生成，哈希与上表一致。

**文件名必须是纯 ASCII**：第一次用了「知食 NutriTrack」，GitHub 把资产名截成了 `_NutriTrack-…`（中文前缀被丢掉），清单里的链接于是指向一个不存在的文件。当时靠手工 `--name NutriTrack` 绕过去；现在发布器自己做这件事——文件名只取显示名里的 ASCII 部分，中文只留在快捷方式与卸载列表里（见 `docs/10` §三）。

### 12.4 遗留

- Android 的「检出 → 下载 → 拉起系统安装器」仍然只有模拟器上的失败路径验证（那台 AVD 没有网络），真机复验未做。
- Windows 安装包 145 MB，本机网络上传 GitHub 时反复在 40–50 MB 处断开（`408 Upload body timed out` / `Connection was reset`），两个安装包改由维护者手动上传到 Release。APK 与 `update.json` 已由脚本传完。

## 十三、第九轮：发布流程规范化（2026-11-04）

### 13.1 改了什么

| 项 | 内容 |
| --- | --- |
| 目标多选 | 把「平台 / 系统 / 架构 / 产物类型」收成一张 8 项的清单（`ReleaseTargets`），界面里勾选，命令行用 `--target`；默认勾三种真机形态 |
| 一条流水线 | `ReleasePipeline` 串起打包 → 汇总 → 写清单 →（可选）上传，界面、命令行、一键脚本共用同一条 |
| 清单改由 C# 生成 | 流水线手里有真实来源，不再从文件名反推；`tools/make-update-json.ps1` 转调 `--manifest-only` |
| GitHub 上传 | 自建 REST 客户端（不用 `gh release upload`，它不报进度）；token 取 `GITHUB_TOKEN` 或 `gh auth token`；同名资产先删后传；清单提交回 main |
| 一键脚本 | 界面按钮把当前勾选写成 `release-<版本>.ps1`，用 `Start-Process -Wait` 调发布器（WinExe 不等待，`&` 会立刻「完成」） |
| 默认显示名 | 没填应用名时去掉宿主后缀：`NutriTrack.Desktop` → `NutriTrack`，电脑包与手机包文件名前缀一致 |

### 13.2 验证

| 项 | 结果 |
| --- | --- |
| 单元测试 | 671 通过（新增目标目录、定义校验、清单生成、脚本转义、上传请求序列共 22 条） |
| 界面 | `artifacts/smoke/publisher-release.png`：左栏四个分组、目标清单默认勾选、底栏三个按钮都在 |
| 脚本端到端 | 生成 `release4.ps1` → 运行 → 产出 `NutriTrack-0.3.0-windows10-x64-setup.exe`（145.4 MB）与 `update.json`（1 条、哈希与包一致） |
| 上传 | 只有单测覆盖请求序列（建 Release → 传资产 → 提交清单）与进度回调；真实网络上传未跑（145 MB，交由维护者） |

### 13.3 遗留

- 上传进度的**观感**没有在真机上验证过：单测保证了回调会走到 100%，但网络抖动时的表现要等一次真实上传。
- 一键脚本没有模拟「失败退出码」：`Start-Process` 的 `ExitCode` 判过了，但没造一个必失败的场景。
