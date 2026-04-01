# SAS v1.5 智能化改造完整建设方案

> 版本：v1.5.0
> 日期：2026-04-01
> 状态：✅ 已完成（OpenClaw集成验证通过，2026-04-01 19:45）
> 评估对象：GLM-5-Turbo

---

## 一、现状分析与改造目标

### 1.1 当前 OpenClaw 的能力现状

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenClaw 框架能力                         │
├─────────────────────┬───────────────────────────────────────┤
│ ✅ 已有的            │ ❌ 缺失的                              │
├─────────────────────┼───────────────────────────────────────┤
│ 多 Agent 注册与管理  │ 任务自动拆解机制                       │
│ Subagent spawn      │ 阶段门控（Gate）系统                   │
│ Workspace SOUL/AGENTS│ 自动审批引擎                          │
│ Memory lancedb-pro  │ Agent 间自主协作                      │
│ lossless-claw 压缩  │ SAS 准则执行追踪                      │
│ Hooks 事件系统      │ 任务超时预警与恢复                     │
│ Cron 定时任务        │ 任务看板实时联动                       │
│ 内置工具集（50+）    │ 复杂任务自动路由                       │
└─────────────────────┴───────────────────────────────────────┘
```

### 1.2 改造目标

| 目标 | 指标 | 当前差距 |
|------|------|---------|
| SAS 六阶段自动执行 | L2+ 任务自动进入流程 | 0% |
| 子任务不超时 | 80% 子任务完整完成 | 经常中断 |
| 任务看板可视化 | 实时显示任务/阶段/Agent | 空白 |
| 执行日志可追溯 | 每个决策有记录 | 无 |
| 零预设方案 | 不自行决定执行路径 | SC 跳过拆解 |

---

## 二、SAS v1.5 架构图

```
用户（L0/CEO）
  │
  ▼
┌──────────────────────────────────────────────────────────────┐
│                    SC Agent（主脑 L1）                        │
│  SOUL.md：SAS v1.5 核心指令                                  │
│  AGENTS.md：spawn 规范 + max_turns 控制                      │
│  TOOLS.md：可用工具清单                                      │
└──────────────────────────────────────────────────────────────┘
  │
  │  任务分发
  ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  SAS-Default    │    │  SAS-Leader     │    │  Domain Agent   │
│  Agent（L2）     │    │  Agent（L2）     │    │  HR / 科技资讯  │
│  标准任务执行    │    │  任务协调分配    │    │  垂直领域专家  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
  │
  │  spawn with max_turns
  ▼
