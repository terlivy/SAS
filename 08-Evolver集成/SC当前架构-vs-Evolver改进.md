# SC 当前完整架构 vs Evolver 增强方案
## SnoopyClaw 现状盘点 + Evolver 改进蓝图

> **盘点日期**：2026-04-21
> **目的**：厘清当前架构，确定 Evolver 能改进的具体模块

---

# 一、当前完整架构（全览）

```
┌─────────────────────────────────────────────────────────────────────┐
│                        运行环境                                      │
│  Windows 11 WSL2 (Ubuntu 22.04) / Node.js v22.22.1              │
│  Python 3.10+ / FastAPI + Vue 3 + SQLite                        │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     SC 主脑（SnoopyClaw）                            │
│  模型：MiniMax-M2.7-highspeed（主） / GLM-5-Turbo（备）          │
│  向量记忆：LanceDB Pro (bge-m3, 1024维)                           │
│  通信：Webchat / 飞书 / TUI / Telegram                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.1 Skills（技能系统）

### 自建 Skills（28个）

| 类别 | Skills | 功能 |
|------|--------|------|
| **核心运维** | `system-healer` | 自愈机制（Ollama/Gateway/端口检查+修复） |
| | `workspace-manager` | 工作区审计、优化、维护 |
| | `harness-leader` | 多 Agent 协作编排 |
| | `sas-default` | SAS 准则执行 |
| | `sas-task-planner` | 任务全生命周期管理 |
| | `task-planner` | 通用任务规划 |
| | `self-improving-agent` | 持续改进 |
| **博士工作站** | `postdoc-admin` | 行政事务 |
| | `postdoc-cnas` | CNAS 认证管理 |
| | `postdoc-anticheat` | 反蒸馏/安全审计 |
| | `postdoc-policy` | 政策情报追踪 |
| | `postdoc-recruit` | 人才引进管理 |
| | `postdoc-ip` | 知识产权管理 |
| | `postdoc-project` | 科研项目管理 |
| | `postdoc-director` | 所长事务 |
| **文档处理** | `word-docx` | Word 文档 |
| | `excel-xlsx` | Excel 处理 |
| | `powerpoint-pptx` | PPT 处理 |
| | `doc-handler` | Word/PDF/Excel 综合 |
| | `official-document-template` | 公文排版（GB/T 9704-2012） |
| **设计与爬虫** | `frontend-design-3` | 前端界面生成 |
| | `diagram-generator` | 图表生成 |
| | `playwright-scraper-skill` | 网页爬取 |
| | `graphify` | 知识图谱生成 |
| **工具类** | `memory-lancedb-pro-skill` | 向量记忆管理 |
| | `nano-pdf` | PDF 编辑 |
| | `video-frames` | 视频帧提取 |
| | `websearch` | 网页搜索 |
| | `clawteam` | 多 Agent 协作（CLI） |
| | `snoopyclaw-skills` | SC 专属技能 |

### OpenClaw 内置 Skills（50+个）

| Skills | 功能 |
|--------|------|
| `github` / `gh-issues` | GitHub 操作 |
| `healthcheck` | 主机安全检查 |
| `node-connect` | 节点连接诊断 |
| `weather` | 天气查询 |
| `cron` | 定时任务管理 |
| `feishu-*` | 飞书全家桶（日历/表格/文档/IM） |
| `canvas` | Web Canvas |
| `coding-agent` | 编码 Agent |
| `clawhub` | Skill 市场 |

---

## 1.2 Agents（子智能体）

| Agent | 模型 | 角色 |
|-------|------|------|
| **main（SC）** | MiniMax-M2.7-highspeed | 主脑，统筹协调 |
| **hr** | GLM-5-Turbo | 招聘、入职、团队管理 |
| **doc-expert** | DeepSeek Chat | 文档处理专家 |
| **postdoc-assistant** | ? | 博士后工作站全流程辅助 |
| **sas-default** | ? | SAS 准则执行 |
| **sas-leader** | ? | SAS 任务分解+派发 |
| **sas-sop-expert** | ? | SAS 准则每日优化 |
| **deepseek** | DeepSeek Chat | 编程任务 |
| **weather** | GLM-4.5-Air | 天气查询 |
| **tech-news** | ? | 科技新闻收集 |

---

## 1.3 记忆系统

```
记忆存储
├── LanceDB Pro（向量库）
│   ├── bge-m3 embedding（1024维）
│   ├── smartExtraction（LLM自动6类提取）
│   ├── Weibull 智能遗忘
│   └── 多作用域隔离
│
├── 文件记忆
│   ├── MEMORY.md（精简索引）
│   ├── memory/
│   │   ├── people.md
│   │   ├── projects.md
│   │   ├── lessons.md（14条教训）
│   │   ├── tech-stack.md
│   │   └── YYYY-MM-DD.md（每日日志）
│   └── MyTasks/（任务归档）
│
└── Session 记忆
    └── session-index.md（跨会话索引）
