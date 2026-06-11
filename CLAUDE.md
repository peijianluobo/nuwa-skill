# CLAUDE.md

此文件为 Claude Code 在此仓库中工作时提供指南。

## 语言偏好

始终使用中文回复。所有解释、注释、沟通都使用中文，代码标识符和技术术语保留原文。

## 项目说明

**女娲.skill** 是一个 Agent Skill（元技能），输入人名/主题/模糊需求，自动深度调研 → 思维框架提炼 → 生成可运行的人物视角 Skill。基于 [Agent Skills 协议](https://agentskills.io)，跨 Claude Code、Cursor、Codex 等 50+ runtime 运行。

核心理念：蒸馏的是 **HOW they think**，不是 WHAT they said。提取的是心智模型、决策启发式、表达 DNA、价值观/反模式、诚实边界。

## 项目结构

```
nuwa-skill/
  SKILL.md                 # 技能定义入口（YAML 前页 + 645 行 6 阶段流程）
  LICENSE                  # MIT
  README.md                # 中文主 Readme（多语言版本：README_EN/ES/JA/KO.md）
  examples/                # 15 个已生成的 Skill 示例
  scripts/                 # 流程辅助脚本
    download_subtitles.sh  #   用 yt-dlp 下载 YouTube 字幕
    srt_to_transcript.py   #   SRT/VTT → 纯文本
    merge_research.py      #   扫描 6 个研究文件，生成 Phase 1.5 审查汇总表
    quality_check.py       #   Phase 4 6 项自动质检
  assets/                  # 品牌素材（hero.gif, banner.svg）
```

## 核心执行流程（6 阶段）

`SKILL.md` 定义了完整的端到端流程：

| 阶段 | 名称 | 说明 |
|---|---|---|
| 0 | 入口分流 | 明确人名→直接路径；模糊需求→诊断→推荐候选人 |
| 0.5 | 目录创建 | 在 `.claude/skills/[name]-perspective/` 下建立 Skill 骨架 |
| 1 | 并行调研 | 6 个 sub-agent 同时运行（著作、对话、表达 DNA、外部评价、决策、时间线） |
| 1.5 | 审查检查点 | 展示汇总表（矛盾数、信息缺口），用户确认后继续 |
| 2 | 框架提炼 | 三重验证提取心智模型、决策启发式、表达 DNA、价值观/反模式 |
| 2.5 | 提炼确认 | 提取摘要确认 |
| 3 | Skill 构建 | 基于 `skill-template.md` 填入所有提取结果，生成 Agentic 协议 |
| 4 | 质量验证 | 3 题验证 + 1 边界测试 + 6 项 PASS/FAIL 质检（最多 2 轮迭代） |
| 5 | 双代理精炼 | auto-skill-optimizer + skill-creator 并行审查改进 |

### 调研来源优先级

用户的本地语料 > 作者本人著作 > 长篇对话 > 决策记录 > 社交媒体 > 他人评价 > 二手总结

### 中文人物特殊处理

中文人物切换到 B站/小宇宙/权威中文媒体作为来源。
黑名单来源：知乎、微信公众号、百度百科。

## 关键脚本用法

```bash
# 下载 YouTube 字幕（手动→自动英→自动中 降级策略）
bash scripts/download_subtitles.sh <youtube-url> [output-dir]

# SRT/VTT 转纯文本
python3 scripts/srt_to_transcript.py input.srt [output.txt]

# Phase 1.5 审查汇总（扫描 references/research/01-06.md）
python3 scripts/merge_research.py <skill-directory>

# Phase 4 6 项自动质检（对生成的 SKILL.md 评分）
python3 scripts/quality_check.py <SKILL.md-path>
```

## examples/ 示例结构

### 标准人物 Skill（6 研究文件）
`examples/steve-jobs-perspective/`（以及 Karpathy、Ilya、Paul Graham、Trump、Taleb、张一鸣、张雪峰等）：
```
SKILL.md
references/research/
  01-writings.md           # 著作与系统思考
  02-conversations.md      # 长篇对话
  03-expression-dna.md     # 表达风格 DNA
  04-external-views.md     # 他人评价与批评
  05-decisions.md          # 重大决策记录
  06-timeline.md           # 生平时间线
```

### 带脚本的人物 Skill
`examples/mrbeast-perspective/` 额外包含 `scripts/`：`analyze_titles.py`、`retention_curve_checker.py`、`thumbnail_audit.py`、`fetch_youtube_subtitles.sh`

### 主题 Skill（非人物）
`examples/x-mastery-mentor/`：角色扮演改为「问题路由表 + 框架总览 + 学派对比」，按场景加载参考文件。

### 简化人物 Skill
`examples/elon-musk-perspective/`、`munger-perspective/`、`naval-perspective/`、`feynman-perspective/`：参考文件扁平存放于 `references/` 下，使用中文命名。

## 生成的 SKILL.md 结构

每个生成的 Skill 含以下固定组件：
1. YAML 前页（name、description、触发条件）
2. 角色扮演规则（第一人称、STOP/EXIT 触发器）
3. **Agentic 协议**（问题分类 → 研究维度 → 检查点 → 回答格式）
4. 身份卡片（50 字第一人称自我介绍）
5. 心智模型（3-7 个，每个含证据/应用/局限）
6. 决策启发式（5-10 个 if-then 规则，含案例）
7. 表达 DNA（句式、词汇、节奏、幽默、确定性、类比、引用习惯）
8. 时间线（关键事件 + 对思维的影响 + 近 12 个月更新）
9. 价值观与反模式
10. 思想谱系（影响来源 → 自身 → 影响后人）
11. 诚实边界（明确不能做什么）
12. 附录来源 + 创作者署名

## 无构建/测试

本项目无代码依赖、无构建系统、无传统测试。它是纯 Agent Skill 定义（Markdown + YAML）。质量保证通过 SKILL.md 中定义的 Phase 4 6 项检查 + `scripts/quality_check.py` 脚本实现。