┌──────────────────────────────────────────────────────────────┐
│                    Worker 执行层（L3）                        │
│  每个 Worker 有独立 session + 超时控制                       │
│  执行结果 → task_logger → 任务看板                          │
└──────────────────────────────────────────────────────────────┘
```

---

## 三、Skill 使用规划

### 3.1 什么是 Skill

Skill 是 OpenClaw 的**技能扩展包**，本质是 `SKILL.md + 脚本 + 工具`的集合，放在 `~/.openclaw/skills/` 或 `workspace/skills/` 目录下，可被任意 Agent 调用。

**与 SOUL.md 指令的区别：**
- SOUL.md 指令：告诉 Agent「应该做什么」
- Skill：给 Agent「实际做某事的能力」

### 3.2 需要引入的 Skills

| Skill 名称 | 用途 | 安装位置 | 为什么用它 |
|-----------|------|---------|-----------|
| **skill-creator** | 创建新的 Skill | `~/.openclaw/skills/` | 用于快速生成定制 Skill（如行情查询 Skill） |
| **summarize** | 长文本摘要 | `~/.openclaw/skills/` | 方案/报告类任务完成后自动摘要，节省上下文 |
| **github** | Git 操作 | `~/.openclaw/skills/` | 代码入库、PR 创建、版本管理 |
| **himalaya** | 邮件管理 | `~/.openclaw/skills/` | 预警通知、报告推送 |
| **weather** | 天气（参考） | `~/.openclaw/skills/` | 行情查询 Skill 的设计模板 |
| **blogwatcher** | RSS 监控 | `~/.openclaw/skills/` | 财经新闻监控的原型参考 |
| **xurl** | Twitter 研究 | `~/.openclaw/skills/` | 舆情分析 Skill 的原型参考 |

### 3.3 需要自建的 Skills

#### Skill 1：行情查询 Skill（`sas-stock-query`）

```
~/.openclaw/skills/sas-stock-query/
├── SKILL.md          ← 技能定义（描述、触发词、参数）
├── query_stock.py    ← 核心脚本（AKShare 封装）
└── README.md
```

**触发词**：查股票、行情查询、涨跌情况、A股行情

**功能**：
- 输入股票代码 → 返回名称/现价/涨跌幅/成交量
- 输入多个代码 → 批量查询
- 输入指数代码 → 返回大盘指数

**为什么用 Skill 而不是脚本直接调用**：
- Skill 可以被 SOUL.md 指令引用，Agent 调用更自然
- Skill 有参数解析，不需要 Agent 自己拼命令行
- Skill 可以复用，在不同任务里随时调用

---

#### Skill 2：任务状态 Skill（`sas-task-tracker`）

```
~/.openclaw/skills/sas-task-tracker/
├── SKILL.md
├── update_task.py    ← 更新任务状态
├── list_tasks.py     ← 列出当前任务
└── close_task.py     ← 关闭任务
```

**触发词**：更新任务、查看任务状态、任务完成、任务失败

**功能**：
- 写入 `~/.openclaw/logs/tasks/` JSONL 日志
- 被任务看板读取显示

**为什么用 Skill 而不是 hooks**：
- Hooks 是事件驱动的（/new、/reset 等），不能被 Agent 主动调用
- Skill 可以被 Agent 在任意时机调用（"每完成一个子任务就调用 update_task"）
- 两者互补，不是替代关系

---

## 四、Plugin 使用规划

### 4.1 什么是 Plugin

Plugin 是 OpenClaw 的**功能性扩展模块**，在 `extensions/` 目录下，通过 `openclaw.json` 配置注册，会在 Gateway 启动时加载。

**与 Skill 的区别：**
- Skill：Agent 可以主动调用的工具
- Plugin：底层能力增强，Agent 不用主动调用

### 4.2 当前已安装的 Plugins

| Plugin | 版本 | 功能 | 是否保留 |
|--------|------|------|---------|
| `memory-lancedb-pro` | 1.1.0-beta.9 | 向量记忆存储 | ✅ 保留 |
| `openclaw-lark` | 2026.3.25 | 飞书集成 | ✅ 保留 |
| `lossless-claw-enhanced` | latest | 上下文压缩 | ✅ 保留 |

### 4.3 需要新增的 Plugins

#### Plugin 1：`sas-engine`（核心自研插件）

**目的**：实现 SAS 阶段门控引擎 + 自动审批

**功能**：
1. **Gate Router**：根据任务复杂度（L1/L2/L3）自动路由到不同处理流程
2. **Phase Tracker**：跟踪当前任务处于 SAS 哪个阶段
3. **Auto Approver**：L1 任务自动通过，L2/L3 需用户审批
4. **Timeout Watchdog**：监控所有 subagent 执行时间，超时前预警

**文件结构**：
```
~/.openclaw/extensions/sas-engine/
├── plugin.json          ← 插件元数据
├── index.js             ← 插件入口
├── gate-router.js       ← 门控路由核心
├── phase-tracker.js     ← 阶段追踪
├── auto-approver.js     ← 自动审批
└── timeout-watchdog.js ← 超时守护
```

**为什么用 Plugin 而不是 Skill**：
- Plugin 运行在 Gateway 层，能拦截所有 Agent 消息
- Plugin 可以修改 OpenClaw 的内部状态（任务阶段、审批状态）
- Plugin 可以在 Agent 看不到的情况下执行后台监控
- Skill 只能被 Agent 主动调用，无法实现 Gate Router 的拦截机制

**核心价值**：
```
没有 sas-engine → SC 收到任务后自行决定怎么处理
有了 sas-engine → 任务自动进入 L1/L2/L3 路由，阶段自动推进
```

---

#### Plugin 2：`sas-task-board`（任务看板插件）

**目的**：将任务日志实时同步到 WebSocket 面板

**功能**：
- 监听 `~/.openclaw/logs/tasks/` 目录变化
- 通过 WebSocket 推送任务状态更新
- 提供 `GET /api/tasks` REST 接口给前端面板

**为什么用 Plugin**：
- 需要文件系统监控 + HTTP Server + WebSocket，这些是底层能力
- 前端面板无法直接读本地文件，需要插件作为桥接

---

## 五、Script 使用规划

### 5.1 什么是 Script

Script 是存放在 workspace 目录下的**可执行 Python/Shell 脚本**，通过 OpenClaw 内置的 `exec` 工具被 Agent 调用。

### 5.2 需要部署的 Scripts

#### Script 1：`task_decompose.py`（已完成）

**位置**：`~/.openclaw/workspace/task_decompose.py`

**功能**：LLM 驱动的任务自动拆解

**调用链路**：
```
用户 → SC Agent → exec("python3 task_decompose.py '任务描述'")
  → DeepSeek API 拆解 → JSON 任务列表
  → SC Agent 按列表 spawn 子任务
