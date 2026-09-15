<div align="center">

# 花笺 · 剪贴板自动存档补丁

给开源便签 [花笺 Floral Notepaper](https://github.com/Achilng/floral-notepaper) 追加「自动保存剪贴板内容」功能的第三方补丁<br>
自带 GitHub Actions 云端构建,无需本地配置 Rust 环境即可产出 Windows 安装包

[下载](#-下载) · [功能说明](#-功能说明) · [自己构建](#-自己构建) · [工作原理](#-工作原理)

[![Build](https://github.com/poweruser123456/floral-notepaper-patchset/actions/workflows/build.yml/badge.svg)](https://github.com/poweruser123456/floral-notepaper-patchset/actions/workflows/build.yml)
![License](https://img.shields.io/badge/License-MIT-yellow)
![Tauri v2](https://img.shields.io/badge/Tauri-v2-%2324C8D8?logo=tauri)

</div>

---

## 📖 这是什么

[花笺](https://github.com/Achilng/floral-notepaper) 是一款优秀的本地便签工具,但没有「自动保存剪贴板历史」的能力。

本仓库以**补丁(git patch)**的形式为它实现该功能:不 fork、不修改上游任何代码,只在云端构建时把补丁应用到上游源码上一并编译。

- 想直接用 → [下载](#-下载)
- 想了解开关和隐私设计 → [功能说明](#-功能说明)
- 想自己编译一份 → [自己构建](#-自己构建)

## ✨ 功能说明

安装并开启后,在**任何应用**里复制的纯文本都会被自动存进笔记:

- **自动归档** — 打开花笺设置里的「剪贴板自动存档」开关后,浏览器、编辑器等任意程序中复制的内容自动存入笔记,复制完就丢的内容不再漏掉
- **按天归档** — 每天一条「剪贴板归档」笔记,每条记录带时间戳,方便回溯
- **细节处理** — 超长内容按 2 万字符截断并标记;复制图片/文件静默跳过;重复内容自动去重
- **隐私设计** — 默认关闭;关闭即完全停止监听;所有数据只写在本机花笺笔记目录,不联网上传

## 📥 下载

从 [Actions](https://github.com/poweruser123456/floral-notepaper-patchset/actions) 页面进入最近一次成功的构建,在底部 **Artifacts** 区下载 `floral-notepaper-patched-windows`,内含:

| 文件 | 适合 |
|------|------|
| `花笺_x.x.x_x64-setup.exe` | 想正式安装使用(会覆盖官方版) |
| `floral-notepaper-portable.exe` | 想先试用:单文件直接运行,可与官方版**并存** |

> [!TIP]
> 试用建议用便携版:官方版照常使用,便携版里打开开关体验新功能,互不干扰。

构建产物保留 14 天;正式版 Release 将在功能验证后发布。

## 🔨 自己构建

准备 Node.js 24 与 Rust stable 工具链,然后:

```bash
git clone https://github.com/Achilng/floral-notepaper.git
cd floral-notepaper
git apply /path/to/0001-clipboard-auto-archive.patch   # 本仓库 patches/ 目录下
npm ci
npm run tauri build
```

也可以 **fork 本仓库**:push 即自动云端构建,产物在 Actions 页,零本地环境。

## ⚙️ 工作原理

<details>
<summary>点开查看构建流程与实现说明</summary>

云端构建流程(每次 push 或手动触发):

```
拉取上游源码,锁定基线提交 69a43ae
        ↓
应用 patches/0001-clipboard-auto-archive.patch
        ↓
npm ci → npm run tauri build
(Windows Server,MSVC 工具链,与官方发布完全同口径)
        ↓
产出便携版 + 安装包 → 上传 Artifacts
```

功能改动:13 个文件(+552/−1),核心为新增 Rust 模块 `clipboard_watch.rs`,监听收敛在 Rust 侧单点,避免多窗口重复写入。剪贴板变更事件由 [tauri-plugin-clipboard-x](https://github.com/ayangweb/tauri-plugin-clipboard-x) 提供。

跟进上游新版本:开发副本 rebase 新基线 → 更新 `build.yml` 中 `UPSTREAM_REF` → 重新生成补丁。

</details>

<details>
<summary>维护者:如何重新生成补丁</summary>

```bash
cd <开发副本>
git diff 69a43ae HEAD -- . ':(exclude).github/workflows/verify.yml' \
  > <本仓库>/patches/0001-clipboard-auto-archive.patch
```

- 排除 `verify.yml`(构建使用本仓库自己的配方)
- 包含 `Cargo.lock`(锁定依赖版本)
- 补丁文件禁止行尾转换:仓库 `.gitattributes` 已声明 `*.patch -text`,勿删

</details>

## 📄 许可证

上游花笺以 [MIT](https://github.com/Achilng/floral-notepaper/blob/main/LICENSE) 授权,© Achilng。本补丁同样以 MIT 发布。

## 🙏 致谢

- [花笺 Floral Notepaper](https://github.com/Achilng/floral-notepaper) — 上游项目
- [tauri-plugin-clipboard-x](https://github.com/ayangweb/tauri-plugin-clipboard-x) — 剪贴板监听插件
- [Tauri](https://tauri.app) — 应用框架
