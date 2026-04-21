# SC × Evolver 集成方案
## SnoopyClaw Agent Smart × GEP Self-Evolution Engine

> **基于 SAS v1.7 CEO 模式分析 | 2026-04-21**

---

## 一、项目背景

### 1.1 Evolver 是什么

Evolver 是基于 GEP（Genome Evolution Protocol）的 AI Agent 自进化引擎，核心功能：
- 扫描 memory/ 日志，自动发现错误模式和信号
- 从 Gene/Capsule 库选择最匹配的进化资产
- 输出 GEP 引导的进化提示词
- 记录 EvolutionEvent 审计追踪
- 支持 4 种进化策略（balanced/innovate/harden/repair-only）
- 内置 Ops 模块（生命周期管理、健康监控、自愈）

**与 SC 的契合度**：⭐⭐⭐⭐⭐

### 1.2 SC/SAS 当前架构

```
SC (SnoopyClaw 主脑)
├── SOUL.md           — 核心身份 + SAS CEO 模式
├── MEMORY.md         — 记忆索引 + 红线规则 + 工作模式
├── HEARTBEAT.md      — 心跳巡检（仅检查 Ollama）
├── Skills
│   ├── system-healer      — 自愈机制
│   ├── workspace-manager   — 工作区管理
│   ├── harness-leader     — 多Agent协作
│   ├── memory-lancedb-pro — 向量记忆
│   └── ...（13个自建技能）
└── 任务归档 → memory/YYYY-MM-DD.md

SAS 框架（v1.7）
├── 六阶段：接收 → 计划 → 执行 → 检查 → 交付 → 归档
├── L1/L2/L3 复杂度分级
├── 红线规则（12条）
└── 模型知识库
```

### 1.3 集成的核心价值

| Evolver 模块 | SC 现有能力 | 集成价值 |
|-------------|-----------|---------|
| Ops 模块（lifecycle/check） | HEARTBEAT.md | 从简单巡检升级为完整运维模块 |
| Gene/Capsule 进化资产 | lessons.md / projects.md | 结构化为可复用进化资产 |
| Strategy Presets | 红线规则 | 显式定义工作模式切换 |
| EvolutionEvent | 任务归档 | 增强为能力演进追踪 |
| Signal 去重 | lessons.md | 防止重复修复循环 |

---

## 二、架构对比

### 2.1 Evolver 架构

```
Evolver (Node.js)
│
├── memory/              ← 扫描日志
├── assets/gep/
│   ├── genes.json       ← 可复用进化片段
│   ├── capsules.json    ← 完整解决方案
│   └── events.jsonl     ← EvolutionEvent 审计日志
├── src/
│   ├── evolve.js        ← 核心进化引擎
│   ├── gep/
│   │   ├── prompt.js   ← GEP 提示词生成
│   │   ├── selector.js ← Gene/Capsule 选择器
│   │   └── solidify.js ← 验证 + 固化
│   └── ops/
│       ├── lifecycle.js ← 生命周期管理
│       ├── check.js    ← 健康检查
│       ├── cleanup.js  ← 磁盘清理
│       └── self-repair.js ← 自修复
└── index.js            ← 入口（单次/循环/review）
```

### 2.2 SC 当前架构（升级后目标）

```
SC-Evolver (Python/Skill)
│
├── memory/              ← 已存在（向量+文件）
├── evolver/             ← 新增：进化引擎
│   ├── genes/           ← 可复用教训片段（类似 Gene）
│   ├── capsules/        ← 完整解决方案（类似 Capsule）
│   ├── events.jsonl     ← EvolutionEvent 审计
│   ├── selector.py      ← 信号 → 基因选择
│   ├── strategy.py       ← 进化策略预设
│   └── signal.py        ← 信号提取 + 去重
├── ops/                 ← 新增：运维模块
│   ├── lifecycle.py     ← 生命周期（升级 HEARTBEAT）
│   ├── health.py        ← 健康检查（已有雏形）
│   ├── cleanup.py       ← 清理（可整合）
│   └── self-repair.py   ← 自修复（system-healer 已实现）
└── SC 主脑              ← sessions_spawn 协调
```

---

## 三、集成方案

### 3.1 方案选择：渐进式自建（避免 GPL 传染）

**不直接 fork Evolver 源码**，而是：
1. 参考其架构设计
2. 用 Python/Skill 实现等价功能
3. 通过 subprocess 调用 Evolver CLI（无 GPL 风险）

