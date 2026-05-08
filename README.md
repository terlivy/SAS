# Snoopy Agent Smart (SAS) 工作准则

> **SAS = Snoopy Agent Smart** — AI 多智能体协作工作准则  
> **核心理念**：慢即是快 — 计划先行，逻辑清晰，有可执行计划，有可落地方案，有效评估周期，可控资源成本  
> **落地版本**：v1.5.0（SC 主脑实际运行）
> **最新 git 版本**：v1.6.0（§2.7.5 新增 + VERSION_GOVERNANCE + CHANGELOG）
> **维护者**：SnoopyClaw (SC) · 2026-03-18 至今

---

## 📖 什么是 SAS？

SAS 是一套运行在 [OpenClaw](https://github.com/openclaw/openclaw) 平台上的 **AI 智能体协作工作准则**，定义了多个 Agent 如何分工、审批、记录和迭代。

它不是一个软件，而是一套**方法论 + 自动化脚本 + 插件引擎**的组合体系。

## 🗂️ 仓库结构

```
SAS/
├── README.md                          ← 你在这里
├── VERSION_GOVERNANCE.md             ← 版本治理规范
├── CHANGELOG.md                      ← 变更记录
├── 大模型信息与Agent分配依据.md           ← Provider + Agent 分配矩阵 + 决策树
│
├── 01-需求调研/                        ← 第一阶段：需求调研
├── 02-可行性分析/                      ← 第二阶段：可行性分析
├── 03-实施计划/                        ← 第三阶段：实施计划
├── 04-工作准则/                        ← 核心文档：SAS 准则各版本
├── 05-构建与验证/                      ← 第四阶段：构建与验证
├── 06-迭代记录/                        ← 版本迭代记录
└── 08-Evolver集成/                    ← snoopy-evolver 集成
```

## 📊 版本历史

| 版本 | 日期 | 核心升级 | 状态 |
|------|------|---------|------|
| **v1.0** | 2026-03-19 | 基础准则，AI协作规范 | ✅ 归档 |
| **v1.1** | 2026-03-22 | 记忆系统，持续学习 | ✅ 归档 |
| **v1.2** | 2026-03-25 | 质量体系，代码规范 | ✅ 归档 |
| **v1.3** | 2026-03-29 | 异常监控与恢复 | ✅ 归档 |
| **v1.4** | 2026-03-31 | 六阶段门控流水线 | ✅ 归档 |
| **v1.5** | 2026-04-01 | CEO模式协作架构 | ✅ 落地（SC实际运行） |
| **v1.6** | 2026-04-08 | §2.7.5 SKILL安全审核 + 版本治理规范 | ✅ git最新 |

## 🏗️ SAS 体系架构

```
┌─────────────────────────────────────────────────────┐
│                 L0 用户（CEO）                       │
│           只做两件事：审批计划 + 审批方案              │
├─────────────────────────────────────────────────────┤
│              L1 SC Agent（主脑）                     │
│        接收任务 → 制定计划 → 协调执行 → 汇报          │
├──────────┬──────────┬──────────┬─────────────────────┤
│  L2 专用  │  L2 专用  │  L2 专用  │    L3 通用          │
│  HR       │  doc-    │  weather │   sas-default       │
│ (glm-5)   │  expert  │ (glm-4.5)│  (deepseek-v4)     │
│           │ (glm-4.7)│          │                    │
├──────────┴──────────┴──────────┴─────────────────────┤
│              L2 sas-leader（自治）                   │
│     独立子任务 → 自主协调 → 完成交付                   │
└─────────────────────────────────────────────────────┘
```

## 🤖 Agent 注册（19个，实际运行8个有workspace）

### L1 主脑（1个）
| Agent | 模型 | 职责 | Workspace |
|-------|------|------|-----------|
| **main (SC)** | minimax/MiniMax-M2.7-highspeed | 主脑：协调、规划、汇报 | ✅ |

### L2 专用Agent（7个，有workspace）
| Agent | 模型 | 职责 | Workspace |
|-------|------|------|-----------|
| **sas-leader** | apimart/gemini-3-pro-preview | 自治任务协调 | ✅ |
| **sas-default** | deepseek/deepseek-v4-flash | 通用子任务 | ✅ |
| **sas-sop-expert** | apimart/gpt-5.2 | 准则优化 | ✅ |
| **hr** | zai/glm-5-turbo | 招聘、团队管理 | ✅ |
| **tech-news** | zai/glm-4.5-air | 科技资讯推送 | ✅ |
| **doc-expert** | zai/glm-4.7 | 文档生成 | ✅ |
| **weather** | zai/glm-4.5-air | 天气播报 | ✅ |

### L2 产研Agent（11个，有配置无workspace）
| Agent | 模型 | Skills数 | 状态 |
|-------|------|:---:|------|
| **requirement-analyst** | apimart/gpt-5.2 | 6 | 有配置无workspace |
| **product-manager** | apimart/gpt-5.2 | 7 | 有配置无workspace |
| **technical-architect** | apimart/gpt-5.2 | 4 | 有配置无workspace |
| **ui-designer** | apimart/gemini-3-pro-preview | 4 | 有配置无workspace |
| **developer** | apimart/gpt-5.2-codex | 5 | 有配置无workspace |
| **code-reviewer** | apimart/gemini-3.1-pro-preview-thinking | 4 | 有配置无workspace |
| **security-reviewer** | apimart/gemini-3.1-pro-preview-thinking | 3 | 有配置无workspace |
| **tester** | apimart/gpt-5.2 | 4 | 有配置无workspace |
| **performance-tester** | apimart/gpt-5.2 | 3 | 有配置无workspace |
| **devops** | apimart/gpt-5.2-codex | 5 | 有配置无workspace |
| **operations-agent** | apimart/gpt-5.2 | 4 | 有配置无workspace |

**注意**：xiaowen Agent 已不存在（已在 openclaw.json 中移除）

## 🔗 关联仓库

| 仓库 | 说明 | 链接 |
|------|------|------|
| **SAS** | 本仓库 — 准则文档 | [terlivy/SAS](https://github.com/terlivy/SAS) |
| **SAS-script** | 自动化脚本（10个） | [terlivy/SAS-script](https://github.com/terlivy/SAS-script) |
| **SAS-plug-in** | sas-engine v1.1.0 插件 | [terlivy/SAS-plug-in](https://github.com/terlivy/SAS-plug-in) |
| **snoopyclaw-skills** | SC 自建技能 | [terlivy/snoopyclaw-skills](https://github.com/terlivy/snoopyclaw-skills) |
| **snoopy-evolver** | 基因演化系统 | [terlivy/snoopy-evolver](https://github.com/terlivy/snoopy-evolver) |

## 🛠️ 自动化基础设施

- **task_decompose.py** — 任务自动拆解（规则引擎，L1/L2/L3 分级）
- **task_logger.py** — 任务日志记录（JSONL 格式）
- **sas_sop_expert_v2.py** — SOP 专家每日优化
- **sas_daily_check.py** — 每日系统巡检
- **sas_github_sync.py** — GitHub 同步
- **sync_session_memory.py** — 会话记忆同步
- **health_check.py** — 健康检查（6项矩阵）
- **context_manager.py** — 上下文管理
- **lossless_claw** — DAG无损压缩
- **agent_tracker** — Agent性能追踪（⚠️未接入spawn）

### Cron 定时任务（3个，全部运行正常）
| ID | 名称 | 触发时间 | 状态 |
|----|------|---------|:---:|
| 16f0b64d | 每日会话记忆同步 | 每日02:00 | ✅ ok |
| d2a15c20 | 贵阳天气每日播报 | 每日 | ✅ ok |
| 0c34fb39 | 每日科技资讯推送 | 每日 | ✅ ok |

## 📈 构建完成度

| 模块 | 进度 | 说明 |
|------|------|------|
| 准则文档 | 100% | v1.0 → v1.6 完整演进 |
| OpenClaw 集成 | 95% | 19 Agent + SOUL.md + alsoAllow |
| 自动化脚本 | 100% | 10 个脚本全部就位 |
| 定时任务 | 100% | 3 个 cron 全部运行 |
| **sas-engine 插件** | **v1.1.0** | ✅ 已发布，4个工具可用 |

## 📜 License

内部项目，仅供 SnoopyClaw 团队使用。

---

_由 SnoopyClaw 🐾🦞 维护 · 基于 OpenClaw 平台 · "慢即是快"_