```

---

## 1.4 自动化系统

| 系统 | 当前实现 |
|------|---------|
| **心跳巡检** | HEARTBEAT.md（仅检查 Ollama 是否存活） |
| **定时任务** | Cron（via OpenClaw） |
| **自愈机制** | `system-healer` Skill（部分实现） |
| **GitHub 同步** | `python3 sas_github_sync.py`（每日 02:00） |
| **会话记忆同步** | `python3 sync_session_memory.py`（每日 02:00） |

---

## 1.5 网络与监控

```
┌──────────────────────┐     ┌─────────────────────┐
│  AI Monitor 后端     │     │  AI Monitor 前端     │
│  FastAPI + SQLite    │────▶│  Vue 3 + Tailwind   │
│  端口: 8000         │     │  端口: 3000         │
└──────────────────────┘     └─────────────────────┘
            │                         │
            ▼                         ▼
┌──────────────────────┐     ┌─────────────────────┐
│  ClawTeam 集成      │     │  Webchat 界面      │
│  sessions_spawn     │     │  飞书/TUI/Telegram │
│  team status API    │     │                    │
└──────────────────────┘     └─────────────────────┘
```

---

## 1.6 外部集成

| 集成 | 状态 |
|------|------|
| **飞书** | 消息/日历/表格/文档/多维表格 |
| **Telegram** | Bot（未详细说明） |
| **GitHub** | CLI（gh）已集成 |
| **YouTube** | yt-dlp + cookies 工具链 |
| **ClawTeam** | 已集成（sessions_spawn） |
| **Ollama** | bge-m3 向量模型本地运行 |

---

## 1.7 知识库（Knowledge Base）

路径：`F:\SC-Workspace\knowledge\`

```
knowledge/
├── 00-README.md
├── 01-OpenClaw/          # OpenClaw 体系
├── 02-SnoopyClaw/        # SC 专属
├── 03-技术文档/            # 模型知识库/Token台账
├── 04-项目档案/            # SAS-TradingGraph/AI-Monitor
├── 05-基础工作/            # 技能跟踪/模板工具
├── 06-科研管理/            # 待填充
└── archive/
```

---

# 二、Evolver 改进方案（针对每个模块）

## 2.1 改进方案总览

```
当前架构                          加入 Evolver 思路后
────────────────                ────────────────────────
HEARTBEAT.md（简单心跳）    →    Ops 模块（健康检查矩阵）
system-healer（部分自愈）   →    Gene/Capsule 自愈
记忆系统（存储为主）        →    基因库（可检索解决方案）
上下文（手动管理）          →    信号识别（自动优先级排序）
ClawTeam（任务派发）       →    跨 Agent 经验共享
SAS v1.7（固定流程）       →    策略模式 + 进化引擎
日常日志（流水账）          →    EvolutionEvent（能力演进审计）
```

---

## 2.2 具体改进：HEARTBEAT.md → Ops 模块

### 当前状态

```markdown
# HEARTBEAT.md
1. 检查 Ollama 进程是否存活：pgrep -x ollama
2. 若 Ollama 挂了 → 告诉老板
3. 若是周日 → 执行记忆深度整理
```

**问题**：只检查 Ollama，其他一无所知

### 改进后：Ops 健康检查矩阵

```python
# ops/health_check.py
HEALTH_CHECK_ITEMS = {
    "ollama": {
        "check": "pgrep -x ollama",
        "status": "running",
        "threshold": 1,
        "action": "restart_ollama"
    },
    "gateway": {
        "check": "pgrep -x openclaw",
        "status": "running",
        "threshold": 1,
        "action": "restart_gateway"
    },
    "disk": {
        "check": "df -h / | awk 'NR==2 {print $5}'",
        "status": "<80%",
        "threshold": 80,
        "action": "cleanup_disk"
    },
    "context_usage": {
        "check": "session_status (API)",
        "status": "<150K",
        "threshold": 150000,
        "action": "archive_context"
    },
    "agents": {
        "check": "clawteam team status",
        "status": "all_alive",
        "threshold": 1,
        "action": "notify_and_repair"
    },
    "memory_db": {
        "check": "lancedb ping",
        "status": "healthy",
        "threshold": 1,
        "action": "restart_lancedb"
    }
}
```

**输出示例**：
```
【Ops 健康报告 - 19:50】
├── Ollama:      ✅ 运行中 (PID 375)
├── Gateway:     ✅ 运行中
├── 磁盘使用率:  ⚠️ 78%（建议清理）
├── 上下文使用:  ✅ 45K/200K
├── 子Agent存活:  ⚠️ 3/4（backend 挂了）
└── Memory DB:   ✅ 健康
```

---

## 2.3 具体改进：system-healer → Gene/Capsule 自愈

### 当前状态

```python
# system-healer skill
def check_and_fix():
    if not ollama_running():
        return "Ollama 挂了，手动重启：ollama serve"
    # ... 其他检查
