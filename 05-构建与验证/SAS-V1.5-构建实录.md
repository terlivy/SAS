# SAS v1.5 智能化改造构建实录

> 版本：v1.5.0
> 日期：2026-04-01
> 状态：✅ 已完成（OpenClaw集成验证通过，2026-04-01 19:45）
> 验证人：SnoopyClaw (SC)
> 面向：SAS v1.5 CEO 模式落地验证
> 模型分配：sas-default = MiniMax-M2.7-highspeed（主），GLM-5-Turbo（备）

---

## 一、SAS v1.5 是什么

### 1.1 一个工作准则，不是一个软件

SAS（Snoopy Agent Smart）是一套 **AI 协作工作准则**，定义了在 OpenClaw 平台上，多个 Agent 如何分工、如何审批、如何记录工作。

SAS v1.5 是这个准则的第 8 个版本，核心升级是 **CEO 模式**——用户只做两件事：审批计划 + 审批方案，其余全部由 Agent 自治。

### 1.2 SAS v1.5 的六阶段流程

```
阶段1：接收任务
  用户输入任务 → SC Agent 接收
    ↓
阶段2：制定计划（L2/L3 需要用户审批）
  任务复杂度判断（L1/L2/L3）
  → 生成执行计划
  → L1 直接过，L2/L3 需用户确认
    ↓
阶段3：执行工作
  SC 协调 → spawn 子任务
  → 各 Agent 执行
    ↓
阶段4：质量检查
  自检 → 验证 → 复核
    ↓
阶段5：交付成果
  完整交付物 + 使用说明
    ↓
阶段6：归档记忆
  关键决策记录 → 经验沉淀
```

### 1.3 四层 Agent 分工

```
L0 用户（CEO）
  └── 只审批：计划 + 方案

L1 SC Agent（主脑）
  └── 接收任务、协调、汇报

L2 SAS-Default / SAS-Leader / Domain Agent
  └── 标准执行 + 任务协调 + 垂直领域专家

L3 Worker（spawn 的子任务）
  └── 具体执行
```

---

## 二、三个仓库的分工

| 仓库 | 存什么 | 当前位置 |
|------|--------|---------|
| **SAS** | 工作准则原文（MD 文件） | `https://github.com/terlivy/SAS.git` |
| **SAS-script** | 自动化脚本（Python） | `https://github.com/terlivy/SAS-script.git` |
| **SAS-plug-in** | OpenClaw 插件代码（TypeScript） | `https://github.com/terlivy/SAS-plug-in.git` |

---

## 三、这几天做了什么（构建实录）

### Day 1（3月28日）：确认 SAS v1.4 → v1.5 升级

**做了什么**：
- 从 SAS 仓库拉取 v1.5 准则文档（`Snoopy Agent Smart (SAS)工作准则-v1.5.md`，1264行）
- 确认 v1.5 核心变化：CEO 模式、L1-L3 复杂度分级、四层 Agent 权限体系
- 将 v1.5 准则导入 memory-lancedb-pro（68条记忆）

**文件位置**：
```
SAS 仓库 → /tmp/SAS/
  └── Snoopy Agent Smart (SAS)工作准则-v1.5.md

memory-lancedb-pro:
  └── ~/.openclaw/memory/lancedb-pro/  (68条 v1.5 准则记忆)
```

---

### Day 2（3月30日）：建立 SAS 三层架构

**背景问题**：
- SAS 准则有 100,000 字，直接塞进 Agent instructions 只有 2,000 字（2% 覆盖率）
- Token 消耗过大（~30,000 tokens/次）

**解决方案：三层架构**

```
Layer 1: Agent Instructions（~2,000字符）
  └── 核心理念 + 六阶段流程 + 执行铁律

Layer 2: Memory System（按需检索）
  └── 完整 SAS 准则存入 memory-lancedb-pro
  └── 任务执行时自动检索相关章节

Layer 3: 外部文档（人工查阅）
  └── 复杂场景查阅完整文档
```

**文件位置**：
```
Agent instructions 配置：
  ~/.openclaw/workspace-sas-default/SOUL.md  (SAS v1.5 精简指令)

Memory 向量数据库：
  ~/.openclaw/memory/lancedb-pro/  (68条记忆)
```

---

### Day 3（3月31日）：注册 9 个 Agent + 配置插件

**已注册的 Agents（9个）**：

