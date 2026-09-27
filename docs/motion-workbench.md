# 动效工作台 / Motion Workbench（动效基因库 v1.1.0）

> 把「收集来的动效」变成「能按需求挑出来、能调、能直接组一套」的模板库：
> 60 个模板（官方 30 + 社区 30）+ 30 套组合方案（119 步）+ 一个模型优先、失败会如实回退的动效助手
> + 一条把一句话需求拆成五个轴、逐轴给理由的智能推荐链路。

![动效工作台](screenshots/motion-workbench.png)

## 它比「资源管理器」多了什么

前一版（见 [motion-lab.md](motion-lab.md)）解决的是**采集与管理**：动效从哪来、属于哪类、有没有重复、代码产物是什么。
工作台解决的是**挑与用**：手上有个具体需求（"苹果官网风格的 Hero，要高级感"），能不能三步内拿到一份能跑的方案。

| | 采集层（v1.0） | 工作台（v1.1.0） |
| --- | --- | --- |
| 数据 | `motion_resource`（采集来的资源） | `motion_template`（官方模板，含预览结构 + 五类产物 + 可调参数） |
| 单位 | 一条资源 | 一个模板，或一套把多个模板串起来的**组合方案** |
| 入口 | 分类树 + 列表 | 一句话描述 → 助手挑模板 → 组方案 → 调参 → 出代码 |
| 判断依据 | 分类与标签 | 四项子分算出的**推荐指数** + **运行档位**（性能成本） |

## 推荐指数：一个算法，四个子分

`推荐指数 = 视觉 30% + 代码质量 25% + 复用价值 25% + 性能表现 20%`，四舍五入到整数，再折算成半星与 S/A/B/C 等级（S 90+ / A 80+ / B 70+ / C 其余）。

关键取舍：**库里存的那一列只是缓存**，列表与详情都走同一个算法重算。否则哪天调了权重，老数据会带着旧口径继续显示，
同一屏里出现两套标准。卡片上因此直接摊开三项——视觉、性能、难度——让人看见分是怎么来的，而不是只给一个总分。

## 发现方式：六个分面 + 组合方案

左栏把「怎么找到我想要的」拆成六个可单击的分面，每项后面是实时数量：

左栏的顺序是**推荐 → 运行性能 → 内容维度**：先看推荐组合，再按性能成本收窄，最后才按内容细分——
运行档位排在场景、风格前面，是因为它决定这个动效能不能用在你手上这台设备上。

| 分面 | 取值 |
| --- | --- |
| 推荐组合方案 | 5 套现成方案（Luxury Product Card / AI SaaS Landing / Premium Hero / Portfolio Opening / Dashboard Boot） |
| 按运行性能 | 轻量 / 均衡 / 依赖 GPU 加速 |
| 按使用场景 | 网站首页 / 个人主页 / AI 产品页 / 数据后台 / 游戏界面 / 登录页面 |
| 按视觉风格 | 极简 / 高级 / 赛博 / 玻璃 / 有机 |
| 按技术类型 | CSS / GSAP / Framer Motion / Three.js / Canvas |
| 按模板分组 | 基础交互 / 产品页面 / 高级效果（各 10 个） |
| 按难度 | 入门 / 进阶 / 高阶 |

分面是可叠加的（场景 + 风格 + 关键词一起筛），再点一次取消，也可以一键清除。**组合方案**把几个模板按应用顺序串成场景级方案
（如 `Premium Hero` = 背景 → 内容 → 交互），点开即按第一个成员预览，成员可逐个检查。

## 组合模式：一套方案要被拆开看

单个动效是素材，组合才是「场景级的答案」—— 但组合不能是一句空话，
所以每一套方案都被拆成**按应用顺序排列的步骤**，每一步回答四个问题：

| 列 | 例子 |
| --- | --- |
| 用哪个动画 | 幕布分屏入场 |
| 负责哪一层 | 内容 |
| 作用是什么 | 首屏像幕布一样从中间分开，先给空间再给内容 |
| 代价多大 | 性能 A · 轻量 · 2 个可调参数 |

点某一步，中间的舞台就切到那一步的模板并载入它自己的可调参数；
「依次预览」会按顺序每 4.2 秒自动走一步（走完自动停，手动点步或切换方案也会停），
等于把「这页效果是怎么一层层叠出来的」当场演一遍。

