# PiliPlus BTR（安卓 APK）

> 本仓库是 [aierlma/PiliPlus](https://github.com/aierlma/PiliPlus) 的 fork，**只多一条自己的安卓 APK 流水线**：
> 上游那套「定期合并 BTR 作者与官方 main、再发未签名 IPA」的自动化在 aierlma 的仓库里运行，在这里不运行，也不受本 fork 影响。
> 上游的 iOS 包仍由 aierlma 发布：[releases/latest](https://github.com/aierlma/PiliPlus/releases/latest)。

## 安卓版（本仓库的产物）

- **下载**：[Releases](https://github.com/sodaa-l/PiliPlus/releases) 的附件。每次产出 3 个按 ABI 拆分的 APK
  （`arm64-v8a` / `armeabi-v7a` / `x86_64`），手机一般装 `arm64-v8a`；每个包的 SHA-256、大小与签名证书指纹
  记录在同一次 release 的 `PiliPlus-BTR-android-build-info.json` 里。
- **正式版与预览版**：只有上游 `pubspec.yaml` 里的版本号变了（如 `2.1.6` → `2.1.7`）时发的才是**正式 release**；
  同一个版本号下的后续跟随构建都标为 **Pre-release**（滚动预览版）。所以 [releases/latest](https://github.com/sodaa-l/PiliPlus/releases/latest)
  与 Releases 页的「Latest」标记只在版本号变更时前进，可能比最新一次构建旧；想装最新的滚动构建就进
  [Releases](https://github.com/sodaa-l/PiliPlus/releases) 挑时间最新的那个（两类包签名相同、构建号递增，可直接覆盖安装）。
- **包标识**：`com.example.piliplus.btr`，应用名「PiliPlus BTR」，可与官方 PiliPlus 共存安装。
- **内容**：与同期 aierlma 的 iOS 包构建自**同一个来源提交**，含本地 HTTP 代理、多 Range 并发、CDN 选择/竞速等 BTR 功能。
- **构建**：`.github/workflows/android-release.yml`。每天 UTC 12:47（北京 20:47）跟随 aierlma 的 `btr` 最新提交构建，
  也可在 Actions 页手动 `Run workflow`。构建前会跑与上游相同的 BTR 测试作为**发布门禁**：不过就不发版，
  并自动开一个 issue 记录失败现场。
- **更新**：应用内的「检查更新」已经指向**本仓库**，但只认上面的**正式版** —— 只有上游版本号变化、发了新的正式
  release 时才提示，同版本号的滚动构建（Pre-release）不会打扰。弹窗里的下载按钮会用浏览器打开对应 ABI 的
  APK 直链（需要系统允许浏览器安装未知应用）。想跟最新的滚动构建，请 Watch 本仓库（Releases only）或手动
  查看 [Releases](https://github.com/sodaa-l/PiliPlus/releases)。
- **分支**：流水线文件放在默认分支 `apk-pipeline` 上，这样 `Sync fork` 碰不到它；`btr` 只用于跟随上游。
  ⚠️ 不要对 `btr` 使用 `Sync fork → Discard commits`（会强制重置该分支）；要同步就用 Merge，或者干脆不动它。

## 上游 aierlma 的说明

> 以下内容来自上游 README，描述的是 **aierlma 仓库**的 iOS 流水线，在本 fork 不运行，保留备查。

上游个人维护的 [PiliPlus](https://github.com/bggRGjQaUbCoE/PiliPlus) + BTR 分支，保留 [nishuodedui1145-del/PiliPlus](https://github.com/nishuodedui1145-del/PiliPlus) 的本地 HTTP 代理、多 Range 并发、CDN 选择/竞速、BTR 设置与日志，同时由 GitHub Actions 定期检查并合并 BTR 作者的 `btr` 与 PiliPlus 官方的 `main`。

上游发布[正式未签名 IPA](https://github.com/aierlma/PiliPlus/releases/latest)，通过 [AltGallery](https://github.com/aierlma/AltGallery) 更新。BTR 开关默认关闭，在「我的 → 设置 → 音视频设置」开启；播放页的「更多 → BTR」提供快设与日志。上游 iOS 版的应用内更新检查指向 aierlma 的仓库。

## 同步与发布

上游仓库的默认分支为 `btr`。`BTR and PiliPlus sync and iOS release` 在 GitHub 每天检查一次（cron `17 11 * * *`，纽约夏令时 07:17、冬令时 06:17），也可从 Actions 手动运行。GitHub 计划任务可能延迟；每次检查当前最新提交，不积压逐个版本构建。

1. Ubuntu 分别检查 BTR 作者 `nishuodedui1145-del/PiliPlus/btr`、PiliPlus 官方 `bggRGjQaUbCoE/PiliPlus/main` 和当前个人分支的 SHA，并与最近正式 IPA 对应的源码比较。两种更新都已纳入且应用输入没有变化时直接结束；个人分支的每一个维护提交无需另有 release。任何一个来源有新提交，都独立触发候选合并，无需等待另一方更新。
2. macOS 先普通合并 BTR 作者的新提交，再普通合并官方新提交。BTR 作者落后于官方时，只合并其新增历史，不把新版官方代码覆盖回旧版。任一来源冲突会回滚整个临时候选；个人 README 和 `.github/workflows/` 保留个人配置，两个来源的 README 分别保存为 `README-BTR-upstream.md` 和 `README-upstream.md`。
3. 检查 BTR 接线、独立 bundle ID 和个人更新地址；生成最新测试镜像，运行自包含 BTR 测试、完整 `flutter analyze`，再构建未签名 iOS IPA。
4. 解析实际 IPA，验证版本、构建号和最低系统要求；仅通过全部门槛后才快进 `btr`。发布脚本再比较最近正式版的应用输入：应用变化才发布；纯维护变化只同步历史，不上传或公开另一个 IPA。若运行期间分支已变化，停止推进，避免覆盖用户提交。

发布按应用输入去重，忽略根目录 `README*.md`、`docs/`、`test/`、`tool/`、`.github/`、`.vscode/` 以及 `.gitattributes`、`.gitignore`、`analysis_options.yaml` 的维护变化。Dart 代码、资源、平台文件、`lib/scripts/` 中的构建补丁、依赖锁文件、SDK/版本配置及未知路径仍触发完整发布门槛。构建号沿用 Git 历史提交计数，因此数字可能跳跃，并不表示中间发布了多个 App 版本。`retry_blocked` 只重试失败输入，不强制为相同应用内容再发一版。编译命令或打包行为若在 `.github/` 或 `tool/` 中修改，需要维护者同时调整应用构建输入。沿用现有 workflow，新的纯维护上游历史仍完整验证，但只同步、不发版；已同步的个人维护提交在计划阶段直接跳过构建。

例如 5485 到 5488 增加的是三个维护提交，应用代码、资源和依赖相同；按此规则不会再次发布功能相同的 IPA。已有正式附件保留，不回撤用户已安装的包。

个人正式 iOS 包使用 `官方主版本.官方次版本.个人构建号` 作为 `CFBundleShortVersionString`，例如基于官方 `2.1.6` 的 build 5502 显示为 `2.1.5502`。这符合 iOS 的三段数字版本格式，也让 SideStore 的正式版检查在官方未改 `2.1.6`、但实际源码有更新时识别新包。`CFBundleVersion` 仍为 Git 历史计数；官方原始版本保存在 IPA 的 `PiliPlusUpstreamVersion`、`build-info.json` 的 `upstream_version`、应用内版本说明与 Release 说明中，Tag/附件名继续沿用官方版本与个人构建号。版本写入仅在构建临时工作区发生，不修改上游 `pubspec.yaml` 中的版本策略。AltGallery 读取实际 IPA 的版本，不虚构订阅版本；相同包不会一直提示更新。维护变化仍由上面的应用输入规则排除。

发布前下载上一正式版的 `build-info.json`，要求新 IPA 的三段版本和构建号都严格递增；重复、回退或上游改版本策略导致新版本变低时，在任何 Release 写入和分支推进前阻断。普通快进历史、并发互斥及推进前再次核对远端 SHA 共同防止并行候选发布同一个版本。

在 SideStore 添加个人 BTR 源后，从该源安装或关联 PiliPlus BTR；刷新源后，已安装的旧版会在 My Apps 出现 Update，可直接下载、签名并覆盖更新，无需手动下载 IPA。Refresh 只续签，不能代替 Update。通知横幅还需要 SideStore 通知权限及 iOS 允许的后台检查；GitHub 和订阅不会向设备实时推送。

个人 iOS 版的应用内更新检查按构建号比较正式个人 IPA，并核对 AltGallery 独立 BTR 源中的版本、构建号和下载链接。只有订阅源已经收录的较新包才弹出更新提示；同一个包不会因为 GitHub Release 创建时间晚于编译时间重复提醒。GitHub 已发布而源尚未刷新时，自动检查保持安静，手动检查说明正在等待订阅同步。AltGallery 保留每 6 小时的全量生成，并由 `Sync watched release sources` 每小时第 23 分钟核对正式 `build-info.json` 与独立源和合集：未变化时不下载 IPA、不写入，落后时只更新 BTR 并合并已有条目。两个写入流程串行运行，旧 API 响应不能覆盖较新包；源必须核对实际 IPA，而不是猜测文件名版本。GitHub 可能延迟或丢弃计划任务，因此不承诺即时推送；维护者也可手动运行同一工作流验证收录。用户刷新现有源后在 My Apps 更新即可，无需卸载或手动下载 IPA。该检查不改应用标识、签名或用户数据，也不更改其他平台的更新逻辑。

并发上限为 1；一次检查最多进行一次构建，不自动重试相同的个人、官方、BTR 三个 SHA 输入。合并冲突、SDK 补丁失效、测试或构建失败时，GitHub Actions 标红并在本仓库记录一个带源 SHA 的 issue，保持上一正式 IPA。相同三个 SHA 的后续计划检查跳过，避免每天浪费构建；修复后提交新代码，或手动勾选 `retry_blocked` 重试。发生冲突仍需维护者处理，持续检查并不保证两个来源的任何变更都能无冲突合并。

发布流程会先创建 draft release，上传验证过的包，再推进分支并将 release 改为正式；部分失败留下可恢复的 draft，不替换旧正式版。Tag 以版本 + Git 历史构建号 + 源码 SHA 识别，附件名包含版本与构建号，`build-info.json` 记录个人源码、PiliPlus 官方和 BTR 作者三个 SHA、IPA 元数据与 SHA-256。生成过程不执行 IPA。

所有定时工作都在 GitHub，不依赖 Codex 会话、Mac 常驻进程或个人访问令牌。日常同步使用仓库内置的 `GITHUB_TOKEN`。保留自己的 CI 配置意味着两个来源的 workflow 改动需单独检查；源码更新仍走普通 Git 合并。此流程不调用 LLM；冲突被明确阻断，可在有 Copilot 权限时交给 Copilot 提议修复，再重新通过相同验证。

停止：在 Actions 禁用 `BTR and PiliPlus sync and iOS release`。日志位于该 workflow 的运行页，构建产物保留 14 天；失败 issue 是重复运行的阻断记录。GitHub 对长期没有仓库活动的公共计划任务可能暂停，恢复时重新启用该 workflow。

## 构建与测试

Flutter 版本由官方 `pubspec.yaml` 固定，SDK 与 UI 包需要 `lib/scripts/patch.ps1 iOS` 的补丁。标准 macOS GitHub runner 构建，不需要 Apple 签名凭据；产物需自行侧载。

本地构建个人正式 iOS 包时运行 `pwsh lib/scripts/build.ps1 btr-ios`，应用同样的 iOS 版本写入；其他平台沿用原版本生成方式。

本 fork 的安卓包在本地按同样方式复现（都是仓库内置脚本，无需改代码）：

```sh
pwsh lib/scripts/patch.ps1 android   # 打 Flutter SDK / material_ui 补丁，含 flutter pub get
pwsh lib/scripts/build.ps1 android   # 写版本号（安卓会拼上 commit 短哈希）并生成 pili_release.json
flutter build apk --release --split-per-abi \
  --dart-define-from-file=pili_release.json --android-project-arg btr=1 --no-pub
```

`--android-project-arg btr=1` 决定包名变体 `com.example.piliplus.btr` 与应用名「PiliPlus BTR」。

```sh
python3 -m unittest discover -s tool/tests -v
dart tool/tests/btr_update_test.dart
python3 tool/check_btr.py
python3 tool/make_btr_mirror.py
flutter test --no-pub test/standalone/btr_bitrate_unit_test.dart test/standalone/btr_cdn_racer_test.dart test/standalone/btr_round22_test.dart test/standalone/btr_round23_test.dart test/standalone/btr_round26_test.dart
```

这些测试自包含；需要真实 B 站临时媒体链接的 `btr_proxy_e2e_test.dart` 不在无人值守测试中，不能将自动验证等同于真机播放验收。

## 来源与许可

官方说明：[README-upstream.md](README-upstream.md)。BTR 作者说明快照：[README-BTR-upstream.md](README-BTR-upstream.md)。BTR 功能与设计：[README-BTR.md](README-BTR.md)、[docs/btr/DESIGN.md](docs/btr/DESIGN.md)。保留原始 [NOTICE](NOTICE) 署名与 [GPL-3.0 LICENSE](LICENSE)。此 fork 不代表 PiliPlus 官方或 Bilibili。