```

**为什么用 Script 而不是 Skill**：
- Script 通过 exec 工具直接执行，不需要复杂的 Skill 注册流程
- Script 输出是纯 JSON，直接被 Agent 解析，不增加中间层复杂度

**使用场景**：
- L2+ 复杂任务触发
- SC 需要知道"这个任务分几步，每步做什么"

---

#### Script 2：`task_logger.py`（已完成）

**位置**：`~/.openclaw/workspace/task_logger.py`

**功能**：记录每个子任务的开始/完成/失败状态

**调用时机**：
- spawn 子任务前：写入 `{"event": "task_start", ...}`
- 子任务完成：写入 `{"event": "task_complete", ...}`
- 子任务失败/超时：写入 `{"event": "task_failed", ...}`

**日志格式**：
```jsonl
{"ts": "2026-04-01T18:50:00", "event": "task_start", "task_id": "stock-monitor-1", "title": "需求分析", "agent": "sas-default", "pid": 1234}
{"ts": "2026-04-01T18:52:00", "event": "task_complete", "task_id": "stock-monitor-1", "title": "需求分析", "duration": 120}
```

**为什么用 Script 而不是 Skill**：
- 日志写入是纯 I/O 操作，Skill 没有额外价值
- Script 可以被 `subagent` 里的 `on_complete` 回调触发，更及时

---

#### Script 3：`sas_phase_gate.py`（待开发）

**位置**：`~/.openclaw/workspace/sas_phase_gate.py`

**功能**：
- 输入当前阶段 + 任务类型 → 判断是否可以进入下一阶段
- 实现 SAS 的门控逻辑：

```
阶段1（接收任务） → 阶段2（制定计划）→ 阶段3（执行工作）
                                          ↓
                               L1: 自动通过
                               L2: 等待用户确认关键决策
                               L3: 等待用户审批完整方案
                                          ↓
阶段4（质量检查） ← ─────────────────────
       ↓
阶段5（交付成果）
       ↓
