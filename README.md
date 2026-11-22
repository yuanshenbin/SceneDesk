<p align="center">
  <img src="resources/icon.png" width="96" alt="SceneDesk Logo" />
</p>

<h1 align="center">SceneDesk</h1>

<p align="center">短剧与短视频创作工作台</p>

<p align="center">
  Windows · macOS · Linux<br />
  作品资料 · 小说改编 · 素材资产 · 提示词 · 分镜整理 · 大模型制作包
</p>

SceneDesk 面向短视频和短剧制作者，把小说、剧本、参考图、角色造型、提示词、分镜和模型生成记录整理到同一个作品工作区，方便从 0 到 1 建立可交给外部大模型的制作资料。

应用内集成思源编辑内核，作品资料和公共笔记支持目录、正文、基础排版和自动保存。SceneDesk 负责整理和交接创作资料，画面生成、视频生成、配音和剪辑继续使用外部工具。

目前为公开测试版本，欢迎通过当前仓库的 Issues 反馈问题和提出建议。

[下载与安装](#下载与安装) · [签名与安装信任](#签名与安装信任) · [功能一览](#功能一览) · [快速上手](#快速上手) · [本地开发](#本地开发) · [问题反馈](#问题反馈)

## 下载与安装

当前版本为 `1.0.0-beta.1`，在 [GitHub Releases](https://github.com/yuanshenbin/SceneDesk/releases/latest) 选择适合自己系统和芯片架构的安装包。

| 系统 | 架构 | 安装包 |
| --- | --- | --- |
| Windows | Intel / AMD x64 | `.exe` 安装程序 |
| macOS | Apple Silicon / M 系列，arm64 | 对应的 arm64 `.dmg` 或 `.zip` |
| macOS | Intel，x64 | 对应的 x64 `.dmg` 或 `.zip` |
| Linux | Intel / AMD x64 | `.AppImage` 或 `.deb` |

## 签名与安装信任

### 签名材料

SceneDesk 的 macOS、Windows 和 Linux 签名彼此独立。打包电脑需要保存完整的 `certificates/` 文件夹，不能在另一台电脑重新生成，否则签名身份会改变。

```text
certificates/
├── signing.json                    # 私钥密码、签名名称和固定指纹，仅打包端保存
├── scenedesk-mac.p12               # macOS 私钥证书，仅打包端保存
├── scenedesk-mac.cer               # macOS 公钥证书，可随安装包分发
├── scenedesk-windows.pfx           # Windows 私钥证书，仅打包端保存
├── scenedesk-windows.cer           # Windows 公钥证书，可随安装包分发
├── scenedesk-linux-private.asc     # Linux GPG 私钥，仅打包端保存
└── scenedesk-linux-public.asc      # Linux GPG 公钥，可随安装包分发
```

第一次创建签名材料时，只在可信的打包电脑执行：

```bash
npm run signing:init
node scripts/internal-signing.mjs check mac win linux
```

初始化需要 OpenSSL 3、GnuPG 和 Node.js。脚本生成有效期 30 年的材料，已有完整材料时不会覆盖。证书续期时保留原私钥并执行：

```bash
npm run signing:renew
```

续期后 macOS 和 Windows 公钥指纹会变化，需要重新分发公钥并重新信任。Linux GPG 指纹保持不变。私钥、密码配置和 `signing.json` 不得放进 GitHub 或 Release 附件。

### 按平台构建

构建入口会先检查目标平台的签名材料，检查失败时不会清理产物、递增版本或开始打包。

| 目标 | 检查 | 构建 |
| --- | --- | --- |
| macOS | `node scripts/internal-signing.mjs check mac` | `npm run build:mac` |
| Windows | `node scripts/internal-signing.mjs check win` | `npm run build:win` |
| Linux | `node scripts/internal-signing.mjs check linux` | `npm run build:linux` |

macOS 构建会把 `scenedesk-mac.p12` 导入本次构建专用的临时钥匙串，完成后删除。Windows 使用 `scenedesk-windows.pfx` 为程序和 NSIS 安装器签名。Linux 使用 `scenedesk-linux-private.asc` 为 AppImage、deb 和 `SHA256SUMS` 生成分离签名。

### 首次安装与信任配置（请先阅读）

SceneDesk 由个人维护，使用免费生成的固定自签名证书。Windows、macOS 请同时下载 Release 中对应的公开证书，完成下列配置后再正常打开应用。这些证书与 HTTPS 抓包证书无关。证书、构建目录和私钥不提交到仓库。

#### Windows

下载 EXE 和 `scenedesk-windows.cer`。在开始菜单搜索 `cmd`，右键「命令提示符」，选择**以管理员身份运行**。将下面命令中的 `D:\SceneDesk\scenedesk-windows.cer` 替换成证书在你电脑上的实际完整路径，保留路径两侧的英文半角单引号，路径前后不要添加空格，再复制到管理员 CMD 窗口中执行，无需先打开 PowerShell。

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "Import-Certificate -FilePath 'D:\SceneDesk\scenedesk-windows.cer' -CertStoreLocation 'Cert:\LocalMachine\Root'"
```

输出中显示 `LocalMachine\Root`、证书指纹（`Thumbprint`）和 `SceneDesk Internal Windows`，且没有报错，即表示证书导入成功，**整台电脑的所有 Windows 账号均信任该证书**，无需为每个账号重复导入。这一步只配置证书信任，还需运行 EXE 完成安装，再从开始菜单或桌面快捷方式正常打开 SceneDesk，日常运行无需管理员权限。证书信任不会自动为所有账号安装应用，SmartScreen 仍可能提示。

#### macOS

当前 macOS 安装包使用自签名证书，尚未经过 Apple 公证。首次打开可能提示「Apple 无法验证 SceneDesk 是否包含恶意软件」。

先下载并双击 `scenedesk-mac.cer`，导入「登录」钥匙串，在「钥匙串访问」找到 `SceneDesk Internal macOS`，将其「代码签名」设为「始终信任」，关闭窗口并按系统提示确认。

再打开 DMG，将 SceneDesk 拖入「应用程序」并推出安装磁盘。确认安装包来自本项目 Releases 后，在「终端」中执行下面的命令：

```bash
xattr -dr com.apple.quarantine "/Applications/SceneDesk.app"
```

该命令仅移除 SceneDesk 的下载隔离标记，不能替代代码签名信任。如果提示权限不足，可改用 `sudo xattr -dr com.apple.quarantine "/Applications/SceneDesk.app"`，按提示输入电脑登录密码。执行成功后，从「应用程序」、启动台或 Dock **手动打开 SceneDesk** 即可，以后无需每次执行命令。

也可以在启动被拦截后，进入「系统设置 → 隐私与安全性」，找到 SceneDesk 并选择「仍要打开」。本机放行不等于完成 Apple 公证。

#### Linux

Linux 无需安装 Windows/macOS 证书，也无需执行 `xattr`。AppImage 首次运行可能需要增加执行权限，在下载目录打开终端，将下面文件名替换为实际下载的文件名后执行：

```bash
chmod +x "./SceneDesk-版本号-x64.AppImage"
"./SceneDesk-版本号-x64.AppImage"
```

以后可双击 AppImage 正常打开，请放在当前用户可写的目录，在线更新需要文件及父目录可写。Debian / Ubuntu 也可安装 `.deb`，安装后从系统应用菜单打开，更新时可能需要管理员授权。Linux 的 GPG 公钥用于验证下载文件，不是启动或在线更新的必装证书。

Release 中的 Linux 公钥用于核对安装包和校验文件：

```bash
gpg --import scenedesk-linux-public.asc
gpg --show-keys --with-fingerprint scenedesk-linux-public.asc
gpg --verify SHA256SUMS.asc SHA256SUMS
sha256sum --check SHA256SUMS
```

安装者应先通过可信渠道核对公钥指纹，再按系统提示完成首次信任。进入「设置 → 关于 SceneDesk」可以检查并下载新版本，更新需满足对应平台的签名和目录权限要求。具体发布版本、安装包和变更记录以 Releases 为准。

## 功能一览

| 分类 | 功能 | 用途 |
| --- | --- | --- |
| 作品管理 | 创作预览 | 查看当前作品方向、制作阶段和入口 |
| 作品管理 | 小说改编 | 按书名、章节和页码保存原文、前情与改编剧本 |
| 作品管理 | 短剧资料 | 管理选题、拆片、人物、分集大纲、场次剧本和资料正文 |
| 公共工作区 | 笔记本 | 创建多级文件夹和笔记，记录日常资料、课程内容和学习笔记 |
| 素材整理 | 资产库 | 索引本地图片，按目录标签、用途、角色和造型筛选 |
| 素材整理 | 公共资产 | 在多个作品之间共享同一份图片索引，原图仍保留在原位置 |
| 提示词 | 项目提示词 | 按项目保存分类、标签、正文、参考图和历史版本 |
| 提示词 | 通用模板 | 保存可跨项目复制的文本模板，不携带项目图片和生成记录 |
| 提示词 | 生成记录 | 登记外部生成图片、使用的大模型、提示词正文快照和调整备注 |
| 制作整理 | 分镜台 | 保存镜号、集场、场景、景别、时长、对白、参考图和输出图 |
| 制作整理 | 模型制作包 | 导出分集剧本、场次定稿、镜头清单、提示词、参考图和角色资料 |

## 创作流程

1. 创建作品，填写题材、受众、画幅、平台、目标集数和时长。
2. 导入小说或资料，逐章记录原文和改编稿，或直接创建原创剧本。
3. 在资产库整理角色、造型、场景、道具和画风参考图。
4. 在分镜台按场次拆分镜头，补充画面描述、对白、时长和参考图。
5. 在提示词页面保存可复用的描述，登记外部模型和生成结果。
6. 导出模型制作包，交给外部大模型完成画面或视频生成。

SceneDesk 保留制作时使用的文本、提示词和参考图关系，剧本改动后可以回查需要复核的镜头。应用不提供视频剪辑、成片审看、在线模型调用、素材下载和云同步。

## 快速上手

### 创建作品与整理资料

1. 打开「创作预览」，创建一个作品。
2. 在「小说改编」中导入或录入原文，按章节保存改编稿。
3. 在「短剧资料」中建立分集、场次和阶段资料。
4. 点击资料标题进入正文编辑，正文会自动保存到本机应用数据目录。

### 导入素材

1. 打开「资产库」，选择「导入目录」或「导入图片」。
2. 导入目录时，目录下各层子文件夹名称会自动成为图片标签，目录层数不限制为固定层级。
3. 原图不会移动、压缩或删除，移除索引只解除 SceneDesk 内的引用。
4. 在图片详情中补充用途、角色、造型和共享范围。

### 整理提示词和分镜

1. 在「提示词」中保存标题、分类、标签、正文和参考图。
2. 登记外部生成图片时填写实际使用的大模型名称、版本和调整备注。
3. 在「分镜台」中建立镜头，关联提示词、参考图和输出图片。
4. 在制作包页面检查缺失资料后导出交接文件。

### 公共笔记

「公共工作区 → 笔记本」与当前作品无关，可以创建文件夹、子文件夹和笔记。笔记标题和正文自动保存，支持右键重命名、移动、复制、置顶、加星、回收站恢复和正文搜索。

## 本地开发

需要 Node.js、npm 和 Git。安装版用户无需为了运行应用另行安装 Node.js。

```bash
npm ci
npm run siyuan:stage
npm run dev
```

类型检查和编译检查：

```bash
npm run typecheck
npm run build:compile
```

测试命令：

```bash
npm test
npm run test:ui
npm run test:dev
```

`test:ui` 使用 Chrome 无头运行，系统文件选择器、剪贴板和部分 Electron 原生能力由替身提供。真实安装包的签名、原生窗口、系统选择器和三平台安装流程需要在对应平台验收。

### 构建安装包

```bash
npm run build -- --target mac-arm64
npm run build -- --target mac-x64
npm run build -- --target win-x64
npm run build -- --target linux
```

构建脚本会先检查目标平台的思源资源和签名材料，再编译并调用 electron-builder。输出目录为 `release.noindex/`，构建成功后只保留当前版本的交付文件。正式版递增补丁号，测试版递增 `-beta.n` 序号，也可以使用 `--no-bump-version` 保留当前版本。

内置思源按 `desktop-zh-basic` 资源档位准备，只保留桌面编辑器、中文界面和基础编辑资源。资源准备、许可证和对应源码入口随安装包进入 `siyuan/` 目录。

## 数据与安全边界

- 素材索引只保存用户主动选择的本地文件，不自动扫描全盘、不上传、不下载、不修改源文件。
- IPC 只接受白名单参数和已授权的项目、资产、文档 ID，renderer 不提交任意路径。
- 正文、项目元数据和公共笔记保存在本机。正常退出会等待已接收的保存请求，强制结束程序或断电时不能保证最后一次输入已经写入。
- 内置思源使用应用专用工作区和本机回环地址，编辑器视图不允许外部网络请求和外部跳转。

## 问题反馈

欢迎通过当前仓库的 **Issues** 提交问题和功能建议。

反馈问题时，请尽量提供应用版本、操作系统与芯片架构、复现步骤、预期结果和实际结果。日志、截图和素材路径请先脱敏。

## 致谢

感谢 [Electron](https://www.electronjs.org/)、[React](https://react.dev/)、[Ant Design](https://ant.design/)、[electron-vite](https://electron-vite.org/)、[electron-builder](https://www.electron.build/) 和 [思源笔记](https://github.com/siyuan-note/siyuan) 等开源项目。

第三方项目遵循各自的许可证和分发要求。
