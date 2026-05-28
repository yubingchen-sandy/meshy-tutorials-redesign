# Tutorials / Blog 信息架构与 SEO 改造提案

> 目标：把"视频墙"形态的 `/tutorials` 改造成结构化知识库；把 Blog 与 Tutorials 按搜索意图和漏斗位置区分；让详情页拿到正确 schema 富摘要而不是误用 VideoObject 流量。

---

## 1. TL;DR

1. **取消 `/tutorials` 下的 Blog Guides 入口**。Blog 内容回归 `/blog/<category>`，必要时 301 旧 URL。
2. **`/tutorials` 改成左侧边栏知识库**，导航以"用户做什么"为分类，而不是 YouTube playlist。
3. **URL 一律扁平：`/tutorials/<slug>`**。分类信号通过 breadcrumb + category 标签 + JSON-LD `BreadcrumbList` 表达，不在 URL 里嵌套。30+ 篇之后再评估是否升级嵌套。
4. **详情页骨架以图文 step-by-step 为主**，视频是可选嵌入。Schema 按内容主体选择：HowTo / VideoObject / Article 三选一，避免硬塞 VideoObject。
5. **每个 YouTube 视频派生一个配套图文页**：视频转写 → LLM 整理步骤 → 人工配图校对。视频党留在 YouTube，搜索流量走图文。

---

## 2. Blog vs Tutorials 定位

| 维度 | `/blog` | `/tutorials` |
|---|---|---|
| 漏斗位置 | 上层（引流、科普、SEO 关键词） | 下层（教学、激活、留存） |
| 搜索意图 | Informational：`what is X` / `X vs Y` / `how does X work` | Transactional / How-to：`how to do X in Meshy` / `Meshy + Blender workflow` |
| 内容形态 | 长文、对比、综述、行业观点、案例 | Step-by-step 可执行，跟着做完拿到产出 |
| 受众心态 | 浏览、闲逛、研究市场 | 带着具体问题来找答案 |
| 主 CTA | "Try Meshy free" → 试用 | "Open this in Workspace" → 复现 |
| 内链方向 | 引向 `/tutorials` 教用户用产品 | 引向 `/pricing`、`/api` 转化 |
| 主 schema | `BlogPosting` / `Article` + `FAQPage` | `HowTo` + `BreadcrumbList` + `FAQPage`（视频主导才上 `VideoObject`） |

**判断一篇内容该去 blog 还是 tutorials 的规则**：

- 用户读完是否**立刻能在产品里做出某个具体产出**？是 → tutorials。否 → blog。
- 标题动词是 "How to..." 且步骤可枚举？→ tutorials。
- 标题是 "X vs Y" / "What is X" / "Top 10 ..." / "The state of ..."？→ blog。

---

## 3. URL 架构

### Tutorials（新 — 扁平 URL）

**决案**：所有教程详情页统一走 `/tutorials/<slug>`，**不嵌套**。分类通过 breadcrumb、category 标签、`BreadcrumbList` schema、以及列表页的 `?category=` 过滤来表达。

```
/tutorials                                  ← 知识库首页 / 列表（左侧栏 + 主区）
/tutorials?category=image-to-3d             ← 列表过滤视图（不是独立页面）
/tutorials?category=rigging
/tutorials?category=workflows-blender
/tutorials?category=api
/tutorials/<slug>                           ← 所有详情页都是这个形态
/tutorials/videos                           ← 全部视频（保留逛逛体验，但不再是首页）
```

**为什么扁平**：
- 8 篇起步，没到需要 URL 层级才能管的体量。
- slug 直接用现成的（如 `character-auto-rigging-workflow`），零迁移负担。
- URL 永远稳定。即便将来增类目，新增的也是 `?category=` 值或 breadcrumb 文案，不动 URL。
- 嵌套 URL 在 50+ 篇时再考虑；届时已有 GSC 数据知道用户实际路径，迁移有依据，可一次性 301。

### Blog（建议）

```
/blog                                       ← 现有
/blog/category/guides                       ← 长文教程类科普
/blog/category/comparisons                  ← X vs Y
/blog/category/industry                     ← 行业 / 案例
/blog/<slug>                                ← 文章
```

---

## 4. 左侧边栏分类（Tutorials Sidebar）

