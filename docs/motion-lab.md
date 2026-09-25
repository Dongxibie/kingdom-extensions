# 动效基因库 / Motion Lab

> 收集网页动效、给它们分类去重、在沙箱里预览效果、并生成可复用的多语言代码。

## 这个模块解决什么问题

看到好动效的常规做法是「收藏一堆链接」，然后链接失效、忘了它长什么样、也说不清它到底用了什么技术。
这个模块把动效当成**可管理的资源**：有分类、有来源、有许可、有可执行的代码产物，能搜、能筛、能预览、能一键生成四种技术栈的写法。

## 一次采集发生了什么

```
关键词 ──▶ GitHub 搜索 API ──▶ 逐条分类 ──▶ 三级去重 ──▶ 写库（DRAFT / READY）
              │                    │            │
              │                    │            └─ 仓库地址 → 内容指纹 → README 文本相似
              │                    └─ 关键词权重打分：命中哪些词、得了多少分都留在结果里
              └─ Redis 缓存 30 分钟；关键词之间固定间隔；限流立刻停
```

### 分类

按关键词权重打分（如 `hover` 3 分、`mouseenter` 2 分），十个分类各有一张权重表。
命中不了任何关键词时**落 `DRAFT`**（草稿）交给人确认分类，而不是硬塞一个看起来像的分类——采进来的东西宁可是"待定"，也不要给一个错误的标签。

### 三级去重

| 级别 | 判据 | 拦下什么 |
| --- | --- | --- |
| 一级 | 仓库地址（归一化大小写与末尾斜杠） | 同一个仓库被不同关键词重复搜到 |
| 二级 | 内容指纹 `SHA1(名称\|来源\|分类)` + 数据库唯一键 | 改名/改来源之外的重复入库 |
| 三级 | README 文本相似度（3-gram Jaccard + 包含率兜底） | 同一份说明被复制到多个仓库 |

第三级为什么要两套判据：Jaccard 用并集做分母，遇到「README 抄了主干又追加几段」会被稀释到阈值以下；
补一条包含率判据（短的一侧几乎被长的一侧完整覆盖）能兜住这种情况，同时用长度比值下限防止「一句话恰好出现在长文里」被误判。
误判的代价是资源被静默跳过，而漏采比误采更糟——这个取舍写在代码注释里。

### 限流与重试（三条纪律）

1. 每个关键词之间有固定间隔，不并发轰接口。
2. 响应头 `X-RateLimit-Remaining: 0`、HTTP 403、HTTP 429 → **立刻停止**，在结果 `notes` 里说明原因（带重置时间）。
3. 只有 5xx 与网络异常才退避重试（1s / 2s，最多两次）；其余 4xx 直接失败——重试没有意义。

每个关键词的执行情况都会单独回报：`取回 N 条`、`命中缓存`、`触发限流，已停止（…）`、`失败（…）`。前端把这些原文展示出来，不做二次包装。

## 代码产物：五种，可编辑可导出

| 产物 | 用途 |
| --- | --- |
| Prompt | 描述这个动效，便于让模型在其他项目里复现 |
| Vue 3 | 单文件组件（模板按类名搭出结构，CSS 原样进 `<style>`） |
| React | 函数组件 + 样式常量 |
| CSS | 纯 CSS 实现，可直接贴进任何页面 |
| Three.js | 只在「三维」与「粒子」两个分类下生成 |

「生成模板」的做法是有意为之：把资源里的 CSS 类名解析出来，按**动效逻辑**决定演示节点的结构与数量（例如骨架屏要三条、粒子要 60 个），
再把 CSS 原样嵌进去。生成结果与预览里看到的一致，因为两边用的是同一份字符串——所见即所得不是靠对齐两张图，而是靠同一份数据。

## 安全预览

预览走 `sandbox="allow-scripts"` 的 iframe，**不给 `allow-same-origin`**：

- iframe 处于不透明源（opaque origin），拿不到宿主页面的 DOM、Cookie、`localStorage`
- 整个前端没有 `eval` / `new Function` / `v-html`，资源里的说明文字一律按文本渲染
- 预览的"重播"靠重建 iframe，而不是在被预览的页面里注入脚本

## 数据模型

```
motion_resource ──1:1── motion_code
       │                     │
       │                     └─ prompt / vue_code / react_code / css_code / three_code
       └─ name, description, category, technology, source_url, repo_url,
          preview_url, tags, license, code_path, content_hash(唯一), status
```

- `status`：`DRAFT`（采集后待确认）/ `READY`（可用）/ `ARCHIVED`（归档）
- `category`：Entrance、Hover、Scroll、Text、Particle、3D、Glass、Cursor、Background、Loading（数据库 CHECK 约束与 Java 常量、前端枚举三处对齐）
- 逻辑删除：`deleted` 标记，删除资源时同事务删掉它的代码记录

## 接口

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/motion/ping` | 模块自检 |
| GET | `/api/motion/categories` | 分类清单 |
| GET | `/api/motion` | 分页列表（分类 / 技术栈 / 关键词可组合，关键词匹配名称、说明、标签） |
| GET | `/api/motion/{id}` | 详情（含五类代码） |
| POST | `/api/motion` | 新增 |
| PUT | `/api/motion/{id}` | 修改 |
| DELETE | `/api/motion/{id}` | 删除（级联删代码） |
| PUT | `/api/motion/{id}/code` | 保存代码产物（upsert） |
| GET | `/api/motion/crawl/keywords` | 采集用的默认关键词 |
| POST | `/api/motion/crawl` | 执行采集（`dryRun=true` 只试算不写库） |

## 测试覆盖

26 个单元测试，全部不依赖 MySQL 与 Redis：

| 测试类 | 用例 | 覆盖点 |
| --- | --- | --- |
| `MotionServiceTest` | 9 | 分类与状态校验、内容哈希、级联删除、分页收敛、代码 upsert |
| `MotionClassifierTest` | 5 | 关键词权重、命中留痕、未命中降级为草稿 |
| `MotionDedupeTest` | 6 | URL 归一化、哈希稳定、格式差异容忍、包含率与长度比值 |
| `MotionCrawlerServiceTest` | 6 | 试算不写库、限额用尽即停、403 不重试、5xx 重试、422 快速失败 |

采集器的测试用 `MockRestServiceServer` 挂假响应，**不打真实 GitHub**：给第二个关键词不准备期望，如果代码在限流后仍去请求，测试会因此失败——用测试把"不该发的请求"钉死。

## 已知的定位边界

采集层现在是**发现层**：它给出「有哪些仓库值得做」，把元数据、分类、去重与来源整理好；
它不去 clone 仓库、不抓源码、不生成截图。素材层（真实代码与预览图）目前由人工确认后手工补，或者用模板生成器产出。
这条路线的下一步是补 Code Extractor（按 topics/语言挑目标文件取源码）与 Screenshot Generator（无头浏览器出预览图），
并在同一层加许可闸门：采集一律落草稿，人工加工过才升为可用。

## 代码位置

- 后端：`backend/src/main/java/com/kingdomstudio/modules/motion/`（controller / service / crawler / entity / mapper / dto / vo）
- 前端：`frontend/src/extensions/motion-lab/`（views / components / api / types / utils）
- 建表与种子：`db/extensions_motion.sql`、`db/extensions_motion_seed.sql`