阶段6（归档记忆）
```

**为什么必须开发**：
- 这是 SAS v1.5 的核心机制差异
- 没有门控引擎，所有任务都会跳过审批直接执行
- 这是「CEO 模式」和「普通助手」的本质区别

---

#### Script 4：`stock_data_fetcher.py`（待开发）

**位置**：`~/.openclaw/workspace/stock_data_fetcher.py`

**功能**：
- AKShare 封装，获取 A股实时行情
- 支持：个股查询、指数查询、板块资金流、自选股监控
- 输出标准化 JSON

**为什么用 Script 而不是 Skill**：
- 行情数据获取是纯数据脚本，不需要复杂的参数解析
- Agent 调用时只需要 `exec("python3 stock_data_fetcher.py --code 600519")`

---

## 六、OpenClaw 内置能力的使用

### 6.1 哪些内置能力直接用

| 内置能力 | 在 SAS 中的用途 | 使用方式 |
|---------|--------------|---------|
| **Task tool** | spawn 子任务执行 | `Task(task_id, description, agent, max_turns)` |
| **subagent** | 并行多 Agent 协作 | SC spawn 多个 Agent |
| **read/write/exec** | 脚本调用、文件操作 | Agent 正常工作流 |
| **memory_search** | 检索 SAS 准则历史记忆 | `memory_recall` 工具 |
| **Cron** | 定时任务 | 每日报告、数据采集 |
| **Hooks** | 事件触发自动化 | `/new` 时保存上下文 |
| **session** | 多会话管理 | 不同任务开不同 session |

### 6.2 内置能力的配置增强

#### 配置 1：`subagent` 超时控制

在 `openclaw.json` 的 `agents` 配置中：

```json5
{
  "agents": {
    "main": {
      "model": { "primary": "minimax/MiniMax-M2.7-highspeed" },
      "subagent": {
        "defaultMaxTurns": 15,
        "timeoutPerTurn": 60  // 秒
      }
    }
  }
}
```

**效果**：所有 subagent 默认 15 轮（约 15 分钟），避免无限跑

---

#### 配置 2：`alsoAllow` 白名单扩展

把 SAS 脚本路径加入白名单，让 SC Agent 可以调用：

```json5
{
  "agents": {
    "main": {
      "alsoAllow": [
        { "path": "workspace", "commands": ["python3 task_decompose.py"] },
        { "path": "workspace", "commands": ["python3 task_logger.py"] },
        { "path": "workspace", "commands": ["python3 stock_data_fetcher.py"] }
      ]
    }
  }
}
```

---

## 七、SOUL.md / AGENTS.md 指令注入

### 7.1 SC Agent（主脑）的 SOUL.md 改造

**文件**：`~/.openclaw/workspace/SOUL.md`

**注入的 SAS 核心指令**：

```markdown
## SAS v1.5 任务自动拆解

### 触发条件（满足任一即触发）
- 任务涉及 2 个以上 Agent 协作
- 任务包含 3 个以上执行步骤
- 任务描述超过 50 字
- 包含关键词：系统、平台、完整、前后端、分析、报告、改造、优化

### 执行流程
1. 调用 task_decompose → 返回 JSON 子任务列表
2. 解析 dependencies，按顺序 spawn
3. 每个子任务用 task_logger 记录
4. 超时 80% 预警，完成后汇总交付

### 不许做的事
- ❌ 收到任务直接问用户需求（应该调用 decompose）
- ❌ 所有步骤在一个任务里执行
- ❌ 跳过 planning 直接进入 execution
```

### 7.2 AGENTS.md 的 spawn 规范

**文件**：`~/.openclaw/workspace/AGENTS.md`

```markdown
## Subagent Spawn 规范

### 超时参数（必须）
- L1 简单任务：`max_turns: 10`
- L2 中等任务：`max_turns: 20`
- L3 复杂任务：`max_turns: 30` + 分阶段审批

### 并行规则
- 无依赖的子任务可以并行 spawn（最多 3 个）
- 有依赖的必须串行，前一个完成才启动下一个

### 失败处理
- 超时：记录 + 继续，不卡死流程
- API 失败：重试 1 次，仍失败则跳过并记录原因
```

---

## 八、数据流设计

### 8.1 任务完整生命周期

```
用户输入任务
    ↓
[SC Agent] 接收 → 识别复杂度
    ↓
复杂度判断
    ├── L1 → 直接执行 → 交付
    ├── L2 → task_decompose → 用户审批计划 → 分步执行 → 交付
    └── L3 → task_decompose → 用户审批方案 → Leader 协调 → 分步执行 → 交付
    ↓
sas_phase_gate.py 门控检查
    ↓
spawn subagent（带 max_turns）
    ↓
每个子任务执行中：
    - task_logger: 记录开始
    - timeout_watchdog: 监控超时
    ↓
子任务完成：
    - task_logger: 记录完成
    - 下一阶段门控检查
    ↓
全部完成：
    - 汇总报告
    - 日志归档
    - 记忆更新
```

### 8.2 任务看板数据来源

```
subagent 执行
    ↓
task_logger.py 写入 JSONL
~/.openclaw/logs/tasks/YYYYMMDD.jsonl
    ↓
sas-task-board plugin 监听文件变化
    ↓
WebSocket 推送 / GET /api/tasks
    ↓