按"用户要做什么"组织，而不是按 YouTube playlist。**每一项指向同一个 `/tutorials` 列表页**，差别只在 `?category=` 过滤值：

```
Core features
  Image to 3D                 /tutorials?category=image-to-3d
  Text to 3D                  /tutorials?category=text-to-3d
  Texturing (PBR / Remesh)    /tutorials?category=texturing
  Rigging                     /tutorials?category=rigging
  Animation                   /tutorials?category=animation
  Multi-view                  /tutorials?category=multi-view

Workflows
  Meshy + Blender             /tutorials?category=workflows-blender
  Meshy + Unity               /tutorials?category=workflows-unity
  Meshy + Unreal              /tutorials?category=workflows-unreal
  Meshy + 3D Printing         /tutorials?category=workflows-3d-printing
  Meshy + ZBrush              /tutorials?category=workflows-zbrush

Developers
  API quickstart              /tutorials?category=api
  Webhooks                    /tutorials?category=webhooks
  Common recipes              /tutorials?category=recipes

────────────────────────────
All video walkthroughs →      /tutorials/videos
```

**约束**：
- 左侧栏二级条目维持在 **15–20 条以内**，超出就抽象成子分类。
- 没有内容的分类不展示（计数为 0 隐藏）。
- 当前 URL（如 `/tutorials/3d-printing-academy`）保留不动；不强行迁到 `?category=`。

---

## 5. 详情页骨架

### 5.1 教程详情页（图文为主，视频可选）

```
─────────────────────────────────────────────
Breadcrumbs:  Home > Tutorials > Image to 3D > Auto-rigging a character
─────────────────────────────────────────────
H1: How to auto-rig a character in Meshy
[Intro paragraph — 3-4 句: outcome / 难度 / 用时 / 前置]
[Hero image: 带注释的产品截图]
[CTA: "Open this in Workspace →"]

## What you'll need                      (H2)
- A character mesh from Meshy
- Browser (no install)

## Step 1 — Upload your model            (H2)
   [annotated screenshot]
   ### Tips                              (H3)
## Step 2 — Choose rig type              (H2)
## Step 3 — Review and export            (H2)

## Optional: watch the video             (H2)
[Embed YouTube — 只有视频内容真的对应步骤时才插]

## FAQ                                   (H2, +FAQPage schema)
- How long does rigging take?
- Can I edit the rig manually?

## Related tutorials                     (H2)
[3 cards]

## CTA footer
"Try this in Workspace" / "View API reference"
─────────────────────────────────────────────
```

### 5.2 主题页（如 `/tutorials/image-to-3d`）

```
Breadcrumbs:  Home > Tutorials > Image to 3D
H1: Image to 3D tutorials
Intro (定位 + 受众 + 能做出什么)
[Featured tutorial — 大卡片]

## Start here
[3 入门卡]

## Step-by-step guides
[卡片列表，按难度 / 主题分组]

## Video walkthroughs
[YouTube 卡片，可选区块]

## FAQ
[5–8 个常见问题]

## Continue with
[跨主题链接：Texturing / Rigging / Export to Blender]
```

---

## 6. Schema 选型规则

| 内容形态 | 主 schema | 辅助 schema |
|---|---|---|
| 图文 step-by-step，无视频 / 视频次要 | **HowTo** | BreadcrumbList, FAQPage |
| 视频主导，文字仅作摘要 | **VideoObject** | HowTo（含 step 时间轴）, BreadcrumbList |
| 科普长文 / 行业内容 | **Article** / **BlogPosting** | FAQPage |
| 主题页 / 列表页 | **CollectionPage** | BreadcrumbList |
| 全站 | Organization, SiteNavigationElement | — |

**原则**：
- VideoObject 不是越多越好。Google 视频富摘要要求**页面主体内容就是视频**，否则不予展示，反而稀释 HowTo 蓝链。
- HowTo 在搜索面板的展示位优于普通 Article，强 how-to 词必须用 HowTo。
- FAQPage 不要塞与正文无关的 FAQ，否则可能被判作 spammy 而失去富摘要。

---

## 7. 内容生产链路

为每个 YouTube 视频产出一个配套图文页：

