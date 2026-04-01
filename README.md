# Snoopy Agent Smart (SAS) 工作准则

> **SAS = Snoopy Agent Smart** — AI 多智能体协作工作准则  
> **核心理念**：慢即是快 — 计划先行，逻辑清晰，有可执行计划，有可落地方案，有效评估周期，可控资源成本  
> **当前版本**：v1.6.0  
> **维护者**：SnoopyClaw (SC) · 2026-03-18 至今

---

## 📖 什么是 SAS？

SAS 是一套运行在 [OpenClaw](https://github.com/openclaw/openclaw) 平台上的 **AI 智能体协作工作准则**，定义了多个 Agent 如何分工、审批、记录和迭代。

它不是一个软件，而是一套**方法论 + 自动化脚本 + 插件引擎**的组合体系。

## 🗂️ 仓库结构

```
SAS/
├── README.md                          ← 你在这里
├── 大模型信息与Agent分配依据.md           ← 📊 Provider + Agent 分配矩阵 + 决策树
│
├── 01-需求调研/                        ← 📋 第一阶段：需求调研
│   ├── Snoopy Agent Smart (SAS)需求调研-deepseek.md    # DeepSeek 调研
│   ├── 通用需求调研提示词.md                            # 通用调研模板
│   ├── Gemini-2.0-pro调研/                             # Gemini 调研
│   └── deepseek-v3调研/                                # DeepSeek-V3 调研
│
├── 02-可行性分析/                      ← 🔬 第二阶段：可行性分析
│   ├── Snoopy Agent Smart (SAS) 可行性分析报告.md       # 完整可行性报告（52KB）
│   └── 迭代智能体配置与工作规范.md                       # 技术架构规范
│
├── 03-实施计划/                        ← 📐 第三阶段：实施计划
│   └── Snoopy Agent Smart (SAS) 实施计划.md             # 全量实施计划（32KB）
│
├── 04-工作准则/                        ← 📏 核心文档：SAS 准则各版本
│   ├── 工作准则-v1.0.md                                   # v1.0 基础版
│   ├── Snoopy Agent Smart (SAS)工作准则-v1.4.md           # v1.4 六阶段门控
│   ├── Snoopy Agent Smart (SAS)工作准则-v1.5.md           # v1.5 CEO模式
│   └── Snoopy Agent Smart (SAS)工作准则-v1.6.md           # v1.6 数据驱动 ⭐ 最新
│
├── 05-构建与验证/                      ← 🔧 第四阶段：构建与验证
│   ├── SAS-v1.5-完整建设方案.md                          # v1.5 完整建设方案
│   ├── SAS-V1.5-构建实录.md                              # v1.5 构建过程记录
│   ├── SAS-V1.5-验证报告.md                              # v1.5 验证结果
│   └── SAS-V1.5-构建完成报告.md                          # v1.5 交付总结
│
└── 06-迭代记录/                        ← 📝 版本迭代记录
    ├── SAS-迭代记录-v1.1.0.md
    ├── SAS-迭代记录-v1.2.0.md
    ├── SAS-迭代记录-v1.3.0.md
    └── SAS-迭代记录-v1.6.0.md
```

## 📊 版本历史

| 版本 | 日期 | 核心升级 | 状态 |
|------|------|---------|------|
| **v1.0** | 2026-03-19 | 基础准则，AI协作规范 | ✅ 归档 |
| **v1.1** | 2026-03-22 | 记忆系统，持续学习 | ✅ 归档 |
| **v1.2** | 2026-03-25 | 质量体系，代码规范 | ✅ 归档 |
| **v1.3** | 2026-03-29 | 异常监控与恢复 | ✅ 归档 |
| **v1.4** | 2026-03-31 | 六阶段门控流水线 | ✅ 归档 |
| **v1.5** | 2026-04-01 | CEO模式协作架构 | ✅ 已完成 |
| **v1.6** | 2026-04-01 | 数据驱动与风险前置 | 🔄 最新 |

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
│  HR       │  小文     │  天气    │   sas-default       │
│  (GLM-5)  │ (DeepSeek)│(GLM-4.5)│  (MiniMax)         │
├──────────┴──────────┴──────────┴─────────────────────┤
│              L3 sas-leader（自治）                    │
│     独立子任务 → 自主协调 → 完成交付                   │
└─────────────────────────────────────────────────────┘
```

## 🔄 六阶段工作流程

```
阶段1：接收任务    → 用户输入 → SC 接收
阶段2：制定计划    → L1/L2/L3 分级 → 用户审批
阶段3：执行工作    → SC 协调 → spawn 子任务
阶段4：质量检查    → 自检 → 验证 → 复核
阶段5：交付成果    → 完整交付物 + 使用说明
阶段6：复盘归档    → 记录决策 → 经验沉淀
```

## 🤖 Agent 注册（9个）

| Agent | 模型 | 职责 | Workspace |
|-------|------|------|-----------|
| **main (SC)** | MiniMax-M2.7-highspeed | 主脑：协调、规划、汇报 | workspace |
| **sas-leader** | MiniMax-M2.7 | 自治任务协调 | workspace-sas-leader |
| **sas-default** | MiniMax-M2.7-highspeed | 通用子任务 | workspace-sas-default |
| **sas-sop-expert** | MiniMax-M2.7 | 准则优化 | workspace-sas-sop-expert |
| **xiaowen (小文)** | DeepSeek Chat | 文档、代码 | workspace-xiaowen |
| **hr** | GLM-5-Turbo | 招聘、团队管理 | workspace-hr |
| **tech-news** | GLM-5-Turbo | 科技资讯推送 | workspace-tech-news |
| **doc-expert** | GLM-4.7 | 文档生成 | workspace-doc-expert |
| **weather** | GLM-4.5-Air | 天气播报 | workspace-weather |

## 🔗 关联仓库

| 仓库 | 说明 | 链接 |
|------|------|------|
| **SAS** | 本仓库 — 准则文档 | [terlivy/SAS](https://github.com/terlivy/SAS) |
| **SAS-script** | 自动化脚本（12个） | [terlivy/SAS-script](https://github.com/terlivy/SAS-script) |
| **SAS-plug-in** | sas-engine 插件（开发中） | [terlivy/SAS-plug-in](https://github.com/terlivy/SAS-plug-in) |
| **snoopyclaw-skills** | SC 自建技能 | [terlivy/snoopyclaw-skills](https://github.com/terlivy/snoopyclaw-skills) |

## 🛠️ 自动化基础设施

- **task_decompose.py** — 任务自动拆解（规则引擎，L1/L2/L3 分级）
- **task_logger.py** — 任务日志记录（JSONL 格式）
- **sas_sop_expert_v2.py** — SOP 专家每日优化
- **sas_daily_check.py** — 每日系统巡检
- **sas_alert_system.py** — 告警系统
- **sas_github_sync.py** — GitHub 同步

## 📈 构建完成度

| 模块 | 进度 | 说明 |
|------|------|------|
| 准则文档 | 100% | v1.0 → v1.6 完整演进 |
| OpenClaw 集成 | 90% | 9 Agent + SOUL.md + alsoAllow |
| 自动化脚本 | 100% | 12 个脚本全部就位 |
| 定时任务 | 100% | 天气 + 资讯 + SOP优化 |
| 插件引擎 | 5% | sas-engine 框架存在，核心未开发 |

## 📜 License

内部项目，仅供 SnoopyClaw 团队使用。

---

_由 SnoopyClaw 🐾🦞 维护 · 基于 OpenClaw 平台 · "慢即是快"_