| Agent | 用途 | workspace |
|-------|------|----------|
| SC（main） | 主脑，负责协调 | `~/.openclaw/workspace/` |
| SAS-Default | 标准任务执行 | `~/.openclaw/workspace-sas-default/` |
| SAS-Leader | 任务协调分配 | `~/.openclaw/workspace-sas-leader/` |
| SAS-SOP专家 | 每日准则优化 | `~/.openclaw/workspace-sas-sop-expert/` |
| HR | 飞书招聘助手 | `~/.openclaw/workspace-hr/` |
| 天气 | 贵阳天气播报 | `~/.openclaw/workspace-weather/` |
| 科技资讯 | 每日资讯爬取 | `~/.openclaw/workspace-tech-news-assistant/` |
| 小文 | 文档处理专家 | `~/.openclaw/workspace-xiaowen/` |
| 文档专家 | doc-expert | `~/.openclaw/workspace-doc-expert/` |

**已安装的 Plugins（3个）**：

| Plugin | 版本 | 功能 |
|--------|------|------|
| `memory-lancedb-pro` | 1.1.0-beta.9 | 向量记忆存储 + 智能检索 |
| `openclaw-lark` | 2026.3.25 | 飞书集成 |
| `lossless-claw-enhanced` | latest | 上下文压缩（75% 阈值，DAG 摘要） |

**插件配置详情**：

```
memory-lancedb-pro:
  - 嵌入模型: bge-m3 (Ollama)
  - Smart Extraction: ON
  - 数据库路径: ~/.openclaw/memory/lancedb-pro/

lossless-claw-enhanced:
  - 摘要模型: MiniMax-M2.7-highspeed
  - 压缩阈值: 0.75（75% 上下文窗口触发）
  - 保护消息数: 32（最新 32 条不压缩）
  - CJK 感知令牌估算
  - 数据库路径: ~/.openclaw/lcm.db
```

---

### Day 4（4月1日）：SAS-script 自动化脚本开发

**问题**：
- SC Agent 收到复杂任务后直接问用户需求，不自动拆解
- subagent 经常做一半就超时，结果不完整

**做了什么**：

#### 1. 写 `task_decompose.py`

**目的**：把复杂任务自动拆成结构化子任务列表

**位置**：
- 本地：`~/.openclaw/workspace/task_decompose.py`
- 仓库：`https://github.com/terlivy/SAS-script.git` → `scripts/task_decompose.py`

**核心逻辑**：
```
用户输入复杂任务
  → DeepSeek LLM API 拆解（主要）
  → 规则引擎降级（LLM 失败时）
  → 输出结构化 JSON：
      {
        "total_subtasks": 6,
        "subtasks": [
          {id, title, description, complexity,
           estimated_timeout, sas_phase, dependencies}
        ],
        "execution_strategy": "sequential/parallel/hybrid"
      }
```

**实际测试**：
```
输入："帮我分析一下 AAPL 的投资价值"
输出：6 个子任务（需求分析→信息获取→基本面分析→
        技术面分析→估值分析→投资建议）
```

#### 2. 写 `task_logger.py`

**目的**：记录每个子任务的开始/完成/失败状态

**位置**：
- 仓库：`https://github.com/terlivy/SAS-script.git` → `scripts/task_logger.py`

**日志格式**：
```jsonl
{"ts": "2026-04-01T18:50:00", "event": "task_start",
 "task_id": "stock-monitor-1", "title": "需求分析",
 "agent": "sas-default"}
{"ts": "2026-04-01T18:52:00", "event": "task_complete",
 "task_id": "stock-monitor-1", "duration": 120}
```

**存储路径**：`~/.openclaw/logs/tasks/YYYYMMDD.jsonl`

#### 3. SAS-script 仓库现有脚本一览

| 脚本 | 用途 | 状态 |
|------|------|------|
| `sas_daily_check.py` | 每日准则变更检查（MD5哈希对比） | ✅ 已在仓库 |
| `sas_sop_expert_v2.py` | 执行分析报告 | ✅ 已在仓库 |
| `sas_github_sync.py` | GitHub 自动同步 | ✅ 已在仓库 |
| `sas_alert_system.py` | 告警系统（失败率/Token/空转） | ✅ 已在仓库 |
| `task_decompose.py` | 任务自动拆解 | ✅ 新增，已推送 |
| `task_logger.py` | 任务日志记录 | ✅ 新增，已推送 |
| `openclaw_agent_integration.py` | Agent 集成模块 | ✅ 已在仓库 |
| `apply_sas_update_v2.py` | 应用准则更新 | ✅ 已在仓库 |

