# Ariel 开源发布检查 ariel-release-check

把仓库开源、或者发新版本之前，让 AI 帮你查一遍：**用户装一次，以后你更新了，他们能不能自动收到？**

- 先认出仓库是什么：Claude / Codex 插件和 skill、网页、npm / Python 包、Mac 应用、浏览器插件、脚本
- 逐项打 ✅ / ❌：插件市场结构、版本号、改了内容有没有涨版本号、README 装法、开源基本功（LICENSE、密钥、个人路径）
- 你点头后照 [ariel-broll-video](https://github.com/nini818726/ariel-broll-video) 验证过的结构补齐，涨版本号，开合并申请
- 顺手给仓库装上「忘涨版本号就拦下合并」的 GitHub 自动检查
- **查项目在哪些 AI 里能完整用，不能用的改好**：默认查 Claude 桌面 app 的 Code 页面和 Codex；你说「还要能在 Cowork 用」或「再加个 XX」，它就照那个 AI 的官方规范把安装文件、说明、不兼容的地方补好，最后给你一张「在哪能用」的表（没真跑过的会老实写「没实测」）

装好后在对话框打 **`/ariel`** 就能找到它（`ariel-release-check`），或者直接对 AI 说：「帮我做一下开源发布检查，仓库是 ……」

## 安装（装一次，以后作者更新会自动到你手里）

**最省事：让 AI 帮你装**
把这句话发给 Claude 或 Codex：「帮我安装插件市场 nini818726/ariel-release-check 里的 ariel-release-check，并打开自动更新」

**Claude 桌面 app（Code 页面和 Cowork 都能用）/ 网页版**
Customize → Plugins → Add → Add marketplace → Add from a repository → 填 `nini818726/ariel-release-check` → 打开 **Sync automatically** → 装 ariel-release-check

**Claude Code**
```
/plugin marketplace add nini818726/ariel-release-check
/plugin install ariel-release-check@ariel-release-check
```
装完打开自动更新：`/plugin` → Marketplaces → 选 ariel-release-check → Enable auto-update（第三方市场默认是关的）

**Codex**
```bash
codex plugin marketplace add nini818726/ariel-release-check
codex plugin add ariel-release-check@ariel-release-check
```

> ⚠️ 直接下载 zip、手动把文件复制进 skills 文件夹也能用，但**不会自动更新**。

## 发新版本（给维护者）
改完 skill 后跑 `python3 tools/bump.py`（最后一位 +1），再提合并申请。忘了跑，GitHub 上的 version-check 会亮红灯。

## 致谢
密钥扫描、个人路径检查的思路参考了 [marcfargas/skills](https://github.com/marcfargas/skills) 的 pre-release（MIT）。

## 许可证
MIT