```
SC (Python)
    │
    ├── [自建] ops/     — Python 实现，等价替换
    ├── [自建] evolver/ — Python 实现
    └── [调用] Evolver CLI — subprocess.run(["node", "evolver/index.js"])
                    ↑
              无 GPL 传染（外部进程调用）
```

### 3.2 四大模块集成计划

#### 模块 A：Ops 模块（立即可做）

**现状**：
```markdown
HEARTBEAT.md
1. 检查 Ollama 是否存活
2. 若挂了 → 通知老板
```

**Evolver Ops 模块**：
```javascript
// lifecycle.js
start → stop → status → check

// check.js
- 健康检查
- 错误率检测
- 停滞检测 → 自动重启
```

**SC 升级方案**：将 HEARTBEAT.md 升级为 `ops/` Skill

```
ops/ Skill
├── check()      — Ollama + Gateway + 子进程
├── self_repair() — system-healer 已有
├── cleanup()    — 临时文件 + 旧日志
├── lifecycle()  — 启动/停止/状态
└── schedule()   — cron 整合
```

**实现方式**：参考 Evolver 的 `src/ops/` 架构，用 Python Skill 实现

#### 模块 B：Gene/Capsule 进化资产（高价值）

**现状**：
```markdown
lessons.md       — 教训列表（文字描述）
projects.md      — 项目记录
memory/lessons.md — 按类组织
```

**Evolver 基因资产**：
```javascript
// genes.json
{
  "id": "gene_001",
  "name": "Ollama Restart",
  "description": "当 Ollama 挂了时的标准恢复流程",
  "signals": ["ollama", "crash", "port 11434"],
  "solution": "重启 Ollama 服务...",
  "validation": ["node -e 'check ollama'"],
  "usage_count": 12,
  "success_rate": 0.95
}
```

**SC 基因库方案**：
```python
# evolver/genes.json
{
  "gene_id": "SC_gene_001",
  "name": "Ollama Restart",
  "description": "Ollama 挂了时的标准恢复",
  "signals": ["ollama", "crash", "11434"],
  "solution_prompt": "当检测到 Ollama 挂了，执行以下步骤...",
  "validation": "pgrep -x ollama",
  "lessons_ref": "memory/lessons.md#ollama",
  "usage_count": 5,
  "success_rate": 1.0
}
```

**集成方式**：
1. 将 lessons.md 中的教训结构化为 Gene
2. lessons.md 保留全文描述，genes.json 提供结构化索引
3. 选择器基于信号匹配基因

#### 模块 C：Strategy Presets（中等价值）

**现状**：MEMORY.md 有红线规则，但无模式切换机制

**Evolver Strategy Presets**：
```bash
EVOLVE_STRATEGY=balanced    # 50%创新 30%优化 20%修复
EVOLVE_STRATEGY=innovate   # 80%创新 15%优化  5%修复
EVOLVE_STRATEGY=harden     # 20%创新 40%优化 40%修复
EVOLVE_STRATEGY=repair-only #  0%创新 20%优化 80%修复
```

**SC Strategy Presets**：
```markdown
## 工作模式（v1.8 新增）

### SC_MODE=balanced（默认）
- 新任务处理：L1 直接执行，L2/L3 走完整 SAS
- 自进化：30% 精力在能力优化
- 修复优先级：中

### SC_MODE=innovate（冲刺期）
- 新任务处理：加速处理，L1 跳过自检
- 自进化：80% 精力在新能力
- 修复优先级：低

### SC_MODE=harden（稳定期）
- 新任务处理：严格质量门控
- 自进化：60% 精力在测试+验证
- 修复优先级：高

### SC_MODE=repair（应急期）
- 新任务处理：暂停
- 自进化：0%
- 修复优先级：最高
```

#### 模块 D：EvolutionEvent 审计（高价值）

**现状**：任务归档在 `memory/YYYY-MM-DD.md`

**Evolver EvolutionEvent**：
```json
{"ts":"2026-04-21T10:00:00Z","type":"gene_applied","gene_id":"...","input_signals":[...],"outcome":"success"}
```