#### 4. crontab 定时任务配置

```
# 每日早上8点（北京时间）检查准则变更
0 8 * * * cd /home/openclaw && python3 scripts/sas_daily_check.py

# 每日晚上9点（北京时间）生成执行分析报告
0 21 * * * cd /home/openclaw && python3 scripts/sas_sop_expert_v2.py

# 每小时检查告警条件
0 * * * * cd /home/openclaw && python3 scripts/sas_alert_system.py
```

#### 5. SOUL.md 指令注入

**SC Agent（主脑）的 SOUL.md** 注入 SAS v1.5 核心指令：

**文件**：`~/.openclaw/workspace/SOUL.md`

**注入内容**：
```
## SAS v1.5 任务自动拆解

### 触发条件（满足任一即触发）
- 任务涉及 2 个以上 Agent 协作
- 任务包含 3 个以上执行步骤
- 任务描述超过 50 字
- 包含关键词：系统、平台、完整、前后端、分析、报告...

### 执行流程
1. 调用 task_decompose → 返回 JSON 子任务列表
2. 解析 dependencies，按顺序 spawn
3. 每个子任务用 task_logger 记录
4. 超时 80% 预警，完成后汇总交付

### 不许做的事
- ❌ 收到任务直接问用户需求（应调用 decompose）
- ❌ 所有步骤在一个任务里执行
- ❌ 跳过 planning 直接进入 execution
```

**SAS-Default Agent 的 SOUL.md**：已更新为 SAS v1.5 完整精简指令

#### 6. alsoAllow 白名单配置

**目的**：让 SC Agent 能够通过 `exec` 工具调用 `task_decompose.py`

**文件**：`~/.openclaw/openclaw.json`

**配置**：
```json
{
  "agents": {
    "main": {
      "alsoAllow": [
        {
          "path": "workspace",
          "commands": ["python3 task_decompose.py"]
        }
      ]
    }
  }
}
```

---

## 四、SAS-plug-in 插件现状

### 4.1 插件设计（TODO，未实际部署）

**插件名称**：`@openclaw/sas-engine`

**设计功能**：
1. **Phase Engine**（`phase-engine.ts`）：六阶段门控流水线
2. **Approval Engine**（`approval-engine.ts`）：自动审批规则引擎
3. **Watchdog**（`watchdog.ts`）：超时守护
4. **State Machine**（`state-machine.ts`）：任务状态机

**插件配置设计**：
```json
{
  "plugins": {
    "entries": {
      "sas-engine": {
        "package": "@openclaw/sas-engine",
        "enabled": false,
        "config": {
          "phaseTimeouts": {
            "plan": 3600,
            "design": 7200,
            "implement": 14400,
            "test": 10800,
            "review": 3600
          },
          "watchdog": {
            "pendingWarnThreshold": 7200,
            "pendingRecoverThreshold": 21600
          }
        }
      }
    }
  }
}
```

### 4.2 当前状态

**仓库中有代码**（TypeScript，beta 阶段），但**未安装到 OpenClaw**。

插件未安装原因：
1. 尚未发布到 npm（`@openclaw/sas-engine` 不存在）
2. 源码存在但未构建
3. 需要 TypeScript 编译 + npm 打包

---

## 五、SAS 六阶段在当前系统的实现对应

| SAS 阶段 | 系统中的实现 | 状态 |
|---------|------------|------|
| 阶段1 接收任务 | SC Agent 接收消息 | ✅ 工作 |
| 阶段2 制定计划 | task_decompose.py（手动调用） | ⚠️ 半自动，需手动触发 |
| 阶段3 执行工作 | subagent spawn（带 max_turns） | ✅ 可用 |
| 阶段4 质量检查 | SOUL.md 指令（描述性） | ❌ 无强制机制 |
| 阶段5 交付成果 | SC Agent 汇报 | ✅ 工作 |
| 阶段6 归档记忆 | memory-lancedb-pro（自动向量存储） | ✅ 工作 |

**关键缺口**：
- 阶段2（制定计划）：SC Agent 没有自动调用 task_decompose，需要新 session + LLM 自觉遵守
- 阶段4（质量检查）：没有强制 Gate 机制，Agent 自行判断

---

## 六、所有文件当前位置总览

### OpenClaw 配置

