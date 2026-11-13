---
name: release-check
description: 开源发布检查：用户要把 GitHub 仓库开源、发新版本、或者问「用户能不能自动更新」时，先判断仓库是什么（Claude/Codex 插件和 skill、网页、npm/Python 包、Mac 应用、浏览器插件、脚本），跑脚本逐项查「用户装一次能不能自动收到更新」和开源基本功（LICENSE、密钥、个人路径），给 ✅/❌ 清单，用户点头后照 ariel-broll-video 验证过的结构补齐、涨版本号、开合并申请。触发词：开源发布检查、发布前检查、能不能自动更新、用户收不到更新、发新版、bump 版本、release check、准备开源。Claude 和 Codex 通用。
---

# 开源发布检查（Claude / Codex 通用）

**说话方式**：大白话、少说话。能让用户点的就不让打字，一次只问一个问题。报告只列有问题的项，没问题的合成一句「其余 N 项都没问题」。

本 skill 文件夹里有：
- `scripts/check_repo.py`：检查脚本（只读，Python 自带库）；`scripts/update_check.py`：本 skill 自己查新版本
- `assets/`：要补进用户仓库的现成文件——`bump.py`（一键改齐版本号）、`check_version_bump.py` + `version-check.yml`（GitHub 自动检查忘涨版本号）、`update_check.py`（skill 开工前查新版本）
- `references/plugin-layout.md`：插件市场仓库结构和各文件模板
- `references/readme-install.md`：README 安装段模板
- `references/channels.md`：不是插件的仓库怎么做自动更新

## 第 0 步：开工前
`python3 <本skill>/scripts/update_check.py`：查本 skill 有没有新版本。只在最后一行是 `STATUS=updated` 或 `outdated` 时跟用户说一句，其他情况不吭声。

## 第 1 步：拿到仓库
- 用户给本地路径 → 直接用。给 GitHub 链接 → 先找本机有没有这个仓库（`git remote -v` 对得上），没有就 `git clone` 到用户指定的位置，没指定就问一次放哪。
- `git status --porcelain` 有没提交的改动 → 告诉用户，问「先提交 / 先放一边（stash）/ 就这样查」。只检查不改的话可以直接查。
- `git fetch origin`，后面拿 `origin/main` 比较「改了内容有没有涨版本号」。默认分支不叫 main 就用 `--base origin/<默认分支>`。

## 第 2 步：跑检查
`python3 <本skill>/scripts/check_repo.py <仓库> [--base origin/main]`

每行一项：✅ 通过、❌ 必须修（不修用户收不到更新或有法律/安全问题）、⚠️ 建议修、ℹ️ 说明。最后一行 `STATUS=PASS / WARN / FAIL`。
检查项编号：T0 仓库类型；P1–P7 插件自动更新（市场清单、版本号一致、改了没涨、README 装法、开工查新版、一键改版本号、GitHub 自动检查）；C1 其他类型的发货渠道；G1–G5 开源基本功（LICENSE、README、密钥、个人路径、.env）。

## 第 3 步：给用户看结果 —— 🔴 STOP
用一张小表：项目 | 问题（大白话）| 不修会怎样。末尾一句总评（「能自动更新」/「改完这 N 处就能自动更新」/「这类仓库只能做到提醒更新」）。
然后问一个问题，给按钮：**全部补上** / **只补 ❌** / **先不补**。用户没点之前不改任何文件。

## 第 4 步：补齐（在新分支 `release-check/<年月日>` 上做）
| 检查项 | 怎么补 |
|---|---|
| P1 没有插件市场结构 | 照 `references/plugin-layout.md` 建文件。要挪动已有的 SKILL.md 时，**先 🔴 STOP 单独问一次**（会改变文件位置）。 |
| P1 缺 Codex 两个文件 | 照 plugin-layout.md 补，version 抄 Claude 那份。 |
| P2 版本号不一致 | `python3 tools/bump.py <Claude plugin.json 里的版本号>` 一次改齐。 |
| P4 README | 按 `references/readme-install.md` 补缺的那几行，只加不删用户原有内容。 |
| P5 开工不查新版本 | 复制 `assets/update_check.py` 进 skill 的 `scripts/`，改 `REPO`、`MARKET` 两行；在 SKILL.md 开工第一步加：「`python3 <本skill>/scripts/update_check.py`：只在最后一行是 `STATUS=updated` 或 `outdated` 时跟用户说一句」。 |
| P6 没有一键改版本号 | 复制 `assets/bump.py` 到 `tools/bump.py`。 |
| P7 没有 GitHub 自动检查 | 复制 `assets/check_version_bump.py` 到 `tools/`，`assets/version-check.yml` 到 `.github/workflows/`。 |
| C1 其他类型 | 照 `references/channels.md` 改仓库里能改的；要用户本人开账号、授权的（npm、PyPI、Vercel、应用商店），列出步骤让用户自己做。 |
| G1 没 LICENSE | 问用户用哪种，按钮：**MIT（推荐）** / **Apache-2.0** / **先不加**。版权人写用户的 GitHub 用户名和当年年份。 |
| G3 有密钥 | **不自己处理。** 告诉用户：去对应网站作废这个密钥，再删文件里的；已经推送过的当作已泄露。 |
| G4 个人路径 | 改成 `~` 或相对路径。 |
| G5 提交了 .env | `git rm --cached <文件>`，加进 `.gitignore`，并按 G3 提醒作废里面的密钥。 |