**SC EvolutionEvent（增强版）**：
```json
{"ts":"2026-04-21T10:00:00Z","type":"lesson_encoded","lesson_id":"...","trigger_task":"...","encoded_to":"genes.json","confidence":0.85}
{"ts":"2026-04-21T11:00:00Z","type":"task_completed","task_id":"...","tokens_used":1234,"duration":45,"quality_score":0.9}
{"ts":"2026-04-21T12:00:00Z","type":"gene_applied","gene_id":"SC_gene_001","trigger_signal":"ollama_down","outcome":"success","recovery_time":30}
```

---

## 四、Fork 计划（Evolver-SC）

### 4.1 项目定位

**Evolver-SC**：Evolver 的 Python/OpenClaw 版本
- 保留 Evolver 的 GEP 协议和概念
- 用 Python + Skill 实现
- 与 OpenClaw 深度集成
- 开源（MIT），独立于 GPL

### 4.2 目录结构

```
snoopy-evolver/            ← Fork 项目
├── SKILL.md               ← OpenClaw Skill 入口
├── evolver/
│   ├── __init__.py
│   ├── evolve.py          ← 核心引擎（类似 src/evolve.js）
│   ├── gep/
│   │   ├── prompt.py      ← GEP 提示词（类似 gep/prompt.js）
│   │   ├── selector.py    ← 基因选择器（类似 gep/selector.js）
│   │   ├── solidify.py   ← 验证固化（类似 gep/solidify.js）
│   │   └── protocol.py    ← GEP 协议定义
│   ├── genes/             ← 基因库（替代 assets/gep/genes.json）
│   │   └── genes.json
│   ├── capsules/          ← 胶囊库（替代 assets/gep/capsules.json）
│   │   └── capsules.json
│   ├── events/            ← 审计日志（替代 assets/gep/events.jsonl）
│   │   └── events.jsonl
│   └── ops/               ← 运维模块（类似 src/ops/）
│       ├── lifecycle.py
│       ├── health.py
│       ├── cleanup.py
│       └── signal.py       ← 信号提取 + 去重
├── memory/                 ← 记忆目录（集成 SC 记忆系统）
├── tests/
├── README.md
└── LICENSE (MIT)
```

### 4.3 实施阶段

#### Phase 1：Ops 模块（1天）

**目标**：替换 HEARTBEAT.md 为 ops/ Skill

```python
# ops/health.py
def check():
    """健康检查：Ollama + Gateway + 关键进程"""
    checks = {
        "ollama": pgrep("ollama"),
        "gateway": check_port(18789),
        "memory": check_memory_usage()
    }
    return all(checks.values())

# ops/lifecycle.py
# 参考 src/ops/lifecycle.js 实现 start/stop/status/check
```

**验收标准**：
- [ ] `skill:system-healer` 能调用 `ops/health.check()`
- [ ] 错误率 > 阈值时自动触发修复流程
- [ ] 记录 EvolutionEvent

#### Phase 2：基因库 + 选择器（3天）

**目标**：建立 SC 基因库，实现信号 → 基因选择

```python
# evolver/genes/genes.json
{
  "genes": [
    {
      "gene_id": "SC_gene_001",
      "name": "Ollama 重启",
      "signals": ["ollama", "crash", "11434"],
      "solution": "参考 system-healer 修复流程",
      "validation": "pgrep -x ollama",
      "success_rate": 1.0,
      "usage_count": 5
    }
  ]
}

# evolver/selector.py
def select_gene(signals: List[str]) -> Gene | None:
    """根据信号匹配最合适的基因"""
    # 1. 提取信号
    # 2. 在 genes.json 中搜索
    # 3. 按 success_rate * usage_count 排序
    # 4. 返回 top-1
```

**验收标准**：
- [ ] 从 lessons.md 自动提取基因
- [ ] 信号匹配精度 > 80%
- [ ] 基因库支持 CRUD 操作

#### Phase 3：Strategy Presets（1天）

**目标**：在 MEMORY.md 中定义工作模式

```markdown
## SC 工作模式（v1.8 新增）

当前模式：balanced
切换命令：/sc mode innovate
```

**验收标准**：
- [ ] SC 能根据模式调整处理方式
- [ ] 模式切换有审计日志

#### Phase 4：EvolutionEvent 审计（2天）

**目标**：增强任务归档为 EvolutionEvent

```python
# evolver/events/logger.py
class EvolutionEventLogger:
    def log(self, event_type: str, data: dict):
        """记录 EvolutionEvent"""
        with open("events/events.jsonl", "a") as f:
            f.write(json.dumps({
                "ts": datetime.now().isoformat(),
                "type": event_type,
                **data
            }) + "\n")

    def query(self, event_type: str = None, limit: int = 100):
        """查询事件"""
```