```

**问题**：修复步骤是硬编码的，没有积累

### 改进后：基因驱动的自愈

```python
# evolver/genes/genes.json
{
  "genes": [
    {
      "gene_id": "gene_ollama_restart",
      "name": "Ollama 重启流程",
      "signals": ["ollama", "crash", "11434", "model_load_failed"],
      "solution": """
        1. 检查进程：pgrep -x ollama
        2. 若存在则 kill 对应 PID
        3. 若不存在则执行：ollama serve
        4. 等待5秒后验证：pgrep -x ollama
        5. 若失败则发送通知到飞书
      """,
      "validation": "pgrep -x ollama && echo 'ok'",
      "success_rate": 0.95,
      "usage_count": 12,
      "last_used": "2026-04-21T10:00:00Z"
    },
    {
      "gene_id": "gene_gateway_restart",
      "name": "Gateway 重启",
      "signals": ["gateway", "bind", "port", "18789"],
      "solution": "openclaw gateway restart",
      "validation": "curl -s http://127.0.0.1:18789/health",
      "success_rate": 0.90,
      "usage_count": 5
    },
    {
      "gene_id": "gene_context_archive",
      "name": "上下文归档",
      "signals": ["context", "near_limit", "150k", "memory_full"],
      "solution": """
        1. 查询当前 session context 使用量
        2. 识别低优先级上下文（历史日志、已完成任务）
        3. 归档到 memory/YYYY-MM-DD.md
        4. 通知用户已自动归档
      """,
      "validation": "session_status API < 100K",
      "success_rate": 0.88,
      "usage_count": 8
    }
  ]
}
```

**工作流程**：
```
Ops 检查发现异常
    ↓
提取信号（signals）：[gateway, bind, failed, 18789]
    ↓
Selector 匹配基因 → 找到 gene_gateway_restart
    ↓
Solidify 执行 solution
    ↓
Validation 验证
    ↓
EvolutionEvent 记录（success/fail + time）
    ↓
Gene usage_count + success_rate 更新
```

---

## 2.4 具体改进：记忆系统 → 信号检索 + 基因库

### 当前状态

- lessons.md：文本格式教训记录
- 向量检索：靠关键词匹配
- 问题：找到教训，但不知道怎么用

### 改进后：向量检索 + 基因库双层检索

```
用户问：「上次 ClawTeam agent 挂了怎么办」
    ↓
SC 提取信号：["clawteam", "agent", "down", "tmux", "restart"]
    ↓
┌─────────────────┐    ┌─────────────────┐
│  向量检索        │    │  基因库匹配      │
│  memory/lessons │    │  evolver/genes  │
└────────┬────────┘    └────────┬────────┘
         │                        │
         ▼                        ▼
    「ClawTeam agent      「上次修复流程：
     挂了，Sessions        gene_cta_restart
    _as_tasks 检查...」      signals 完全匹配！」
         │                        │
         └───────────┬────────────┘
                     ▼
              SC 返回解决方案：
              「执行 gene_cta_restart：
               1. clawteam lifecycle idle team-name
               2. tmux kill-session -t agent-name  
               3. clawteam spawn --team team-name
               验证：clawteam team status」
```

---

## 2.5 具体改进：上下文 → 信号识别的自动管理

### 当前状态

- 手动管理 200K context
- 满了靠 reset
- 没有优先级概念

### 改进后：上下文优先级矩阵

```python
# ops/context_manager.py

