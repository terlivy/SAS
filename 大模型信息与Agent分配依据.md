# 大模型信息与 Agent 分配依据

> **最后更新**：2026-04-01  
> **维护者**：SnoopyClaw (SC)  
> **用途**：记录所有可用模型的能力、成本、适用场景，以及每个 Agent 的模型分配决策依据

---

## 一、Provider 总览

| Provider | API 来源 | 计费方式 | 模型数量 | 状态 |
|----------|---------|---------|---------|------|
| **MiniMax** | 官方 API | Token Plan（月度额度） | 6 | ✅ 主力 |
| **智谱 GLM (zai)** | 官方 API | 资源包（预付费） | 6+ | ✅ 备用 |
| **DeepSeek** | 官方 API | 按量付费（¥/百万token） | 2 | ✅ 编程专用 |
| **硅基流动 SiliconFlow** | 第三方聚合 | 免费额度 | 10+ | ✅ 免费备用 |

---

## 二、模型详细参数

### 2.1 MiniMax（主力 Provider）

| 模型 | 别名 | 上下文 | 速度(TPS) | 特长 | 分配 |
|------|------|--------|----------|------|------|
| MiniMax-M2.7 | M2.7 | 200K | 60 | 复杂推理、规划、长文 | SAS-leader |
| MiniMax-M2.7-highspeed | M2.7-Fast | 200K | 100 | 极速任务、日常对话 | **SC 主脑**、sas-default |
| MiniMax-M2.5 | M2.5 | 200K | 中 | 复杂推理 | 备用 |
| MiniMax-M2.5-highspeed | M2.5-Fast | 200K | 快 | 通用 | 备用 |
| MiniMax-M2.1 | M2.1 | 200K | 快 | 通用 | 备用 |
| MiniMax-M2 | M2 | 200K | 快 | 通用 | 备用 |

**计费**：Token Plan 月度套餐，固定额度，用完停止（无超额费用）

**分配依据**：
- SC 主脑用 highspeed 版本——日常对话+规划任务，速度优先
- sas-leader 用标准版——需要更强的推理能力来协调复杂子任务
- MiniMax 对 OpenClaw 的工具调用（tool calling）支持良好

---

### 2.2 智谱 GLM（备用 Provider）

| 模型 | 上下文 | 速度 | 计费 | 余额 | 到期 | 分配 |
|------|--------|------|------|------|------|------|
| **GLM-5-Turbo** | 200K | 中 | 资源包（预付） | ~1261万 tokens | **2026-04-18** | HR、SAS-SOP专家 |
| GLM-4.7 | 128K | 中 | 赠送 | ~179万 tokens | 2026-06-18 | 文档专家（备用） |
| GLM-4.5-Air | 128K | 快 | 赠送×2 | ~2033万 tokens | 2026-06-17 | 天气、资讯 |
| GLM-4.1V-Thinking-FlashX | 128K | 快 | 资源包 | ~1000万 tokens | 2026-04-19 | 视觉推理 |
| GLM-4.6V | 128K | 快 | 赠送 | ~598万 tokens | 2026-06-17 | 视觉任务（备用） |
| GLM-4.5-Air（搜索） | - | - | 赠送 | 99次 | 2026-06-17 | 搜索任务 |

**分配依据**：
- GLM-5-Turbo 深度定制支持 OpenClaw——作为 HR 和 SAS-SOP 专家的首选，这类 Agent 需要 OpenClaw 深度集成能力
- GLM-4.5-Air 余额充足（2033万），分配给简单任务的 Agent（天气、资讯）
- GLM-4.7 赠送到 06-18，作为文档专家备用
- ⚠️ GLM-5-Turbo 04-18 到期，**需提前续费或迁移到 MiniMax**

**注意事项**（terlivy 评价）：
- GLM-5 Turbo 逻辑偏弱，"脑袋还是智谱的逻辑"
- 国内拼逻辑只有 DeepSeek 能碰 Claude，核心是 QWEN 和 Opus

---

### 2.3 DeepSeek（编程专用）

| 模型 | 上下文 | 输入价格 | 输出价格 | 分配 |
|------|--------|---------|---------|------|
| **DeepSeek Chat (V3)** | 128K | ¥1/百万token | ¥2/百万token | **小文、编程子智能体** |
| DeepSeek Reasoner (R1) | 64K | ¥4/百万token | ¥16/百万token | 复杂推理（按需） |

**分配依据**：
- DeepSeek Chat 编程能力强（function calling 支持），性价比高
- 所有编码任务（前后端开发、脚本编写）统一使用 DeepSeek
- R1 推理能力强但贵 8 倍，仅在需要深度推理时按需使用
- 预估月费用：~¥10（日常编码场景）