**整体性能成本取最重的一步，不取平均。** 平均值会把「有一处是 GPU 档」这件事抹平，
而用户真正要判断的是「跑到那一步会不会卡」。所以三档各占几步摊开写，
降级建议点名到具体某一步：*「最重的一步是『粒子星网』，移动端可以把它换成同场景的轻量模板，其余步骤照旧。」*

内置 30 套方案覆盖六个场景：网站首页 10 套（Apple Product Page / Luxury Hotel / Cyber Launch …）、
个人主页 6 套（Designer Portfolio / Photographer Gallery …）、AI 产品页 5 套（AI Model Launch / AI Wellness …）、
数据后台 4 套（Analytics Console / Ops Console …）、游戏界面 3 套（Esports Site / Game Loading Screen …）、
登录页面 2 套。性能上 A 级 6 套、B 级 18 套、C 级 6 套 —— C 级都是真的用到三维或 Canvas 的方案，不是随便标的。

30 套方案由已有的 60 个模板组合而成，其中 55 个模板进了至少一套组合：
组合这一层是在**真的复用**已入库的动效，而不是又造一批新素材。

## 智能推荐：一句话拆成五个轴，逐轴给理由

助手解决的是「让模型帮着挑」，智能推荐解决的是「不靠模型也能给出稳定、说得清的结果」。
入口在工作台右上角，页面分三栏：左边说清需求、中间给方案、右边讲理由。

**一句话进来，先被拆成五个可计算的轴**（每一轴都可以是空的 —— 说不清就不猜）：

| 轴 | 取值 | 例 |
| --- | --- | --- |
| 场景 scene | Landing Page / Dashboard / Portfolio / Login / AI SaaS / Game UI | 「科技感**首页**」→ 网站首页 |
| 风格 style | Minimal / Luxury / Cyber / Glass / Organic | 「**苹果**官网风格」→ 极简 |
| 情绪 emotion | premium / tech / calm / playful / warm / bold | 「苹果官网风格」→ 高级感 |
| 性能 performance | LOW 轻量优先 / MEDIUM 均衡 / HIGH 效果优先 | 「**移动端优先**，轻量一点」→ 轻量优先 |
| 触发 interaction | load / hover / scroll / click | 「**滚动**时慢慢出现」→ 滚动驱动 |

词表里的每个词都对应库里真实存在的枚举值；命中痕迹会显示在界面上（"命中：首页"），
所以「为什么被理解成网站首页」是可以当面念出来的，而不是一个说不清的分数。

**排序用的是一张写死的权重表**：

```
推荐分 = 场景匹配 40 + 风格匹配 25 + 触发匹配 18 + 情绪匹配 15 + 性能匹配 12
       + 推荐指数加成（0-10，模板自身推荐指数 ÷ 10）
```

- 不命中的轴**不加分也不扣分** —— 说不清的轴不该把合适的东西压下去；
- 性能轴是唯一的例外：用户明说「轻量优先」时，依赖 GPU 的模板扣 8 分并在理由里写明「已往后放」；
- 性能等级由运行档位与性能子分算出：轻量且子分高是 **A**、均衡是 **B**、依赖 GPU 是 **C**，卡片上直接标出。

**同一句话，任何时候结果都一样**：这条链路不调用模型，没有随机、没有哈希遍历顺序，
所以「同一句输入返回稳定推荐」是被测试钉住的行为，不是一句期望。

**推荐只排序、不隐藏**：不适配的模板只是排到后面，并在理由里说明原因。

## 动效助手：模型优先，但说清楚这次是谁答的

顶部搜索栏是入口。它的行为和有意的克制，都在下面这几条里：

**模型的职责被收窄到四件事**：理解自然语言需求 → 判断风格 → 从**现有模板**里挑 → 组合成一套方案并给参数建议。
它**不生成代码**：代码永远来自模板库里那份已经验证过的实现。于是"模型说得好听但代码跑不起来"这类问题从根上不会出现。

**两道校验，模型说什么不算数**：

1. 模型返回的每个 `templateKey` 都回库核对，不存在就丢弃并记日志；
2. 方案里一个有效成员都没有时，整份方案视为没给。

