# Kingdom Extensions · 动效基因库 & 音乐 Agent

> 两个给个人开发者工作台做的扩展模块：一个负责**收集与再造网页动效**，一个负责**把曲子翻译成乐器按键**。
> 它们与主站（项目王国 / 技术图鉴 / 成长时间线 / 代码知识库）相互独立，各自有一套完整的数据库表、后端接口与前端页面。

| | 动效基因库 / Motion Lab | 音乐 Agent / Music Agent |
| --- | --- | --- |
| 一句话 | 动效资源的采集、分类、去重、编辑与多语言代码生成 | MIDI / 简谱解析 → 乐器按键映射 → 演奏时间线 → 回放 |
| 界面 | 三栏工具台：分类树 + 资源列表 / 实时预览 + 代码面板 | 三栏工作台：输入与曲库 / 时间线与虚拟键盘 / 乐器档案与导出 |
| 技术要点 | Redis 缓存采集、三级去重、沙箱预览、模板化代码生成 | `javax.sound.midi` 纯 Java 解析、音阶/半音两套键位展开、WebAudio 回放 |
| 数据表 | `motion_resource`、`motion_code` | `music_task`、`music_note`、`instrument_profile` |
| 后端接口 | 10 个（含采集） | 12 个（含映射与乐器档案） |
| 单测 | 26 个 | 34 个 |

代码在本机工作台的仓库里，本仓库负责**讲清楚这两个模块是什么、怎么设计的、怎么验证的**：

- 后端模块：<https://github.com/Dongxibie/kingdom-studio/tree/master/backend/src/main/java/com/kingdomstudio/modules>
- 前端模块：<https://github.com/Dongxibie/kingdom-studio/tree/master/frontend/src/extensions>
- 建表与种子：<https://github.com/Dongxibie/kingdom-studio/tree/master/db>

---

## 截图

**动效基因库**：左侧分类与资源列表，中间沙箱实时预览（图中是正在播放的骨架屏微光），右侧五类代码产物与模块能力。

![动效基因库](screenshots/motion-lab.png)

**按关键词采集 GitHub**：默认只试算不写库；统计里逐项区分「扫描 / 新增 / 重复来源 / 重复指纹 / 文本相似」，每个关键词的执行情况单独列在下方。

![采集](screenshots/motion-lab-crawl.png)

**音乐 Agent**：曲库、钢琴卷帘时间线、按键轨、虚拟键盘与按键序列导出。

![音乐 Agent](screenshots/music-agent.png)

**桌面代理（模拟派发）**：把按键序列翻译成「按下 / 松开」成对的命令流，被规则调整或拦下的地方逐条说明。

![桌面代理](screenshots/desktop-agent.png)

---

## 三条可以现场演示的链路

### 1 · 采集动效并生成代码

在动效基因库点「采集 GitHub」→ 选关键词 → 先试算：系统会按关键词打分类、按「仓库地址 → 内容指纹 → README 文本相似」三级去重，并告诉你每一项被跳过还是新增；关掉试算开关再执行才会写库。入库后在右侧「AI 产出」里可以生成 Prompt / Vue 3 / React / CSS / Three.js 五种产物并保存。

采集器有三条纪律，都写在代码里并有单测兜着：

1. **不疯狂请求**：关键词之间固定间隔；响应头 `X-RateLimit-Remaining: 0` 或 HTTP 403/429 时立刻停下并如实说明，不继续撞限流。
2. **重试只针对值得重试的**：5xx 与网络异常退避 1s / 2s 重试；403/429 与其它 4xx 直接失败，不做无意义重试。
3. **缓存**：同一关键词的原始响应在 Redis 缓存 30 分钟，重复点采集不会重复打接口。

### 2 · 上传 MIDI 或粘贴简谱，得到按键序列

上传 `demo/scale.mid`（C 大调音阶）或 `demo/twinkle.mid`（小星星），也可以直接粘贴简谱：

```
1=C 4/4 BPM=96
1 1 5 5 | 6 6 5 - | 4 4 3 3 | 2 2 1 -
```

解析后会得到音符时间线、按键轨与一份可直接复制去练的按键序列文本。同一首小星星走 MIDI 与走简谱两条入口，解析结果完全一致（14 个音、96 BPM、10000ms、音域 C4–A4）——两条解析链路互相印证。

### 3 · 切换乐器档案，看同一首曲子怎么变

内置三套档案，同一首 C 大调音阶在三套档案上的结果是设计好的：

