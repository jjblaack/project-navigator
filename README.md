# project-navigator

project-navigator 是一个“长期记忆索引层”Skill：用项目根目录的一份 MAP.md 作为唯一入口，给 Agent 提供全局视角（背景/哲学/模块关系/知识位置），并强制先读地图再跳转到目标文档，避免在仓库中盲目搜索。

## 核心思想
- 极简原则：不引入复杂推理规则；只维护一份 MAP.md 作为索引入口。
- 舞台模式：MAP.md 提供“该去哪读”的线索，Agent 依据语义自行推理并直达对应文件。
- 长期可演进：通过脚本自动维护索引，文档越多、地图越准，越用越懂。

## 目录与内容约定
- 文档集中存放在单一目录（默认 `./docs`，可通过脚本参数修改）。
- MAP.md 位于项目根目录 `./MAP.md`，包含：
  - 生成元信息（生成时间、更新命令、摘要规则、忽略规则等）
  - 分区索引（Philosophy / Architecture / Conventions / Playbooks / Reference / Misc）
  - 条目格式：`- [标题](相对路径) - Summary (tags: ...)`
- 每份文档建议包含最小元数据：

```markdown
---
description: 一句话说明本文用途与适用场景（建议 ≤ 80 字）
tags:
  - philosophy
  - architecture
---

# 文档标题
```

## 使用方法（对话流程）
当用户提到项目逻辑/架构/哲学/跨模块任务时，按以下顺序：
1. 读取项目根目录 `./MAP.md`
2. 根据条目标题与 Summary 选择最相关的目标文档
3. 直接读取目标文档（严禁先盲搜）
4. 如果 MAP.md 不存在/过期/缺漏：先运行维护脚本刷新地图，再回到步骤 1

## 维护脚本（自动更新 MAP.md）
脚本位置：
- `./.trae/skills/project-navigator/scripts/maintain_map.py`

默认生成/更新：

```bash
python3 ./.trae/skills/project-navigator/scripts/maintain_map.py
```

常用参数：

```bash
python3 ./.trae/skills/project-navigator/scripts/maintain_map.py \
  --docs-dir ./docs \
  --map-file ./MAP.md \
  --lang-sections \
  --max-per-category 200 \
  --summary-max-len 80
```

其他能力：
- 自定义忽略规则（可重复传参）：`--ignore '**/drafts/**'`
- 仅预览不落盘：`--dry-run`
- 只读取每个文件的前 N 行提取元信息（提升大仓库性能）：`--max-head-lines 200`
- 不在条目尾部展示 tags：`--no-tags`

## 团队集成建议
- 建议在 CI 或 pre-commit 中运行维护脚本，确保 MAP.md 与 docs 同步更新：

```bash
python3 ./.trae/skills/project-navigator/scripts/maintain_map.py
```

## 评估用例（Evals）
- 场景 A：用户问“我们要增加一个新模块，请根据项目哲学告诉我应该参考哪个现有文档？”
  - 期望：先读 MAP → 选 Philosophy/Architecture 相关条目 → 读目标文档 → 给出引用与建议
- 场景 B：用户说“我刚写完了一份关于新特性的哲学文档，请同步到地图中。”
  - 期望：运行脚本刷新 MAP → 再读 MAP 验证条目出现 → 返回直达路径

