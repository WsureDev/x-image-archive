# 基于个人XP的grok bot帮我逛X收图项目 · Image Source Archive

## 中文

这是一个**个人爱好与学习用途的图片来源目录**，用于收藏grok bot打标过的图片的目录。

**本仓库只分享 JSONL 中的来源链接和元数据，不分发图片本身。** 仓库和 Release 均不包含原图、缩略图或图片压缩包。原始图片由来源网站提供；链接可能失效，或需要在来源网站登录才能访问。

**本仓库收纳的图片基于本人XP+grok bot理解的本人XP进行纯llm打标，如果与你的XP或者预期不符，请自行搭建管线，本项目不对产出数据质量做保证，更不对任何人的下议院做保证**

### 原理与流程

1. 先交给grok bot自己的X账号，然后设置XP，审核要求
2. 接着让grok bot每天白天隔3小时从首页刷100-150张图，逐张审核，剔除非人物、不够诱惑、不含xp、漏点的图。
3. 将通过的图提交到Cloudfare后端（一个由 queue+D1纯vibe实现的暂存队列）
4. 在NAS端消费这个队列
5. 每日数据汇总后，commit到此项目数据集，并push

```mermaid
flowchart TD
  A["个人 X 账号 / Personal X account"] --> B["Grok bot：设置 XP 与审核规则 / Set preferences and review rules"]
  B --> C["白天每 3 小时浏览首页 / Browse the home feed every 3 daytime hours"]
  C --> D["每批 100–150 张，逐张 LLM 审核 / Review 100–150 images per batch"]
  D --> E{"符合偏好与审核规则？ / Pass review?"}
  E -->|"否 / No"| F["跳过 / Skip"]
  E -->|"是：仅提交来源元数据 / Yes: submit source metadata"| W
  subgraph CF["Cloudflare 暂存后端 / Cloudflare buffer"]
    W["Worker API"] --> Q["主队列 / Main queue"]
    W --> T["18 小时延迟检查 / 18-hour completion check"]
    T -->|"尚未完成 / Not yet archived"| DB["D1 离线待办 / Offline backlog"]
  end
  Q --> N["NAS 消费程序 / NAS consumer"]
  DB --> N
  N --> P["私人图库与 SQLite / Private gallery and SQLite"]
  N -.->|"完成回执 / Completion receipt"| W
  P -->|"仅来源链接与元数据 / Source links and metadata only"| U["主进程内置发布器 / Built-in publisher"]
  U --> G["git fetch → git pull → commit → push"]
  G --> J["每周 JSONL 与目录 / Weekly JSONL and manifest"]
  G --> R["每月 JSONL Release / Monthly JSONL Release"]
```

- `weeks/YYYY-Www.jsonl`：每周一个文件，日常更新。
- `manifest.json`：文件目录、记录数、校验值和最后发布日期。
- 每月 Release：一个 `YYYY-MM.jsonl`，汇总上个月的入库记录。

JSONL 是“一行一条 JSON”的文本格式。每条记录包含来源图片 URL、原帖链接（如有）、作者、标签、原帖时间 `publishedAt` 和入库时间 `archivedAt`。周文件按入库时间分组，不代表图片一定是在该周创作或发布。

每天发布前会先 `git fetch`、`git pull`，再提交和推送。用户对 README 等说明文档的修改会保留；已发布的月度 Release 保留当时的快照。

## English

This is a **personal hobby and learning project**: a Grok-bot-assisted pipeline for browsing X and cataloguing image sources according to my own preferences ("XP"). The catalog contains sources for images tagged by Grok bot.

**This repository shares source links and metadata in JSONL only. It does not distribute the images themselves.** Neither the repository nor its Releases include originals, thumbnails or image archives. Original images remain on their source websites, where links may expire or require login.

**Selection and tags are generated entirely by an LLM, based on my personal preferences and Grok bot's interpretation of them. If your tastes or expectations differ, please build your own pipeline. This project makes no guarantees about dataset quality—or anyone's libido.**

### How it works

1. Connect a personal X account to Grok bot and specify the preferences and review criteria.
2. During daytime, ask Grok bot to browse 100–150 images from the home feed every three hours and review them one by one. Reject images without a person/character, images that do not meet the desired preferences or level of suggestiveness, and images containing nudity.
3. Submit source metadata for accepted images to a Cloudflare backend built around Queues and D1. The queue carries links and metadata, not image files.
4. The NAS consumer processes queued tasks and organizes the images in a private local gallery with a SQLite catalog. An 18-hour completion check saves unfinished tasks in the D1 backlog so they can be recovered after the NAS has been offline.
5. A publisher inside the same main application takes a daily metadata snapshot. It runs `git fetch` and `git pull` before committing and pushing to this dataset repository. It maintains weekly JSONL files and publishes a JSONL-only monthly Release. The bilingual diagram above shows the complete flow.

### Files and update schedule

- `weeks/YYYY-Www.jsonl`: one file per ISO archive week, updated by the daily publication.
- `manifest.json`: file index, record counts, checksums and the last publication date.
- Monthly Releases: one `YYYY-MM.jsonl` containing the previous month's archive records, published under `archive-YYYY-MM`.

JSONL is plain text with one JSON object per line. Records contain the source image URL, original post URL when available, author, tags, original post time (`publishedAt`) and archive time (`archivedAt`). Weekly grouping follows archive time, not necessarily the creation or publication date of the image.

The default daily publication time is 03:15 in Asia/Shanghai. Scheduling runs inside the main application, with startup recovery and retries after failures. User edits to README and other documentation are preserved. Published monthly Releases remain historical snapshots.

## 相关项目 / Related projects

- [主项目源码与部署说明 / Source code and setup](https://github.com/WsureDev/social-image-archive) — Cloudflare Worker、Queues/D1、NAS 消费图库与内置 JSONL 发布器。 / Worker, Queues/D1, NAS archive and built-in JSONL publisher.