**验收标准**：
- [ ] 所有 SAS 任务完成时记录 EvolutionEvent
- [ ] 可按类型/时间/基因ID 查询
- [ ] 定期生成演进报告

#### Phase 5：集成 + 测试（2天）

**目标**：完整集成测试

- [ ] Ops 模块与 SC 和谐运行
- [ ] 基因库与 lessons.md 数据一致
- [ ] EvolutionEvent 正确记录
- [ ] Strategy 模式正确切换
- [ ] 与 Evolver CLI 集成（可选）

### 4.4 资源估算

| Phase | 估算时间 | Token 消耗 | 风险 |
|-------|---------|-----------|------|
| Phase 1 | 1天 | ~50K | 低 |
| Phase 2 | 3天 | ~150K | 中 |
| Phase 3 | 1天 | ~30K | 低 |
| Phase 4 | 2天 | ~80K | 低 |
| Phase 5 | 2天 | ~60K | 中 |
| **总计** | **9天** | **~370K** | — |

---

## 五、GEP 协议适配

### 5.1 GEP 协议核心概念

| GEP 概念 | Evolver 实现 | SC 实现 |
|---------|-------------|--------|
| Gene | genes.json | genes.json |
| Capsule | capsules.json | capsules.json |
| EvolutionEvent | events.jsonl | events.jsonl |
| Signal | 从日志提取 | 从 heartbeat/error 提取 |
| Selector | gep/selector.js | evolver/selector.py |
| Solidify | gep/solidify.js | evolver/solidify.py |
| Mutation | 显式 Mutation 对象 | 任务决策对象 |
| Personality | 可进化的 PersonalityState | SC 红线规则 |

### 5.2 SC 的 Signal 来源

| Signal 类型 | 来源 | 优先级 |
|------------|------|--------|
| OllamaDown | HEARTBEAT.md | P0 |
| TaskFailed | 子Agent 回报 | P0 |
| ContextNear | session_status | P1 |
| MemoryFull | lancedb-pro | P1 |
| LessonLearned | SC 决策 | P2 |
| SkillMissing | 任务分配失败 | P1 |

---

## 六、风险评估

| 风险 | 概率 | 影响 | 缓解 |
|------|------|------|------|
| 照搬 Evolver 架构，触发 GPL | 低 | 高 | 只参考架构，用 Python 重写 |
| 与现有 system-healer 冲突 | 中 | 中 | Phase 1 先做兼容测试 |
| Token 消耗超预期 | 中 | 中 | 分阶段交付 |
| 用户不习惯新模式 | 低 | 低 | 保持向后兼容 |

---

## 七、立即行动

### 今日可做（< 30分钟）

```bash
# 1. Clone Evolver 看看实际代码结构
git clone https://github.com/EvoMap/evolver.git ~/evolver-reference

# 2. 评估 Node.js 是否可用
node --version  # 检查是否 >= 18

# 3. 试用 Evolver CLI
cd ~/evolver-reference
node index.js --review  # 审查模式，不自动执行
```

### 本周可做

1. **Phase 1**：将 HEARTBEAT.md 升级为 Python Skill
2. **Phase 2**：从 lessons.md 提取前 10 个基因

### 下周可做

1. **Phase 3**：实现 Strategy Presets
2. **Phase 4**：EvolutionEvent 审计

---

## 八、结论

**Evolver 对 SC/SAS 的价值**：

1. ⭐⭐⭐⭐⭐ **Ops 模块**：从简单心跳升级为完整运维体系
2. ⭐⭐⭐⭐ **Gene/Capsule**：将零散教训结构化为可复用资产
3. ⭐⭐⭐⭐ **EvolutionEvent**：SAS 任务归档增强为能力演进追踪
4. ⭐⭐⭐ **Strategy Presets**：显式工作模式切换

**不建议直接 fork Evolver 源码**（GPL 风险），而是：
- 参考架构设计
- 用 Python Skill 实现等价功能
- 通过 subprocess 调用 Evolver CLI（无 GPL 传染）

**推荐路径**：渐进式自建，Phase 1 从 Ops 模块开始。

---

*本方案由 SC 主脑基于 SAS v1.7 CEO 模式分析生成*
*灵感来源：Evolver (GPL-3.0, github.com/EvoMap/evolver)*
