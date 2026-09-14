# floral-notepaper patchset(花笺剪贴板自动存档补丁集)

对上游 [Achilng/floral-notepaper](https://github.com/Achilng/floral-notepaper)(花笺,Tauri 2 桌面便签)追加「剪贴板自动存档」功能的**补丁集仓库**。

本仓库**不含上游源码**(只有几十 KB),GitHub Actions 云编译时才拉上游 → 打补丁 → 编译 → 产出 exe。

## 仓库结构

```
patches/0001-clipboard-auto-archive.patch   功能补丁(13 文件,+552/-1,含 Cargo.lock)
.github/workflows/build.yml                 云编译配方:拉上游(锁定基线)→ 打补丁 → npm run tauri build → 出产物
```

## 触发构建

- push 到 `main` 自动触发;或在 GitHub 仓库 Actions 页手动 Run workflow。
- 产物在该次运行页面的 Artifacts 区:`floral-notepaper-patched-windows`
  (内含便携版 `floral-notepaper-portable.exe` 与 NSIS 安装包),保留 14 天。
- 私有库免费额度 2000 分钟/月(Windows 按 2 倍扣),单次构建约 10~20 分钟(首次最慢,之后有 Rust 缓存)。

## 修改代码后重新出补丁

源码开发副本:上游 clone 加上我们的提交 `0af69d1`(位于维护者本机,路径从略)。下文以 `<开发副本>` 指代开发副本工作目录、`<本仓库>` 指代本仓库的工作副本。

1. 在开发副本正常改代码、commit;
2. 重新导出补丁(**排除** verify.yml,**包含** Cargo.lock):

```bash
cd <开发副本>
git diff 69a43ae HEAD -- . ':(exclude).github/workflows/verify.yml' \
  > <本仓库>/patches/0001-clipboard-auto-archive.patch
```

3. commit 并 push 本仓库,云编译自动跑。

> `verify.yml`(开发副本里的项目内 CI)不进补丁:云编译用的是本仓库的 `build.yml`。

## 跟进上游新版本

1. 开发副本:`git fetch origin && git rebase origin/main`(解决冲突);
2. 把本仓库 `build.yml` 里的 `UPSTREAM_REF` 改成新的基线提交短哈希;
3. 按上面命令重新生成补丁、提交、push。

## 云编译环境说明

- Runner:`windows-latest`(4 核 / 16 GB / 14 GB 磁盘,Rust MSVC 工具链),与上游官方发布构建同口径,规避了本机 GNU 工具链缺 MinGW 运行库的问题。
- 构建命令与上游官方 `build-artifacts.yml` 完全一致:`npm ci` → `npm run tauri build`。