CONTEXT_PRIORITY = {
    # 高优先级（永不丢失）
    "critical": [
        "用户当前任务目标",
        "未完成的 SAS 阶段状态",
        "用户明确偏好/指令",
        "关键决策点"
    ],
    
    # 中优先级（可压缩）
    "medium": [
        "已完成任务的摘要",
        "子 Agent 返回的结果",
        "已验证的交付物",
        "历史经验"
    ],
    
    # 低优先级（可归档）
    "low": [
        "详细日志输出",
        "中间试错过程",
        "已完成的简单查询",
        "过期的检查结果"
    ]
}

def manage_context():
    usage = get_context_usage()  # token count
    if usage > 150000:
        # 自动归档低优先级
        archive_low_priority()
        # 压缩中优先级
        compress_medium_priority()
    elif usage > 100000:
        # 警告，提示用户
        notify_user("上下文使用率 50%")
```

---

## 2.6 具体改进：ClawTeam → 跨 Agent 经验共享

### 当前状态

- 各 Agent 独立工作
- 无跨 Agent 知识传递
- Leader 不知道 Worker 学到了什么

### 改进后：ClawTeam + Gene 共享

```
Agent-Backend 执行任务
    ↓
遇到问题「Ollama 模型加载失败」
    ↓
调用 Gene 选择器
    ↓
找到/创建 gene_ollama_model_fix
    ↓
EvolutionEvent 记录：
{
  "type": "gene_discovered",
  "gene_id": "gene_ollama_model_fix", 
  "discovered_by": "Agent-Backend",
  "signals": ["ollama", "model", "load", "failed"],
  "solution": "ollama stop && ollama serve"
}
    ↓
共享到全局基因库
    ↓
Agent-Frontend 下次遇到同样问题
    ↓
直接查基因库 → 找到 gene_ollama_model_fix
    ↓
无需重试，直接应用解决方案
```

**实现方式**：
```python
# evolver/clawteam_integration.py
def on_agent_task_complete(agent_id, task_id, result):
    # 1. 提取结果中的信号
    signals = extract_signals(result)
    
    # 2. 判断是否值得存档
    if is_notable_fix(result):
        gene = create_gene(
            signals=signals,
            solution=extract_solution(result),
            discovered_by=agent_id
        )
        # 3. 存入基因库
        save_to_genes_json(gene)
        
        # 4. 广播给其他 Agent
        broadcast_to_team(f"Agent-{agent_id} 发现了新基因: {gene.name}")

def on_agent_task_failed(agent_id, task_id, error):
    # 1. 提取错误信号
    signals = extract_error_signals(error)
    
    # 2. 在基因库中搜索
    matched = find_genes_by_signals(signals)
    
    if matched:
        # 3. 返回已知解决方案
        return f"其他 Agent 遇到过这个问题: {matched[0].solution}"
    else:
        # 4. 记录为待解决信号
        save_to_pending_signals(signals)
```

---

## 2.7 具体改进：SAS 框架 → 策略模式 + EvolutionEvent

### 当前状态

- 六阶段固定流程
- 无策略切换
- 任务完成即归档，无演进追踪

### 改进后：SAS + Evolver 增强

#### 策略模式（Strategy Presets）

```markdown
## SAS 工作模式（v1.8 新增）

当前模式：balanced（默认）
可用模式：
├── balanced  → 50%创新 30%优化 20%修复
├── innovate  → 80%创新 15%优化  5%修复（冲刺期）
├── harden    → 20%创新 40%优化 40%修复（稳定期）
└── repair    →  0%创新 20%优化 80%修复（应急期）

