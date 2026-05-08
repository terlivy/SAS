# Graph Report - .  (2026-04-15)

## Corpus Check
- 22 files · ~38,064 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 22 nodes · 17 edges · 7 communities detected
- Extraction: 100% EXTRACTED · 0% INFERRED · 0% AMBIGUOUS
- Token cost: 0 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Community 0|Community 0]]
- [[_COMMUNITY_Community 1|Community 1]]
- [[_COMMUNITY_Community 2|Community 2]]
- [[_COMMUNITY_Community 3|Community 3]]
- [[_COMMUNITY_Community 4|Community 4]]
- [[_COMMUNITY_Community 5|Community 5]]
- [[_COMMUNITY_Community 6|Community 6]]

## God Nodes (most connected - your core abstractions)
1. `六阶段工作流程` - 7 edges
2. `SAS (Snoopy Agent Smart)` - 5 edges
3. `SC (SnoopyClaw 主脑)` - 2 edges
4. `MiniMax-M2.7-highspeed` - 2 edges
5. `阶段2: 制定计划` - 2 edges
6. `慢即是快` - 1 edges
7. `CEO 模式协作架构` - 1 edges
8. `main (SC)` - 1 edges
9. `sas-leader` - 1 edges
10. `sas-default` - 1 edges

## Surprising Connections (you probably didn't know these)
- `慢即是快` --references--> `SAS (Snoopy Agent Smart)`  [EXTRACTED]
  04-工作准则/Snoopy Agent Smart (SAS)工作准则-v1.5.md → README.md
- `CEO 模式协作架构` --part_of--> `SAS (Snoopy Agent Smart)`  [EXTRACTED]
  04-工作准则/Snoopy Agent Smart (SAS)工作准则-v1.5.md → README.md
- `OpenClaw 平台` --uses--> `SAS (Snoopy Agent Smart)`  [EXTRACTED]
  04-工作准则/Snoopy Agent Smart (SAS)工作准则-v1.5.md → README.md
- `memory-lancedb-pro` --uses--> `SC (SnoopyClaw 主脑)`  [EXTRACTED]
  05-构建与验证/SAS-V1.5-构建实录.md → README.md
- `main (SC)` --uses--> `MiniMax-M2.7-highspeed`  [EXTRACTED]
  README.md → 大模型信息与Agent分配依据.md

## Communities

### Community 0 - "Community 0"
Cohesion: 0.33
Nodes (6): CEO 模式协作架构, SAS (Snoopy Agent Smart), SC (SnoopyClaw 主脑), 慢即是快, memory-lancedb-pro, OpenClaw 平台

### Community 1 - "Community 1"
Cohesion: 0.33
Nodes (6): 阶段1: 接收任务, 阶段3: 执行工作, 阶段4: 质量检查, 阶段5: 交付成果, 阶段6: 复盘归档, 六阶段工作流程

### Community 2 - "Community 2"
Cohesion: 0.67
Nodes (3): main (SC), sas-default, MiniMax-M2.7-highspeed

### Community 3 - "Community 3"
Cohesion: 1.0
Nodes (2): sas-leader, MiniMax-M2.7

### Community 4 - "Community 4"
Cohesion: 1.0
Nodes (2): 阶段2: 制定计划, task_decompose.py

### Community 5 - "Community 5"
Cohesion: 1.0
Nodes (2): SAS v1.5, SAS v1.6

### Community 6 - "Community 6"
Cohesion: 1.0
Nodes (1): DeepSeek Chat

## Knowledge Gaps
- **17 isolated node(s):** `慢即是快`, `CEO 模式协作架构`, `main (SC)`, `sas-leader`, `sas-default` (+12 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `Community 3`** (2 nodes): `sas-leader`, `MiniMax-M2.7`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 4`** (2 nodes): `阶段2: 制定计划`, `task_decompose.py`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 5`** (2 nodes): `SAS v1.5`, `SAS v1.6`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Community 6`** (1 nodes): `DeepSeek Chat`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.