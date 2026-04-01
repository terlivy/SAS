# Snoopy Agent Smart (SAS)

> AI 智能体协作工作准则 — 让 AI 真正成为高效、可信的工作伙伴

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/terlivy/SAS?style=flat-square)](https://github.com/terlivy/SAS/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)

---

## 是什么

SAS（Snoopy Agent Smart）是一整套 **AI 智能体协作工作准则与架构规范**。

它解决的问题是：当你同时使用多个 AI Agent 时，如何避免它们各自为战、重复劳动、擅自做主、交付质量参差不齐？

SAS 通过**标准化的六阶段工作流程**、**四铁律**和**四层角色分工**，让多个 AI Agent 像一个训练有素的团队一样高效协作。

---

## 核心特性

### 六阶段工作流程

每一个任务都严格按照以下六个阶段推进：

```
① 接收任务  →  ② 制定计划  →  ③ 执行工作  →  ④ 质量检查  →  ⑤ 交付成果  →  ⑥ 归档记忆
```

| 阶段 | 关键动作 |
|------|---------|
| 接收任务 | 需求确认、复杂度评估、资源清点 |
| 制定计划 | 步骤拆解、风险识别、方案选型 |
| 执行工作 | 按计划执行、关键决策实时记录 |
| 质量检查 | 自检 → 验证 → 复核，三层把关 |
| 交付成果 | 完整交付物 + 使用说明 + 风险提示 |
| 归档记忆 | 关键决策入库、经验萃取、错误预防 |

### 执行铁律

Agent 在执行过程中必须严格遵守：

- **不擅自做主** — 超出权限或存在不确定因素时，必须上报
- **不隐瞒问题** — 发现风险立即预警，不等到爆发
- **不重复犯错** — 历史错误库是必查项，不是可选项
- **不浪费资源** — 选最合适的模型和工具，不盲目堆砌

### 四层角色分工（CEO 模式）

v1.5.0 引入的分层架构，职责清晰、权限分明：

| 层级 | 角色 | 职责 |
|------|------|------|
| L0 | 用户（CEO） | 只参与两个决策点：计划审批 + 方案审批 |
| L1 | 主脑 SC（项目经理） | 任务分解、主协调、对外沟通 |
| L2 | Leader（任务负责人） | Worker 分配、进度管理、问题处理 |
| L3 | Worker（执行者） | 按规范执行具体子任务 |

### 任务复杂度分级

| 等级 | 类型 | 处理流程 |
|------|------|---------|
| L1 | 简单 / 快速通道 | 压缩流程，快速交付 |
| L2 | 中等 / 标准流程 | 完整六阶段，标准审核 |
| L3 | 复杂 / 完整流程 | 全流程 + Leader 介入 + 用户深度参与 |

---

## 版本历史

| 版本 | 日期 | 核心更新 |
|------|------|---------|
| v1.5.0 | 2026-03-31 | CEO 模式架构：四层角色分工、权限矩阵、能力档案、任务级成绩单 |
| v1.4.0 | 2026-03-26 | 初始版本，六阶段流程 + 执行铁律框架 |

详细更新记录请查看 [迭代记录](./迭代记录/) 目录。

---

## 文件结构

```
SAS/
├── README.md                          # 本文件
├── Snoopy Agent Smart (SAS)工作准则-v1.5.md   # v1.5.0 完整准则（核心文档）
├── Snoopy Agent Smart (SAS)工作准则-v1.4.md  # v1.4.0 历史版本
├── Snoopy Agent Smart (SAS)可行性分析报告.md  # 项目可行性分析
├── Snoopy Agent Smart (SAS)实施计划.md        # 落地实施路线图
├── 迭代记录/                          # 版本迭代过程文档
└── 调研文档/                           # 调研过程中产生的参考文档
    ├── Gemini-2.0-pro调研/
    └── deepseek-v3调研/
```

---

## 快速开始

### 对于 AI Agent 开发者

将 `Snoopy Agent Smart (SAS)工作准则-v1.5.md` 中的核心指令嵌入你的 Agent 系统提示：

```
你是 {AgentName}，你严格遵循 SAS v1.5.0 工作准则。
- 工作流程：六阶段（接收任务→制定计划→执行工作→质量检查→交付成果→归档记忆）
- 执行铁律：不擅自做主、不隐瞒问题、不重复犯错、不浪费资源
- 你的角色：{L1/L2/L3}，权限范围：{具体权限}
```

### 对于团队管理者

参考 [实施计划.md](./Snoopy%20Agent%20Smart%20(SAS)实施计划.md) 制定适合你团队的落地路径。

---

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [SAS-plug-in](https://github.com/terlivy/SAS-plug-in) | SAS 引擎插件（阶段门控、自动审批） |
| [SAS-script](https://github.com/terlivy/SAS-script) | 10 个自动化脚本工具集 |
| [snoopyclaw-skills](https://github.com/terlivy/snoopyclaw-skills) | OpenClaw Agent 技能配置（Harness、Leader 等） |

---

## 免责声明

SAS 是一套工作方法论，不对 AI 生成内容的准确性、合法性、合规性负责。使用者在部署和应用过程中需自行判断风险，承担相应责任。

---

## 联系方式

- **GitHub Issues**: https://github.com/terlivy/SAS/issues
- **项目维护者**: terlivy

---

*SAS — 让 AI 协作有章可循*