```
~/.openclaw/openclaw.json          ← 主配置文件
~/.openclaw/agents/                ← 9个 Agent 的会话存储
~/.openclaw/extensions/            ← 3个已安装插件
~/.openclaw/cron/jobs.json         ← 定时任务配置
~/.openclaw/lcm.db                 ← lossless-claw 压缩数据库
~/.openclaw/memory/lancedb-pro/    ← 向量记忆数据库
```

### Agent Workspaces

```
~/.openclaw/workspace/             ← SC 主脑 workspace
~/.openclaw/workspace-sas-default/ ← SAS-Default Agent
~/.openclaw/workspace-sas-leader/  ← SAS-Leader Agent
~/.openclaw/workspace-hr/         ← HR Agent
~/.openclaw/workspace-weather/     ← 天气 Agent
~/.openclaw/workspace-xiaowen/    ← 小文 Agent
~/.openclaw/workspace-doc-expert/ ← 文档专家 Agent
```

### SAS-script 脚本

```
/home/openclaw/scripts/                        ← Git 仓库根目录（同步 GitHub SAS-script）
/home/openclaw/scripts/scripts/*.py            ← 实际脚本位置（12个）
/home/openclaw/scripts/task_decompose.py       ← 软链接 → scripts/（兼容旧路径）
```

**Git 同步状态**：✅ 与 SAS-script 远程仓库同步（0 ahead/behind）
**Cron 命令**：`cd /home/openclaw && python3 scripts/sas_xxx.py`

### GitHub 仓库

```
https://github.com/terlivy/SAS             ← 准则文档
https://github.com/terlivy/SAS-script      ← 自动化脚本
https://github.com/terlivy/SAS-plug-in    ← 插件源码（未部署）
https://github.com/terlivy/snoopyclaw-skills  ← Skills
```

---

## 七、已验证有效的内容

### ✅ 验证通过

| 功能 | 验证方式 | 结果 |
|------|---------|------|
| SC 调用 task_decompose | TUI 中 exec 命令 | ✅ DeepSeek LLM 成功拆解出 6 个子任务 |
| SAS v1.5 指令注入 SC | `grep` 检查 SOUL.md | ✅ 指令已写入 |
| memory-lancedb-pro 检索 | OpenClaw TUI 检索 | ✅ 68条 v1.5 记忆可检索 |
| lossless-claw 压缩 | 上下文超过 75% 触发 | ✅ CJK 感知，压缩正常 |
| 9个 Agent 注册 | `openclaw agents list` | ✅ 全部显示 |
| 飞书集成 | 消息收发 | ✅ 正常 |

### ⚠️ 部分有效

| 功能 | 现状 | 限制 |
|------|------|------|
| task_decompose | 脚本存在，可手动调用 | SC 不自动触发，需要新 session |
| SAS-Default Agent | workspace 有 SAS v1.5 指令 | 无独立 TUI 入口，只是工具箱 |
| 任务看板 | task_logger 可写日志 | 前端面板未接入，数据看不到 |

### ❌ 未实现

| 功能 | 原因 |
|------|------|
| 阶段自动门控 | sas-engine 插件未构建/未安装 |
| 自动审批 | 依赖 sas-engine |
| subagent 超时控制 | OpenClaw 无配置项，靠 max_turns |
| 任务看板实时联动 | 需要 sas-task-board 插件 |

---

## 八、给 GLM-5-Turbo 的评估问题

1. **优先级**：阶段2自动拆解（task_decompose 自动触发）和阶段4门控（sas-engine），哪个先做投入产出比更高？

2. **Plugin vs Skill**：sas-engine 的 Phase Engine（拦截所有消息做门控检查）是否可以用 OpenClaw Skill 实现同等效果？还是有本质差异？

3. **触发机制**：SC Agent 的 SOUL.md 写了"调用 task_decompose"，但 LLM 不一定遵守。有什么方式可以让这个指令更强制，而不是依赖 LLM 的自觉性？

4. **subagent 超时**：OpenClaw 的 subagent 没有可配置的 timeout 参数。你建议如何处理"任务做一半被 kill"的问题？继续拆小任务之外有没有别的方案？

5. **内存压力**：每个子任务都调用 task_logger 写 JSONL，I/O 是否会成为瓶颈？如果同时跑 10 个子任务，每秒写入量大约是多少？

6. **插件开发成本**：sas-engine 的 4 个模块（phase-engine、approval-engine、watchdog、state-machine），如果用 TypeScript 从零开发，估计需要多少时间？

---

*本文档由 SAS-SOP 专家生成，如实记录这几天的构建过程和当前状态。*
*版本记录：v1.5.0-20260401-构建实录-初稿*
