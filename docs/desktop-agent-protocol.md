> 本文描述的是桌面代理的通道设计。**代码实现位于主仓库**：
> `backend/src/main/java/com/kingdomstudio/modules/desktop/`（服务端）+ `frontend/src/extensions/music-agent/components/DesktopAgentPanel.vue`（界面）。
> 服务端只生成派发计划，本机代理尚未实现——这是有意划定的安全边界。

# Desktop Agent 执行协议（v1.0）

> 本文件描述「网页端把按键序列交给本机代理」的通道。
> **当前实现只有服务端一侧：`com.kingdomstudio.modules.desktop` 返回的是派发计划，不会产生任何真实的键盘 / 鼠标输入。**

## 1. 角色与边界

| 角色 | 职责 |
| --- | --- |
| Kingdom Studio（服务端 + 网页） | 解析曲子 → 映射按键 → 生成命令流 → 下发 → 显示进度 |
| Desktop Agent（本机程序，**尚未实现**） | 接收命令流 → 在本地计时 → 执行按键 → 回报进度与错误 |

边界（写进代码，不只是写在这里）：

1. 服务端**不接受前端直接提交的按键列表**。派发请求只带 `taskId` + `profileId`，按键由服务端按乐器档案重新计算。
2. 服务端**不执行输入**。命令流只作为响应体返回，`mode` 字段固定为 `MOCK`。
3. 真实执行前必须满足：明确的单人使用场景、可见的急停手段、只对前台窗口生效。

## 2. 通道

- 传输：WebSocket
- 地址：`ws://localhost:8080/api/desktop/ws`（网关 / 反向代理后同路径）
- 编码：UTF-8 JSON 文本帧
- 心跳：客户端每 15 秒发 `{"type":"PING"}`，代理回 `{"type":"PONG"}`

## 3. 消息信封

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `type` | string | 消息类型 |
| `sessionId` | string | 会话 id，由服务端在派发时生成 |
| `atMs` | number | **相对曲首**的毫秒时间；代理按本地时钟计，不依赖服务器时钟 |
| `payload` | object | 具体内容，随 `type` 变化 |

## 4. 消息类型

### 4.1 `AGENT_HELLO`（代理 → 服务端）

代理上线自报家门，服务端据此决定是否允许派发。

```json
{
  "type": "AGENT_HELLO",
  "payload": {
    "version": "1.0",
    "os": "Windows 11",
    "canInjectInput": true
  }
}
```

### 4.2 `SESSION_OPEN`（服务端 → 代理）

一次演奏会话，携带**完整**命令流（不做增量下发：中途补齐反而容易错位）。

```json
{
  "type": "SESSION_OPEN",
  "sessionId": "mock-3f9c1a2b",
  "payload": {
    "taskName": "小星星",
    "profileName": "光遇式 15 键",
    "durationMs": 10000,
    "commands": [
      { "seq": 1, "key": "Z", "action": "PRESS",   "atMs": 0,   "holdMs": 625 },
      { "seq": 2, "key": "Z", "action": "RELEASE", "atMs": 625, "holdMs": null }
    ]
  }
}
```

`action` 只有两个值：`PRESS`（按下）/ `RELEASE`（松开）。每条命令都成对出现，代理不需要推断时长。

### 4.3 `SESSION_TICK`（代理 → 服务端）

每完成一批命令回报一次，用于界面同步播放位置。

```json
{ "type": "SESSION_TICK", "sessionId": "mock-3f9c1a2b",
  "payload": { "done": 12, "total": 40, "atMs": 6000 } }
```

### 4.4 `SESSION_ABORT`（双向）

服务端可以随时中止；代理在急停时也发这条。**收到后代理必须立刻松开所有已按下的键并在回执里列出松开的键。**

```json
{ "type": "SESSION_ABORT", "sessionId": "mock-3f9c1a2b",
  "payload": { "reason": "user-cancel", "released": ["Z"] } }
```

### 4.5 `AGENT_ERROR`（代理 → 服务端）

| code | 含义 |
| --- | --- |
| `NO_INPUT_PERMISSION` | 代理没有输入权限（未授权 / 被安全软件拦截） |
| `UNSUPPORTED_KEY` | 收到协议外的键（协议只传单键） |
| `SESSION_NOT_FOUND` | `sessionId` 不匹配 |
| `ABORTED_BY_USER` | 被本地急停中止 |

## 5. 服务端在派发前已经做的校验

命令流到达代理之前，服务端会把不可行的部分拦下或修正，并在 `warnings` 里逐条说明（见 `POST /api/desktop/dispatch`）：

| 规则 | 处理 |
| --- | --- |
| 同一时刻的多个键 | 各自成对发送，时间点相同（和弦） |
| 同一个键未松开又要按下 | 顺延到上一次松开之后 60ms，并说明 |
| 按住时长 < 40ms | 按 40ms 发送，并说明（否则等于没按） |
| 按住时长 > 2000ms | 截断到 2000ms，并说明 |
| 多字符的键（如 `F5`） | 跳过，并说明（协议只传单键） |
| 命令总数 > 4000 条 | 直接拒绝，要求先裁短曲子 |
| 一个音都落不下键 | 拒绝派发，要求先换档案或改策略 |

## 6. 真实执行前必须补上的事

1. **急停**：代理本地监听 ESC，按住即中止并松开所有键；服务端也保留 `SESSION_ABORT`。
2. **可见性**：界面全程显示「演奏进行中」，并且只有用户明确点击才开始。
3. **前台限定**：只对当前前台目标窗口生效，禁止后台注入与跨窗口广播。
4. **权限**：代理首次运行必须显式申请输入权限，被拒时发 `NO_INPUT_PERMISSION` 而不是降级尝试。
5. **日志**：只记录按键与时间戳，不记录任何其他用户输入内容。

## 7. 相关代码

| 位置 | 说明 |
| --- | --- |
| `modules/desktop/service/DesktopAgentService.java` | 命令流生成与安全规则（纯函数式，已被单测覆盖） |
| `modules/desktop/controller/DesktopAgentController.java` | `GET /api/desktop/ping`、`GET /api/desktop/protocol`、`POST /api/desktop/dispatch` |
| `modules/music/service/InstrumentMappingService.java` | 按键序列的唯一来源 |
| `frontend/src/extensions/music-agent/components/DesktopAgentPanel.vue` | 界面上的「派发计划（模拟）」面板 |
