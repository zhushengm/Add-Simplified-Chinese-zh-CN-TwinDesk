# TwinDesk 简体中文汉化 —— 改动清单与提交指引（中文配套）

> 这是给你自己看的配套说明。**发给原作者请直接用另一份英文文件**
> `TwinDesk-zh-CN-GitHub-Proposal.md`（已写成可直接粘贴的 GitHub Issue 正文）。

---

## 一、怎么用这两份文件

| 步骤 | 操作 |
|---|---|
| 1 | 打开 `https://github.com/InukaAsith/TwinDesk` → **Issues** → **New issue** |
| 2 | 标题填：`Add Simplified Chinese (zh-CN) localization for the desktop app UI (Windows + Linux)` |
| 3 | 正文整段粘贴 `TwinDesk-zh-CN-GitHub-Proposal.md` 的内容（从 Summary 开始即可） |
| 4 | 若作者回复要代码，把改好的文件打包发给他（见第五节） |

建议选 **Issue 而非 PR**：因为你手里是就地改过的安装副本、没有 git 历史，先提 Issue 询问作者想要
「直接合并」还是「抽出 i18n 层」，比直接甩一个没法自动合并的 PR 更容易被接受。

## 二、关键事实速查（避免临场答不上来）

| 项目 | 值 |
|---|---|
| 项目仓库 | `https://github.com/InukaAsith/TwinDesk` |
| 许可证 | GPL-3.0（与本项目一致，可直接贡献） |
| 版本 | **2.3.0**（`resources/app/package.json`） |
| 作者 | inukaasith；mesh 相关文件另署 Erdoğan Yeşil |
| 包名 | Windows：`macshare-win`；Linux：`twindesk` |
| 已随附语言包 | `locales/` 下 55 个 `.pak`，**已含 `zh-CN.pak` 与 `zh-TW.pak`** |
| Windows 端改动 | **48 个文件**（`windows/` + `shared/` + `windows/mesh/`） |
| Linux 端改动 | **89 个文件**（`linux/`）+ **14 个 `shared/`**（与 Windows 共用同一份译文） |
| 侧栏标签页 | Windows 27 个、Linux 26 个，**全部汉化（剩余英文 0）** |

> 很重要的一点，值得在 Issue 里强调：**软件本来就自带 `zh-CN.pak`**，Chromium 层（右键菜单、
> 文件对话框、开发者工具）早就是中文，缺的只是应用自身硬编码的英文文案 —— 这次补上的正是这块。

## 三、改动清单

**侧栏标签页（全部汉化）**

| 分组 | 文件 | 中文标题 |
|---|---|---|
| windows / linux | ui-overview / ui-share / ui-receive / ui-cursor / ui-keyboard / ui-clipboard / ui-notifs / ui-files / ui-mount / ui-terminal / ui-audio / ui-apps / ui-phone / ui-general / ui-about / ui-setup / ui-multi | 概览、共享屏幕、接收屏幕、光标、键盘、剪贴板、通知、文件、挂载、终端、音频、应用、手机、通用、关于、设置、多设备 |
| shared | ui-calls / ui-camera / ui-media / ui-messages / ui-pfiles / ui-remote / ui-hub / ui-nowplaying | 通话、摄像头、照片、信息、手机文件、远程控制、（手机中心）、（正在播放） |

**框架与主进程**

- `windows/ui-kit.js`、`linux/ui-kit.js` —— 共用控件库（方向卡片、状态行、光标锁定、`KIND` 设备名）
- `windows/app.js`、`linux/app.js` —— 主窗口外壳：侧栏对端状态、手机磁贴、**配对横幅全部阶段文案**
- `windows/main.js`、`linux/main.js` —— 托盘菜单、状态行、文件传输 HUD/Toast、防火墙自检、更新通知
- `windows/mesh/main.js`、`linux/mesh/main.js` —— Mesh 模式主进程（同上，另加 `lang=zh-CN` 开关）

**功能介绍与弹窗**

- `shared/feature-art.js` —— 16 项功能介绍文案 **+ 插画 SVG 内的可见文字**
  （`Connected to the other computer` → 已连接到另一台电脑，这类 SVG `<text>` 极易漏）
- `shared/mesh-intro.js` —— 「开启多设备？」弹窗全部文案与说明段落
- `shared/updates.js` —— 更新通知标题/正文与错误提示

**HTML 外壳（可见文本 + 窗口标题 + `lang="zh-CN"`）**

