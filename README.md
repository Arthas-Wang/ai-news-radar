# AI Signal Board

中文 | [English](#english)

高质量 AI/科技新闻聚合项目，支持静态网页展示、24h 增量更新、WaytoAGI 更新日志、OPML RSS 批量接入、失败源替换与告警。

说明：本仓库已适配公开发布，**不会包含作者私有 RSS 订阅文件**。

## 中文

### 0. 在线入口

- 线上页面：
  - `https://learnprompt.github.io/ai-news-radar/`
- 说明：
  - 日常查看请打开这个页面，不要直接打开 `data/latest-24h.json`
  - GitHub Actions 会持续更新 `data/*.json`，GitHub Pages 会展示最新页面

### 1. 这个项目每天更新需要一直开 Codex 吗？

不需要。  
你只要执行一个命令，或者直接用 GitHub Actions 定时运行即可。

- 本地命令（一次）：
  - `python scripts/update_news.py --output-dir data --window-hours 24 --rss-opml feeds/follow.opml`
- 自动化（推荐）：
  - `.github/workflows/update-news.yml` 已配置定时任务，默认每 30 分钟自动更新并提交数据。

### 2. 主要能力

- 官方 AI 节点直连（OpenAI News / OpenAI Codex Changelog / OpenAI Skills / Anthropic / Google DeepMind / Google AI / Hugging Face / GitHub AI）
- 高信号日报补充（AI Breakfast）
- 9 个聚合源补充覆盖（TechURLs / Buzzing / Info Flow / BestBlogs / TopHub / Zeli / AI HubToday / AIbase / NewsNow）
- OPML RSS 批量接入（私有文件 `feeds/follow.opml`，仓库提供模板 `feeds/follow.example.opml`）
- 24h 双视图：`AI强相关` / `全量`
- 全量模式去重开关
- AI 模式默认去重
- 站点与分区聚合展示
- 中英双语标题显示
- WaytoAGI：`当天` / `近7日` 切换
- RSS 失败源自动处理：
  - 能替换官方源则自动替换
  - 无官方 RSS 的源自动跳过，避免浪费抓取时间
- 告警数据输出：
  - `failed_feeds` / `zero_item_feeds` / `skipped_feeds` / `replaced_feeds`

### 2.1 自定义信息源与 Agent 工作流

这个项目采用双层设计：

- 默认层：给普通 AI 爱好者直接使用的 `AI强相关` 信号流。
- 进阶层：给维护者使用的 OPML、自定义源、源健康状态与 GitHub Actions 部署配置。

添加自己的信息源时，优先使用 `feeds/follow.opml` 或 GitHub Secret `FOLLOW_OPML_B64`，不要把私有订阅文件提交到仓库。
详细覆盖策略见 `docs/SOURCE_COVERAGE.md`。给 Codex / Claude Code 使用的项目 Skill 在 `skills/ai-news-radar/SKILL.md`。

源可靠性说明：

- OpenAI News RSS 不覆盖所有开发者侧小更新；因此默认层额外抓取 OpenAI Codex Changelog，并过滤接入 OpenAI Skills 仓库里和 Codex/宠物相关的更新。
- X/Twitter 没有官方 RSS。`feeds/social-x.example.opml` 提供 Karpathy 的 RSSHub 候选示例，适合按需复制进自己的 OPML；公共默认层不强依赖它，避免第三方 X 桥不稳定时拖垮站点。
- AI Breakfast 的 Beehiiv 原始 `/feed` 在命令行和 GitHub Actions 场景可能被 Cloudflare 拦截；默认层通过 Jina Reader 读取公开归档页，只取公开标题和链接。

### 3. 数据输出

- `data/latest-24h.json`
- `data/archive.json`
- `data/source-status.json`
- `data/waytoagi-7d.json`
- `data/title-zh-cache.json`

### 4. 快速开始

```bash
cd ai-news-radar
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp feeds/follow.example.opml feeds/follow.opml
# 把你自己的 OPML 内容替换到 feeds/follow.opml（不要提交到仓库）
python scripts/update_news.py --output-dir data --window-hours 24 --rss-opml feeds/follow.opml
python -m http.server 8080
```

打开：`http://localhost:8080`

### 5. Secrets / API 配置说明（重要）

默认情况下，本项目 **不需要任何 API Key** 才能运行核心抓取流程。  
你目前没有提供 API 密钥，仓库中也不会写入任何密钥信息。

推荐仅在运行环境中配置（不要提交到仓库）：

- 代理（可选）：
  - `HTTP_PROXY`
  - `HTTPS_PROXY`
- 如果你未来接入私有 API/私有 RSS：
  - 把密钥放到 GitHub Secrets（Actions）或本地环境变量
  - 不要写入代码、README、日志、`.env` 示例中的真实值
- 私有 RSS OPML（推荐）：
  - GitHub Actions Secret：`FOLLOW_OPML_B64`
  - 生成方式（macOS/Linux）：
    - `base64 < feeds/follow.opml | pbcopy`（macOS）
    - 然后把内容粘贴到 GitHub 仓库的 Secrets

### 6. GitHub 自动更新

工作流：`.github/workflows/update-news.yml`

- 定时：每 30 分钟
- 任务：执行抓取命令并提交 `data/*`
- RSS OPML：若设置了 `FOLLOW_OPML_B64`，工作流会自动解码为 `feeds/follow.opml`
- 推送权限：使用 `GITHUB_TOKEN`（workflow 内）

---

## English

### 0. Live Site

- Live page:
  - `https://learnprompt.github.io/ai-news-radar/`
- Note:
  - Use this page for daily reading instead of opening `data/latest-24h.json` directly
  - GitHub Actions keeps `data/*.json` updated and GitHub Pages serves the latest UI

Production-grade AI/tech news aggregator with a static web UI, 24h updates, WaytoAGI timeline, and OPML RSS ingestion.

This repo is safe for public release and does **not** include the maintainer's private RSS subscription file.

### 1. Do I need Codex running all day?

No.  
You only need to run one command, or let GitHub Actions run it on schedule.

- One-shot local command:
  - `python scripts/update_news.py --output-dir data --window-hours 24 --rss-opml feeds/follow.opml`
- Scheduled automation:
  - `.github/workflows/update-news.yml` runs every 30 minutes and commits updated data.

### 2. Core features

- Multi-source web aggregation
- First-class official AI update sources (OpenAI News / OpenAI Codex Changelog / OpenAI Skills / Anthropic / Google DeepMind / Google AI / Hugging Face / GitHub AI)
- High-signal newsletter coverage (AI Breakfast)
- 9 aggregator sources for breadth (TechURLs / Buzzing / Info Flow / BestBlogs / TopHub / Zeli / AI HubToday / AIbase / NewsNow)
- OPML RSS ingestion (private `feeds/follow.opml`; template provided as `feeds/follow.example.opml`)
- 24h two-mode UI (`AI-focused` / `All`)
- Dedup toggle in All mode, dedup-by-default in AI mode
- Site + section grouping
- Bilingual title rendering
- WaytoAGI toggle (`Today` / `Last 7 Days`)
- RSS resilience:
  - Auto-replace failed feeds with official sources when available
  - Auto-skip unsupported source types (to save crawl time)
- Alert-friendly status output (`failed_feeds`, `zero_item_feeds`, `skipped_feeds`, `replaced_feeds`)

### 2.1 Custom sources and agent workflow

This project uses a two-layer design:

- Default layer: a curated `AI-focused` signal feed for ordinary AI enthusiasts.
- Advanced layer: OPML, custom sources, source health, and GitHub Actions deployment settings for maintainers.

For custom sources, prefer `feeds/follow.opml` locally or GitHub secret `FOLLOW_OPML_B64` in Actions. Do not commit private subscription files.
See `docs/SOURCE_COVERAGE.md` for source strategy. The Codex / Claude Code project skill lives at `skills/ai-news-radar/SKILL.md`.

Source reliability notes:

- OpenAI News RSS does not cover every developer-side minor update, so the default layer also reads the OpenAI Codex Changelog and filtered OpenAI Skills updates related to Codex/pets.
- X/Twitter has no official RSS. `feeds/social-x.example.opml` includes a Karpathy RSSHub candidate for personal OPML use, but the public default layer does not depend on it.
- AI Breakfast's Beehiiv `/feed` can be blocked by Cloudflare from CLI or GitHub Actions, so the default layer reads the public archive through Jina Reader and extracts public titles and links.

### 3. Output files

- `data/latest-24h.json`
- `data/archive.json`
- `data/source-status.json`
- `data/waytoagi-7d.json`
- `data/title-zh-cache.json`

### 4. Quick start

```bash
cd ai-news-radar
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp feeds/follow.example.opml feeds/follow.opml
# Replace with your own OPML subscriptions (do not commit this file)
python scripts/update_news.py --output-dir data --window-hours 24 --rss-opml feeds/follow.opml
python -m http.server 8080
```

Open: `http://localhost:8080`

### 5. Secrets / API notes

By default, this project needs **no API keys** for the core pipeline.  
No secrets are stored in this repo.

If you later add private APIs/feeds:

- Use environment variables or GitHub Secrets
- Never commit real tokens/keys
- For private RSS OPML in GitHub Actions, store `base64` content in secret `FOLLOW_OPML_B64`