切换方式：/sc mode innovate
```

#### EvolutionEvent（任务演进追踪）

```json
// evolver/events/sas_events.jsonl
{"ts":"2026-04-21T10:00:00Z","type":"task_started","task_id":"T001","task":"AI Monitor监控改造","mode":"balanced","phase":"receive"}
{"ts":"2026-04-21T10:05:00Z","type":"phase_completed","task_id":"T001","phase":"plan","subtasks":4,"tokens_used":3200,"duration":300}
{"ts":"2026-04-21T10:10:00Z","type":"task_completed","task_id":"T001","outcome":"success","tokens_used":12500,"duration":600,"quality_score":0.92}
{"ts":"2026-04-21T10:10:30Z","type":"lesson_learned","task_id":"T001","lesson":"ClawTeam派发4个Agent并行，比串行快3倍","encoded_to":"gene_parallel_execution","confidence":0.88}
{"ts":"2026-04-21T10:15:00Z","type":"strategy_switched","from":"balanced","to":"innovate","reason":"下一个任务L1为主，加速处理"}
```

#### 基因驱动的 SAS 增强

```python
# 每次 SAS 任务完成时自动调用
def on_sas_task_complete(task_id, outcome):
    # 1. 分析任务特征
    signals = extract_task_signals(task)
    
    # 2. 评估是否值得存档为基因
    if outcome.success and signals.complexity > 5:
        gene = Gene(
            name=f"task_pattern_{task.type}",
            signals=signals,
            solution=task.optimal_approach,
            sas_context=task.sas_phase_used,
            success_rate=outcome.score
        )
        save_to_genes(gene)
    
    # 3. 记录 EvolutionEvent
    log_evolution_event({
        "type": "sas_task_completed",
        "task_id": task_id,
        "outcome": outcome,
        "gene_created": gene is not None
    })
    
    # 4. 分析策略有效性
    if outcome.score > 0.9:
        update_strategy_confidence(task.mode, +0.1)
    elif outcome.score < 0.6:
        update_strategy_confidence(task.mode, -0.1)
```

---

# 三、改进方案汇总

## 3.1 各模块改进清单

| 模块 | 当前 | 改进后 | 改动量 |
|------|------|--------|--------|
| **HEARTBEAT** | 只查 Ollama | 健康检查矩阵（6项） | 中 |
| **system-healer** | 硬编码修复 | Gene/Capsule 自愈 | 大 |
| **记忆系统** | 向量+文本教训 | 信号检索+基因库 | 中 |
| **上下文** | 手动管理 | 自动优先级排序 | 小 |
| **ClawTeam** | 独立工作 | 跨Agent经验共享 | 中 |
| **SAS框架** | 固定流程 | 策略模式+EvolutionEvent | 中 |
| **日常日志** | 流水账 | EvolutionEvent审计 | 小 |

## 3.2 实施优先级

| 优先级 | 模块 | 理由 |
|--------|------|------|
| **P0** | Ops 模块 | 立即提升系统稳定性 |
| **P1** | Gene 自愈 | 减少重复修复工作 |
| **P2** | EvolutionEvent | 增强 SAS 演进可见性 |
| **P3** | ClawTeam 基因共享 | 提升多Agent协作效率 |
| **P4** | 信号检索记忆 | 增强记忆可用性 |
| **P5** | 上下文自动管理 | 减少手动清理 |

## 3.3 新增文件清单

```
snoopy-evolver/           # 新增目录（Python Skill）
├── SKILL.md              # OpenClaw Skill 入口
├── evolver/
│   ├── __init__.py
│   ├── genes/
│   │   └── genes.json   # 基因库（初始从 lessons.md 提取）
│   ├── capsules/         # 完整解决方案
│   ├── events/
│   │   └── events.jsonl # EvolutionEvent 审计日志
│   ├── selector.py      # 信号→基因选择器
│   ├── strategy.py      # 策略模式切换
│   └── signal.py        # 信号提取+去重
└── ops/
    ├── __init__.py
    ├── health.py        # 健康检查矩阵
    ├── lifecycle.py     # 生命周期管理
    ├── cleanup.py      # 磁盘清理
    ├── context.py       # 上下文管理
    └── clawteam.py     # ClawTeam 基因共享
```

## 3.4 不影响现有集成的理由

| 现有系统 | 集成方式 | 是否破坏 |
|---------|---------|---------|
| 飞书 | Skill 调用（无依赖） | ❌ 无影响 |
| Telegram | Skill 调用（无依赖） | ❌ 无影响 |
| GitHub | Skill 调用（无依赖） | ❌ 无影响 |
| YouTube | Skill 调用（无依赖） | ❌ 无影响 |
| AI Monitor | 独立服务（端口隔离） | ❌ 无影响 |
| LanceDB Pro | 文件级别集成（不修改DB） | ❌ 无影响 |
| ClawTeam | subprocess 调用（外部进程） | ❌ 无影响 |

**结论**：所有改进都是新增文件，不修改现有配置和代码。

---

*本方案由 SC 主脑基于 SAS v1.7 CEO 模式分析生成*
*灵感来源：Evolver (GPL-3.0, github.com/EvoMap/evolver)*
*方案性质：参考架构设计，Python Skill 渐进式自建，无 GPL 开源义务*