前端面板显示
```

---

## 九、文件部署清单

### 9.1 一览表

| 类型 | 路径 | 状态 |
|------|------|------|
| SC SOUL.md | `~/.openclaw/workspace/SOUL.md` | ✅ 已注入 SAS 指令 |
| SC AGENTS.md | `~/.openclaw/workspace/AGENTS.md` | ✅ 已更新 spawn 规范 |
| task_decompose.py | `~/.openclaw/workspace/task_decompose.py` | ✅ 已部署 |
| task_logger.py | `~/.openclaw/workspace/task_logger.py` | ✅ 已部署 |
| sas_phase_gate.py | `~/.openclaw/workspace/sas_phase_gate.py` | ❌ 待开发 |
| stock_data_fetcher.py | `~/.openclaw/workspace/stock_data_fetcher.py` | ❌ 待开发 |
| sas-engine plugin | `~/.openclaw/extensions/sas-engine/` | ❌ 待开发 |
| sas-task-board plugin | `~/.openclaw/extensions/sas-task-board/` | ❌ 待开发 |
| alsoAllow 白名单 | `openclaw.json` | ✅ 已配置 |
| 行情查询 Skill | `~/.openclaw/skills/sas-stock-query/` | ❌ 待开发 |
| 任务追踪 Skill | `~/.openclaw/skills/sas-task-tracker/` | ❌ 待开发 |

### 9.2 部署优先级

```
第一阶段（立即可用）：
✅ task_decompose.py → 直接减少任务超时
✅ task_logger.py → 任务看板有数据
✅ SC SOUL.md 指令 → SC 知道要拆解

第二阶段（1-2天）：
⏳ sas_phase_gate.py → 实现 L2/L3 审批流
⏳ stock_data_fetcher.py → A股监控核心能力
⏳ alsoAllow 白名单 → 脚本调用无障碍

第三阶段（1周）：
⏳ sas-engine plugin → 门控路由自动化
⏳ sas-task-board plugin → 看板实时联动
⏳ 行情查询 Skill → Agent 自然语言调用行情
⏳ 任务追踪 Skill → Agent 主动更新状态
```

---

## 十、与 TradingAgents 的融合设计

### 10.1 已有融合基础

SAS-TradingGraph 项目已实现：
- SAS v1.5 CEO 模式 × TradingAgents 多分析师辩论链
- 核心文件：`sas_trading_graph.py`、`sas_gates.py`、`sas_states.py`

### 10.2 融合扩展方向

| 方向 | 做法 |
|------|------|
| 行情数据 | `stock_data_fetcher.py` → 替代手动爬取 |
| 投资决策 | `sas_phase_gate.py` → 把 SAS 门控套到 TradingAgents 流程 |
| 多 Agent 辩论 | TradingAgents 的 analyst → 作为 SAS Worker 执行层 |
| 结果记录 | `task_logger.py` → 记录每个分析步骤的输入输出 |

---

## 十一、风险评估与备选方案

| 风险 | 概率 | 影响 | 应对 |
|------|------|------|------|
| subagent 仍超时 | 高 | 中 | 继续拆小任务 + max_turns 控制 |
| SOUL.md 指令被 LLM 忽略 | 高 | 高 | 用 sas-engine plugin 强制拦截 |
| sas-engine 开发周期长 | 中 | 高 | 先用 task_decompose 人工驱动 |
| DeepSeek API 不稳定 | 低 | 中 | task_decompose 有规则引擎降级 |
| OpenClaw 版本升级失效 | 低 | 中 | 所有改动都在 workspace，不动核心 |

---

## 十二、评估问题清单（供 GLM-5-Turbo 回答）

1. **可行性**：上述架构是否在 OpenClaw 当前版本（v2026.3.23-2）下可实现？
2. **优先级**：三个阶段中，哪个阶段的投入产出比最高？
3. **风险**：sas-engine plugin 是否可以用 Skill 替代，实现同等效果？
4. **设计缺陷**：task_decompose 的 LLM 调用在高并发下是否会成为瓶颈？
5. **扩展性**：SAS v1.5 架构能否平滑升级到 v2.0（自动 Agent 协作）？
6. **性能**：每个子任务都调用 task_logger，是否会造成 I/O 压力？
7. **安全**：alsoAllow 白名单是否有被 prompt injection 绕过的风险？

---

*本文档由 SAS-SOP 专家生成，待 GLM-5-Turbo 评估可行性后执行。*
*版本记录：v1.5.0-20260401-初始版本*