`app.html`、`monitor.html`、`appview.html`、`phoneview.html`、`sender.html`、`sound.html`、
`hud.html`、`camera-view.html`、`camera-send.html`、`callpop.html`（Windows 与 Linux 各一套）

**Mesh（多设备）模式**

`{windows,linux}/mesh/ui-{overview,share,cursor,multi,phone}.js`、`mesh/main.js`、`mesh/app.html`

**Linux 专属新增覆盖（Windows 版没有的部分）**

| 领域 | 代表文件 | 典型译文 |
|---|---|---|
| 桌面会话 | `ui-setup.js` | 未找到 X11 或 Wayland 显示。请从桌面环境启动 TwinDesk。 |
| 输入注入 | `ui-general.js` | 读取鼠标与键盘以实现跨屏；通过 uinput 注入 / 在 Wayland 上共享鼠标与键盘所必需 |
| 蓝牙 | `ui-general.js`、`a2dp.js`、`callaudio.js` | 与手机通过蓝牙 / BlueZ 无法识别该设备 / 此电脑的蓝牙已关闭 |
| 网络与防火墙 | `system.js`、`wifi.js`、`wake.js` | ufw 已允许 TwinDesk 通过 / firewalld 阻止了 TwinDesk。 / 未启用防火墙 |
| 屏幕共享 | `ui-share.js`、`ui-receive.js`、`pcscreen.js`、`vdd.js` | Wayland：每次开始共享时，桌面都会询问要共享哪个屏幕 / 已安装。已为"扩展"添加额外屏幕。 |
| 终端 | `terminal.js`、`sshd.js` | 未找到终端应用。请在任意终端中执行 ssh twindesk。 |
| 挂载/共享 | `phonedrive.js`、`smb.js` | GVFS 未挂载该设备 / 该文件夹的名称不能用于共享 |

## 四、三个需要你留意的点（Issue 里已替你写明）

1. **`shared/` 可能同时被 Mac 版使用** —— README 里写明 `shared/` = "Used by every app"，
   且 `windows/mesh/ui-phone.js` 文件头写着 *"Port of mac/ui-phone.js"`。若上游 Mac 版也用
   这些文件，直接合并会让 **Mac 版变成半中文**。这点我在本机无法验证，已在 Issue 中标为待作者确认。
2. **`plural()` 是我唯一改动的代码逻辑** —— 原为 `n + ' ' + w + (n===1?'':'s')`，中文无需复数，
   改为 `n + ' ' + w`。已告知作者可随时还原。
3. **暂无干净 diff** —— 就地改的安装副本、无 git，无法直接生成对上游 `HEAD` 的 `.patch`。
   作者若要代码，我可以把改动文件打包（ZIP）发过去，或按他指定的分支重新生成 diff。

## 五、Linux 版文件放在哪里（你问过的）

Linux 版同样是 Electron 未打包目录，真实应用根目录是：

```
<安装目录>/resources/app/
├── package.json          # main: "linux/main.js"
├── linux/                # ← 本次改的 89 个文件都在这里（含 mesh/ 子目录）
├── shared/               # ← 14 个文件，与 Windows 版共用同一份译文
├── linux-helper/
└── locales/              # 55 个 .pak，已含 zh-CN.pak
```

**放置方式**：把 `C:\Users\zhushengm\Downloads\TwinDesk\TwinDesk\resources\app\linux\` 整个目录
覆盖到目标机器安装目录下的 `resources/app/linux/`，`shared/` 同理覆盖即可。
（AppImage / deb / tar.gz 三种发行形式解压后目录结构一致；若 `.AppImage` 未解包，先
`--appimage-extract` 再替换、或改挂载方式写入。）

## 六、下一步我可以帮你做的

- [ ] 把改动文件分别打包成 Windows / Linux 两个 ZIP，方便你直接发给作者
- [ ] 生成「中文改动对照表」（英文原文 → 中文译文）供作者逐条核对
- [ ] 若作者选择 **B 方案（抽出 i18n）**，我可以把字符串抽成 `locales/zh-CN.json` 提交
- [ ] 顺带做繁体中文（`zh-TW`）—— 语言包里已经有 `zh-TW.pak`

> 提示：TwinDesk 是 GPL-3.0，贡献代码即默认采用同一许可证，无需额外授权文件；
> 但如果你希望署名，可在 Issue 末尾加一句你希望的署名方式（GitHub 用户名即可）。