```
YouTube video
  │
  ▼
自动转写 (Whisper / YouTube transcript)
  │
  ▼
LLM 整理 → step 结构 + 提取关键截图时间点
  │
  ▼
人工校对 → 配图 / 改写 / 加 prerequisites + FAQ
  │
  ▼
发布到 /tutorials/<slug>          ← 扁平 URL；category 仅作元数据
  │
  ├─ HowTo schema
  ├─ BreadcrumbList schema（中段 "Rigging" → /tutorials?category=rigging）
  ├─ 视频作为 "Optional: watch the video" 区块嵌入
  └─ 视频卡反向链回该图文页
```

效果：
- 视频党留在 YouTube，YouTube 自身留存不损失。
- "how to ..." 词路从 Google 落到我们的 HowTo 富摘要页，到产品的直接路径。
- 图文页可独立排版、做内链、加 CTA，转化好得多。

---

## 8. 8 个新 slug 的落地（无需 301）

URL 扁平意味着这 8 篇直接落到 `/tutorials/<slug>`，**slug 沿用你已经定的、不改名**，因此**没有 301 工作量**。category 只是元数据 + breadcrumb 文案：

| Slug（最终 URL） | category（元数据 / breadcrumb 中段） | 主 schema |
|---|---|---|
| `/tutorials/image-to-3d-model-complete-guide` | Image to 3D | HowTo |
| `/tutorials/text-to-3d-model-tutorial` | Text to 3D | HowTo |
| `/tutorials/photo-to-3d-printable-model` | 3D Printing | HowTo |
| `/tutorials/3d-model-for-unity-workflow` | Workflows · Unity | HowTo |
| `/tutorials/character-auto-rigging-workflow` | Rigging | HowTo |
| `/tutorials/pbr-texturing-with-meshy` | Texturing | HowTo |
| `/tutorials/api-quickstart-image-to-3d` | Developers · API | HowTo + 代码块 |
| `/tutorials/export-to-blender-workflow` | Workflows · Blender | HowTo |

每个详情页 breadcrumb 形如：`Home > Tutorials > <category> > <article>`，其中 `<category>` 链接到 `/tutorials?category=...`，叶子 URL 是上表那个扁平形态。

**现有 URL 的处理**：

| URL | 处理 |
|---|---|
| `/tutorials` | 重做内容（左侧栏 + 列表） |
| `/tutorials/3d-printing-academy` | 保留不动，主题独立 |

→ 这 8 个都是可执行教程，**全部进 `/tutorials/<slug>`**，没有去 blog 的。Blog Guides 这个标签可以彻底废弃。

---

## 9. 实施阶段建议

**Phase 0：定方向（本周）**
- Design + SEO 拍板：URL 结构、左侧栏分类、schema 规则。
- 起 10–15 篇优先 tutorial 的 slug 清单。

**Phase 1：知识库骨架（2–3 周）**
- 重做 `/tutorials` 列表：左侧栏 + `?category=` 过滤 + 卡片网格。
- 详情页模板：扁平 URL `/tutorials/<slug>` + breadcrumb + HowTo + FAQ + 视频可选区。
- 8 篇新 slug 上线（沿用现有 slug，无 301）。

**Phase 2：内容回填（持续）**
- 每周 2–3 篇配套图文，优先级按 GSC keyword opportunity 排。
- 老视频反链到新图文页（YouTube description 加链接）。

**Phase 3：监测**
- GSC：HowTo / VideoObject / FAQ 富摘要展现数、CTR。
- 站内：tutorial 详情页到 Workspace CTA 点击率（区分 blog / tutorial 来源）。
- 决策点：3 个月后看 HowTo 富摘要表现，决定是否扩大配套图文产能。

---

## 10. 待确认问题

1. 现有 `/tutorials/3d-printing-academy` 子路由的内容是图文还是纯视频？决定迁移策略。
2. Blog 现在的分类体系是什么？需要新增 `/blog/category/guides` 还是合并到既有类目？
3. 是否有 LLM 内容生产管线可复用？没有的话先手工写 5 篇打样。
4. SEO 团队是否同意把 VideoObject 仅留给视频主导页（短期看视频富摘要数会下降，但 HowTo 应增加）。
5. 左侧栏分类要不要做多语言（中 / 日 / 韩 / 西）？分类名一旦定下不易改。