**回退是明说的，不是假装**。以下情况会回退到内置的可解释检索，并在界面上写明原因：

| 情况 | 界面提示 |
| --- | --- |
| 没配模型（三项配置任一为空） | 「未配置模型（…），当前使用内置检索。」 |
| 超时 / 服务报错 / 返回不是合法 JSON | 「模型本次没有返回可用结果（超时、格式不合法或服务不可用），已改用内置检索。」 |

返回结构里带 `source`（`MODEL` / `RULE`）与 `fallbackReason`，徽章相应显示「模型分析 · 模型名」或「内置检索」——
用户永远知道这次结果是模型给的还是本地检索给的。

**参数建议真的会生效**：模型给的 `--m-duration` 之类的建议值会灌进右侧参数面板并直接作用于预览；
手动换模板时这些覆盖值会被清掉，不让某次建议悄悄影响别的模板。

**模型可替换**：只要服务商兼容 OpenAI `/chat/completions`，改三个环境变量即可切换，不配就不启用（功能不缺失，只是走内置检索）。

```bash
MOTION_LLM_BASE_URL=https://api.deepseek.com/v1
MOTION_LLM_API_KEY=***
MOTION_LLM_MODEL=deepseek-chat
```

## 运行档位：别把高级效果藏起来，也别让人踩坑

同一个"好看"，代价可能差一个数量级。所以每个模板带一个运行档位，判据简单到可以解释：

| 档位 | 判据 | 典型例子 |
| --- | --- | --- |
| 轻量 | 纯 CSS 且性能子分 ≥ 92 | 柔和淡入、向上滑入（transform / opacity 合成属性动画） |
| 均衡 | 纯 CSS 且性能子分 80–91 | 玻璃卡片悬停、极光背景（模糊 / 离屏合成 / 大面积动画） |
| 依赖 GPU 加速 | Three.js、Canvas，或 CSS 但性能子分 < 80 | 三维场景、着色器波纹、粒子星网 |

档位在三处出现：卡片上的徽章、左栏「按运行性能」筛选、右栏的运行建议（适合什么设备、移动端怎么调）。
文案一律正面陈述：轻量讲「几乎无性能代价」，均衡讲「桌面端无压力」，GPU 档讲「高级视觉效果：依赖 GPU 加速，推荐在桌面设备上查看与使用」——不写成硬件不够用。
定位是**标注而不是限制**：30 个模板的分布是 11 轻量 / 16 均衡 / 3 依赖 GPU 加速，高级效果留在库里，用的人自己权衡。

## 参数是数据，不是代码

模板的可调项存在 `params` JSON 里——就是 CSS 变量名与取值范围：

```json
[{ "key": "--m-duration", "label": "时长", "unit": "s", "min": 0.3, "max": 3, "step": 0.1, "default": 0.9 }]
```

接口把它直接吐给前端生成滑块，改完立刻作用到沙箱预览里；导出的 Vue / React / CSS 代码用的是同一批变量。
想给模板加一个可调项，不需要动前后端任何代码，改数据即可。

## 数据模型

```
motion_template ──┬── 被 motion_recipe.member_keys 引用（JSON 数组，按应用顺序）
                  └── 被 motion_rating 打分（target_type = TEMPLATE / RECIPE）
```

- `motion_template`：`template_key`（稳定标识，前端与方案引用它）、分组 / 场景 / 风格 / 技术 / 难度、四项子分、
  `runtime_tier` 与 `runtime_note`、`params`（可调参数 JSON）、`preview_html` / `preview_js`（沙箱预览用）、五类代码产物
- `motion_recipe`：`recipe_key`、名称、场景、`member_keys`、说明与适用场景
- `motion_rating`：人对模板或方案的人工评分与理由（推荐指数由算法给出，人工分是另一个字段，两者不混）

建表脚本 `db/extensions_motion_template.sql` **可以重复执行**：建表用 `CREATE TABLE IF NOT EXISTS`，
老库缺的列（`runtime_tier` / `runtime_note` / `preview_html` / `preview_js`）用 `information_schema` 判断后补，
种子按 `template_key` upsert。脚本开头有 `SET NAMES utf8mb4`，Windows 下 mysql 客户端默认 GBK 也能直接导入。