| 档案 | 键数 | 结果 |
| --- | --- | --- |
| 光遇式 15 键（音阶排列） | 15 | 8 个音全部落键、无需任何调整 |
| 键盘式 12 键（半音排列） | 12 | 7 个落键 + 1 个未落键（C5 超出上限，策略为跳过） |
| 八音盒 8 键（移八度，C4–C5） | 8 | 8 个音全部落键，超范围的音自动挪八度 |

点「Demo 回放」时，时间线走针、音符高亮与虚拟键盘会同步——声音是网页内用 WebAudio 合成的，**不会驱动任何系统输入**。

---

## 设计上最值得说的三件事

**一、按键映射不是「音高减基准音高」**

只有白键的乐器上，C 后面直接是 D，没有 C#；而键盘式排列相邻键差半音。同一个音高在两套乐器上落在第几个键，答案完全不同。所以映射引擎按「半音排列 / 音阶排列 / 自定义」三种方式**先展开一张键位表**（第几个键发什么音），再拿音符去查，而不是用音高之差当按键下标。超出音域时也不是悄悄丢音符，而是按档案指定的策略处理（跳过 / 就近落键 / 移八度），每一种都会在导出文本里写明理由。

**二、预览是沙箱，不是「在页面里跑用户的代码」**

动效预览走 `sandbox="allow-scripts"` 的 iframe（**刻意不给 `allow-same-origin`**）：即使贴进来的 CSS/JS 有问题，它也拿不到宿主页面的 DOM、Cookie 与本地存储。整个前端仓库没有 `eval`、`new Function`、`v-html`。

**三、Desktop Agent 只到协议为止**

「让程序替你去按键」是所有功能里风险最高的一个。所以这一步只做到**协议 + 命令流**：把按键序列翻译成按下/松开成对、带相对时间的命令，并在发送前用写死在代码里的规则校验（同一个键没松开不能再次按下、单次按住不得短于 40ms 或长于 2000ms、协议只传单键、命令总数上限 4000 条），被调整或拦下的地方逐条说明。真实执行留到具备急停与前台窗口限定的前提下再谈。协议见 [docs/desktop-agent-protocol.md](docs/desktop-agent-protocol.md)。

---

## 本地运行

两个模块跑在本机工作台里（需要 MySQL 8.0.16+、Redis、JDK 21、Node 18+）。

下面这些命令都在**主仓库**里执行（本仓库只有文档与演示素材，不含源码）：

```bash
git clone https://github.com/Dongxibie/kingdom-studio && cd kingdom-studio

# 1. 建库建表（按顺序，扩展表依赖主库已存在）
mysql -uroot -proot < db/kingdom_studio.sql              # 主站五张表
mysql -uroot -proot kingdom_studio < db/extensions_motion.sql
mysql -uroot -proot kingdom_studio < db/extensions_motion_seed.sql
mysql -uroot -proot kingdom_studio < db/extensions_music.sql

# 2. 后端（context-path = /api，端口 8080）
cd backend && mvn spring-boot:run

# 3. 前端（端口 5173）
cd ../frontend && npm install && npm run dev
```

打开 <http://localhost:5173> → 左侧菜单「扩展」→ 动效基因库 / 音乐 Agent。
接口文档：<http://localhost:8080/api/swagger-ui.html>（扩展模块对应 10 / 11 / 12 开头的四组标签）。

可选：设置环境变量 `GITHUB_TOKEN` 后，GitHub 采集的限额会从每小时 10 次提升到 30 次；不设置也能用，只是容易触发限流（触发时会立刻停下并告诉你原因）。

---

## 目录

```
kingdom-extensions/
├── docs/
│   ├── motion-lab.md              动效基因库：架构、采集、去重、代码生成、数据模型
│   ├── music-agent.md             音乐 Agent：解析、映射引擎、时间线、回放、数据模型
│   └── desktop-agent-protocol.md  桌面代理执行协议（WebSocket 消息与安全约束）
├── demo/
│   ├── scale.mid                  C 大调音阶（120 BPM，8 个音，C4–C5）
│   ├── twinkle.mid                小星星（96 BPM，14 个音，C4–A4）
│   └── README.md                  两份演示曲目怎么用、能看出什么
└── screenshots/                   四个界面的截图
```

## 许可

文档、演示曲目与截图以 MIT 许可发布，见 [LICENSE](LICENSE)。
界面与代码的著作权归 [Dongxibie](https://github.com/Dongxibie)。