---

### 2.4 硅基流动 SiliconFlow（免费备用）

| 模型 | 上下文 | 价格 | 状态 |
|------|--------|------|------|
| DeepSeek-V3 | 128K | 免费 | ✅ 可用 |
| Qwen3.5-397B | 128K | 免费 | ✅ 可用 |
| Kimi-K2.5 | 128K | 免费 | ✅ 可用 |
| DeepSeek-V3.2 | 128K | 免费 | ✅ 可用 |
| Qwen3-Coder-480B | 128K | 免费 | ✅ 可用 |

**分配依据**：
- 纯免费，作为降级方案——当主 Provider 不可用时切换
- 不作为主力，因为免费额度有限且稳定性不可控
- 适合低风险任务：文档翻译、格式转换、简单查询

---

## 三、Agent 模型分配矩阵

| Agent | 当前模型 | 分配依据 | 可降级到 |
|-------|---------|---------|---------|
| **SC（main）** | MiniMax-M2.7-highspeed | 主脑：复杂推理+规划+日常对话，速度优先 | GLM-5-Turbo |
| **sas-leader** | MiniMax-M2.7 | 自治协调：需要强推理能力协调多 Agent | MiniMax-M2.5 |
| **sas-default** | MiniMax-M2.7-highspeed | 通用子任务：与主脑同规格保持一致性 | GLM-5-Turbo |
| **sas-sop-expert** | MiniMax-M2.7 | 准则优化：需要分析+写作能力 | GLM-5-Turbo |
| **小文（xiaowen）** | DeepSeek Chat | 文档处理+代码生成：DeepSeek 编程强 | GLM-4.7 |
| **HR** | GLM-5-Turbo | 招聘+团队管理：需要 OpenClaw 深度集成 | GLM-4.5-Air |
| **tech-news** | GLM-5-Turbo | 科技资讯：需要 web_fetch + OpenClaw 集成 | GLM-4.5-Air |
| **doc-expert** | GLM-4.7 | 文档生成：GLM 中文能力强 | GLM-4.5-Air |
| **weather** | GLM-4.5-Air | 天气查询：简单任务，用低成本模型 | SiliconFlow 免费 |

---

## 四、模型选择决策树

```
需要选择模型？
├── 编码/开发任务 → DeepSeek Chat
├── 复杂推理/规划 → MiniMax-M2.7
├── 日常对话/协调 → MiniMax-M2.7-highspeed
├── OpenClaw 深度集成 → GLM-5-Turbo
├── 简单查询/低成本 → GLM-4.5-Air
├── 中文文档生成 → GLM-4.7
├── 免费降级 → SiliconFlow DeepSeek-V3
└── 深度推理（不惜成本）→ DeepSeek R1
```

---

## 五、成本监控

### 月度预算（预估）

| Provider | 预估月费 | 说明 |
|----------|---------|------|
| MiniMax | Token Plan（固定） | 月度套餐，不按量 |
| DeepSeek | ~¥10 | 编码子智能体日常使用 |
| GLM | ¥0（资源包内） | 注意 04-18 GLM-5-Turbo 到期 |
| SiliconFlow | ¥0 | 免费 |

### 到期日历

| 日期 | 到期项 | 行动 |
|------|--------|------|
| **2026-04-18** | GLM-5-Turbo 资源包 | 🔴 续费或迁移到 MiniMax |
| 2026-04-19 | GLM-4.1V-Thinking-FlashX | 🟡 评估是否续费 |
| 2026-06-17 | GLM-4.5-Air + GLM-4.6V + 搜索 | 🟢 暂不处理 |

---

## 六、历史决策记录

| 日期 | 决策 | 依据 |
|------|------|------|
| 2026-03-19 | GLM-5-Turbo 作为 SC 主脑 | 资源包免费，OpenClaw 深度定制 |
| 2026-03-24 | DeepSeek Chat 作为编程专用 | 编程能力强，function calling，¥1/¥2 便宜 |
| 2026-03-27 | SC 主脑切换到 MiniMax-M2.7-highspeed | 速度更快（100 TPS），Token Plan 成本可控 |
| 2026-03-27 | 小文从 GLM-4.7 改为 DeepSeek Chat | 编码需求增加，DeepSeek 更适合 |
| 2026-04-01 | sas-leader 用 MiniMax-M2.7 标准版 | 需要最强推理来协调子任务 |

---

_本文档由 SnoopyClaw 维护，模型变更时同步更新。_