## 第 5 步：涨版本号，再查一遍
- 这个分支相比 `origin/main` 改了插件内容（SKILL.md、scripts、assets、references）→ `python3 tools/bump.py`（最后一位 +1；用户说了大版本就按用户说的）。只改了 README、tools/、.github/ 不用涨。
- 再跑一次第 2 步。**还有 ❌ 不进下一步**，回第 4 步。

## 第 6 步：提交、推送 —— 🔴 STOP
给用户看：改了哪些文件（每个一句话）、版本号从几到几、检查结果。问：**推送并开合并申请** / **先不推**。
用户同意后：`git commit` → `git push -u origin <分支>` → `gh pr create`。合并申请的说明里写清：改了什么、版本号、合并后用户怎么收到更新。合并按钮留给用户点。

## 第 7 步：合并后验证
用户说合并了 → 读 `https://raw.githubusercontent.com/<用户名>/<仓库>/main/plugins/<插件名>/.claude-plugin/plugin.json`，版本号等于新版本号才算发出去了。告诉用户：Claude 桌面 app 会自动同步；Claude Code / Codex 开了自动更新的，下次启动就是新版。

## 出错了怎么办
| 情况 | 怎么办 |
|---|---|
| 路径不是 git 仓库 | 只跑检查（P3 会跳过），告诉用户「没有 git 记录，查不了改了没涨版本号」。要补齐就先问用户要不要 `git init`。 |
| `git fetch` 失败（没网、没权限） | 用 `--base` 指向本地的 main 分支再跑；本地也没有就跳过 P3，并在报告里写明。 |
| 浅克隆，P3 显示「没法比较」 | `git fetch --unshallow`，再跑。 |
| 检查脚本报错退出 | 把报错原文告诉用户，按报错修，不要凭感觉手查代替。 |
| `gh` 没登录 | 请用户在终端跑 `gh auth login`；或者只推送分支，把 GitHub 上开合并申请的链接发给用户自己点。 |
| `git push` 被拒（不是自己的仓库） | 告诉用户：要先 fork 到自己账号，问要不要 fork。 |
| 合并后 raw 文件的版本号还是旧的 | GitHub 缓存最多约 5 分钟，等 5 分钟再查一次；还是旧的，就看是不是合并到了别的分支。 |
| 仓库里有好几个插件 | 每个插件单独看 P1–P3；`bump.py --plugin <名字>` 只涨改过的那个。 |

## 不要做的事
| 不要 | 为什么 / 改成 |
|---|---|
| 用户没点头就改文件、推送、开合并申请 | 第 3、6 步是 🔴 STOP，必须等用户。 |
| 直接推到 main | 一律开新分支 + 合并申请，合并按钮留给用户。 |
| 手改几处版本号 | 容易改漏，一律用 `tools/bump.py`。 |
| 改了插件内容不涨版本号 | 用户收不到更新，这是本 skill 要防的头号问题。 |
| 只改 marketplace.json 的版本号 | plugin.json 优先级更高，等于没改。 |
| 替用户处理密钥、注册 npm / PyPI / 应用商店账号 | 涉及账号和安全，只给步骤，用户自己做。 |
| 删掉或改写别人的 LICENSE、版权声明（fork 来的仓库） | 违反开源许可。保留原文，在 README 写明基于谁改的。 |
| 告诉用户「已经能自动更新了」但没做第 7 步验证 | 没验证就说「合并后我帮你确认一下」。 |
