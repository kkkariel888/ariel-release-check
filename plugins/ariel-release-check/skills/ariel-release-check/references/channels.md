# 不是插件的仓库：怎么让用户自动拿到新版

道理都一样：有固定的发货渠道 + 有版本号 + 用户那端会去查。只放在 GitHub 让人下载 zip 的，做不到自动更新。

| 类型 | 发货渠道 | 用户怎么更新 | 要补的东西 |
|---|---|---|---|
| 网页 / 网站小工具 | GitHub Pages、Vercel、Netlify | 打开就是最新版 | 推送到 main 自动上线：GitHub Pages 用官方 `actions/deploy-pages`；或在 Vercel 导入仓库 |
| Node / JS 命令行工具 | npm | `npx <包名>` 每次拿最新版；或 `npm update -g` | `package.json` 写 name / version / bin；发布用 GitHub Actions + npm Trusted Publishing（不存 NPM_TOKEN） |
| Python 工具 | PyPI | `uvx <包名>` 每次拿最新版；或 `pipx upgrade` | `pyproject.toml` 写 version；发布用 `pypa/gh-action-pypi-publish` + PyPI Trusted Publishing |
| Mac 命令行工具 | Homebrew tap（单独仓库 `homebrew-<名>`） | `brew upgrade` | Formula 里的 url 和 sha256 每次发版要更新，可用 GitHub Actions 自动改 |
| Mac 应用 | 应用内 Sparkle，或 Homebrew Cask | 自动提示更新 | 接 Sparkle：Info.plist 写 `SUFeedURL`，每次发版更新 appcast.xml |
| 浏览器插件 | Chrome 应用商店 / Edge 商店 | 商店自动更新 | 上架；manifest.json 的 version 每次发版 +1 |
| 脚本 / 模板 / 教程 | 只有 git | 用户 `git pull` | 能做的只有「有新版就提醒」：照 assets/update_check.py 的思路，开头查一下 GitHub 上的版本号 |

## 怎么挑
- 用户是 AI 工具的用户 → 做成插件（见 plugin-layout.md），最省心。
- 用户只想打开就用 → 做成网页，永远最新。
- 只能是脚本 → 至少加「有新版就提醒」，并在 README 写清楚怎么更新。