## 接口

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/extensions/motion/templates` | 模板分页（场景 / 风格 / 技术 / 难度 / 运行档位 / 关键词任意组合 + 排序） |
| GET | `/api/extensions/motion/templates/{templateKey}` | 模板详情（含预览结构、五类产物、参数定义、档位与运行建议） |
| GET | `/api/extensions/motion/templates/facets` | 六个分面的选项与实时数量 |
| POST | `/api/extensions/motion/templates/search` | 内置的可解释检索：识别场景 / 风格 / 技术 / 关键词，逐条给命中理由 |
| POST | `/api/extensions/motion/templates/assist` | 助手（模型优先，失败回退检索；返回 `source` 与 `fallbackReason`） |
| POST | `/api/extensions/motion/templates/recommend` | 智能推荐：一句话 → 五轴意图 → Top N，每条带命中理由、性能等级与运行档位 |
| GET | `/api/extensions/motion/templates/recipes` | 组合方案列表（按场景筛选；每条带步骤与整体性能成本） |
| GET | `/api/extensions/motion/templates/recipes/{recipeKey}` | 组合方案详情：步骤（动画 / 作用 / 参数 / 性能成本）+ 成员 + 推荐指数 |
| POST | `/api/extensions/motion/templates/ratings` | 提交人工评分与理由 |

`GET /templates/{templateKey}` 与 `GET /templates/recipes` 的形状是重叠的，这里依赖 Spring 的匹配规则：
**字面量路径优先于变量路径**，所以 `facets` / `recipes` / `ratings` 不会被当成 `templateKey`。
代价是模板 key 不能取这三个词——种子里的 key 都是 `smooth-fade`、`aurora-bg` 这类短横线命名。

## 测试覆盖

动效模块共 **97 个**单元测试（含 v1.0 的采集层 26 个），加上音乐 Agent 97 个与通用的 13 个，后端合计 **207 个**用例全绿。
其中工作台与推荐链路的部分：

| 测试类 | 用例 | 覆盖点 |
| --- | --- | --- |
| `MotionTemplateServiceTest` | 8 | 推荐指数加权与等级、分面数量、档位判据（含 Three.js / Canvas 必为 GPU 档）、参数解析、详情装配 |
| `MotionAssistantModelTest` | 9 | 模型应答解析（含 ```json 包裹）、非法 JSON / 空响应 / 非 200 → 回退、杜撰的模板 key 被丢弃、模板清单与参数白名单生成 |
| `MotionIntentParserTest` | 18 | 12 个场景的真实说法逐条解析成五轴；触发 / 性能轴单独钉住；命中痕迹与复述；听不懂与空输入不报错；同一句话两次解析完全一致 |
| `MotionRecipeServiceTest` | 9 | 步骤按方案声明顺序装配（不是库里的顺序）、引用了不存在的模板则跳过且编号仍连续、老数据回退到成员清单、坏 JSON 不炸、整体性能取最重档位并点名到那一步、方案分数为成员均值 |
| `MotionRecommendationServiceTest` | 11 | 加权顺序（场景 40 / 风格 25 / 触发 18 / 情绪 15 / 性能 12）、每条推荐都给得出理由、低预算把 GPU 档往后放、性能等级 A/B/C、Top N 收敛、空输入兜底、结果稳定、卡片显示模板自己的标签 |

模型相关的测试用 `MockRestServiceServer` 挂假应答，**不打真实模型**：给一个返回 401 的假服务，测试要断言的是"代码确实回退了"，
而不是"模型能不能用"。所以这套测试在没有密钥的机器上也能全绿——这也是设计目标之一：模型是可选增强，不是必需品。

## 代码位置

- 后端：`backend/src/main/java/com/kingdomstudio/modules/motion/template/`（controller / service / dto / vo / entity / mapper）
- 前端：`frontend/src/extensions/motion-lab/`（views / components / api / types / styles）
- 建表与种子：`db/extensions_motion_template.sql`、`db/extensions_motion_template_seed.sql`
- 采集层（v1.0 的能力，仍在）：`backend/.../modules/motion/crawler/` 与 `service/MotionCrawlerService.java`
