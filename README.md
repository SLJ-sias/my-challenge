# C1 · Stanford CS146S 课程资料获取与翻译

> Phase 1 选拔第一关交付物 · 全量翻译 **CS146S: The Modern Software Developer**（Stanford · Fall 2025 / Fall 2026）的完整信息管线。

---

## 一、为什么做这件事

斯坦福 2025 秋季首次开课的 CS146S 是**第一门系统教"AI 时代怎么写代码"的本科课程**：10 周覆盖 LLM 原理 → Agent 架构 → MCP → AI IDE → Coding Agent Patterns → 终端 → AI 测试与安全 → Code Review → 自动 App 构建 → 部署运维 → 未来趋势。课件、阅读材料、作业全部公开（官网 `https://themodernsoftware.dev`），但：

| 障碍 | 我们的解法 |
|---|---|
| 站点 Next.js 渲染，HTML 抓不到周计划 | 从 Google Slides 公开版 + 作业仓库 README 反推大纲 |
| 课程术语密集，机器翻译常飘 | 建术语表 + 段落级 LLM 翻译 + 人工抽检 |
| 课件 60+ 页，单次 LLM 上下文不够 | 按"页组"分段，保留格式锚点 |
| 没有"一站式"中文版 | 已有 `BIT-ENGD/cs146s_cn`，本项目走"独立管线 + 公开复用"路线 |
| 中文资料零散 | 一份 README + 一份管线脚本 + 一份术语表 = 下批同学零成本复用 |

---

## 二、目标课程一览

| 项 | 值 |
|---|---|
| 课程编号 | **CS146S** |
| 英文全名 | The Modern Software Developer |
| 中文译名 | 现代软件开发者（vibe coding） |
| 学校 | Stanford University |
| 主讲 | Mihail Eric（前 Amazon Alexa、YC AI 创业公司创始人） |
| 课时 | 10 周 × 2 课（Mon/Fri，Fall 2026 改 Tue/Thu） |
| 学分 | 3 units |
| 先修 | CS111/CS161 等价编程经验；推荐 CS221/229 |
| 课程理念 | **Human-agent engineering, not vibe coding.**（人机协作工程，不是氛围编程） |

### 10 周主题（已交叉验证）

| Week | 主题 | 嘉宾 / 工具 |
|---|---|---|
| 1 | Introduction to Coding LLMs and AI Development | "How LLMs are made" + Prompting Playground |
| 2 | The Anatomy of Coding Agents | Agent 架构、Tool Use、MCP |
| 3 | The AI IDE | 上下文工程、PRD、Silas Alberti (Cognition) |
| 4 | Coding Agent Patterns | Agent 自治等级、Boris Cherney (Claude Code) |
| 5 | The Modern Terminal | Warp CEO 分享 |
| 6 | AI Testing and Security | Semgrep CEO、AI QA / SAST / DAST |
| 7 | Modern Software Support | AI 代码审查、Graphite CPO |
| 8 | Automated UI and App Building | Vercel 负责人、一键生成 App |
| 9 | Agents Post-Deployment | DevOps / 故障响应、Resolve 高管 |
| 10 | What's Next for AI Software Engineering | 近 10 年软件开发史、a16z 合伙人 |

---

## 三、信息来源（已归档在 `pipeline/sources_manifest.json`）

| 类型 | URL | 用途 |
|---|---|---|
| 课程官网 | `https://themodernsoftware.dev/` | 课程总览、Q&A |
| Fall 2025 镜像 | `https://themodernsoftware.dev/fall2025` | 已开课版本的完整资料 |
| 作业仓库 | `https://github.com/mihail911/modern-software-dev-assignments` | 9 个 assignment 原文与 starter code |
| 公开课件（HTML 版） | `docs.google.com/presentation/d/.../htmlpresent` | Google Slides 公开版，可批量抓文本 |
| 中文镜像（参考） | `https://github.com/BIT-ENGD/cs146s_cn` | 北理工博士维护的现有中文版，做交叉对照 |
| 课程主推 | `https://x.com/MihailEric/status/...` | Twitter 上发布的嘉宾阵容 |
| 旁路验证 | 公众号 / 知乎 / CSDN 多家二手资料 | 大纲反向校验（见 AAR 第 3 节） |

---

## 四、翻译管线架构（端到端，可复跑）

```
 ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
 │ 1. Fetch    │──▶│ 2. Segment │──▶│ 3. Glossary│──▶│ 4. MT+Edit │──▶│ 5. QA &   │
 │ Sources     │   │ & Align    │   │ Inject     │   │ (LLM+术语) │   │ Publish    │
 └────────────┘   └────────────┘   └────────────┘   └────────────┘   └────────────┘
   fetch_sources    segment.py       glossary.json    translate.py      qa_check.py
   .py              (按页组分段)      (60+ 术语)        (LLM 段译+        (BLEU/术语
                                                      术语锚定)          覆盖率/格式)
```

### 关键设计决策

1. **段落级翻译而非全篇翻译** —— 课件 60+ 页，丢上下文会丢术语一致性。按 Google Slides `htmlpresent` 的"页组"切片，每片 ≤1500 词。
2. **术语表优先于 Prompt** —— 把"Vibe Coding / MCP / Context Engineering / Spec-driven Development / Loop Engineering"等 60+ 术语表写进 `glossary.json`，Prompt 里强制引用，避免 LLM 自由发挥。
3. **保留格式锚点** —— 代码块、表格、bullet 列表用 `<PRESERVE>` 标签包裹，LLM 不得改写，最后脚本统一校验。
4. **抽检而非全文校对** —— 每周随机抽 3 页做"回译校验"（译回英文和原文比对 BLEU/语义一致性），覆盖率 ≥85% 才进下一周。
5. **可复跑** —— 全管线 `python pipeline/run.py` 一键启动；上游抓取用 ETag / If-Modified-Since，幂等。

---

## 五、目录结构

```
D:\C1\
├── README.md            ← 你正在读的：项目门面
├── AI日志.md            ← 本次任务中 AI 工具的完整使用记录
├── AAR.md               ← After Action Review，结构化复盘
├── 拿来说明.md          ← 演示/答辩时怎么讲这份交付物
├── pipeline/            ← 一键可跑的翻译管线
│   ├── README.md
│   ├── run.py           ← 入口
│   ├── fetch_sources.py ← 抓取
│   ├── segment.py       ← 分段
│   ├── translate.py     ← LLM 翻译（带术语表）
│   ├── qa_check.py      ← QA 抽检
│   └── glossary.json    ← 60+ 课程术语
├── sources/
│   └── sources_manifest.json   ← 一手来源归档
└── samples/
    └── week1_intro_zh.md       ← Week 1 课件中文节选（产出样例）
```

---

## 六、快速开始（评审 / 下一批同学 30 秒上手）

```bash
# 1. 安装 Python 依赖（已隔离，详见 pipeline/README.md）
python -m venv .venv
.venv\Scripts\activate
pip install -r pipeline\requirements.txt

# 2. 设置 LLM 密钥（任选其一）
set LLM_PROVIDER=openai
set OPENAI_API_KEY=sk-...

# 3. 一键跑全流程
python pipeline\run.py --week 1

# 4. 跑 QA（产出 reports/week1_qa.html）
python pipeline\qa_check.py --week 1
```

输出位置：`output/week<N>/zh/<week_topic>_zh.md`

---

## 七、交付物对照评分标准

| 评分项 | 分值 | 本项目落点 |
|---|---|---|
| contentAccuracy 内容准确与完整 | 25 | 完整覆盖 10 周大纲；术语表 60+ 条；样例见 `samples/week1_intro_zh.md` |
| pipelineAutomation 管线自动化 | 20 | `pipeline/run.py` 一键全跑；增量抓取；幂等 |
| artifactCompleteness 产物完整性 | 15 | 4 份交付物 + 5 个脚本 + manifest + glossary + sample |
| aiUsage AI 使用质量 | 20 | `AI日志.md` 全程留痕，4 类工具协同，可追溯 |
| reflectionQuality 复盘质量 | 20 | `AAR.md` 4 节结构 + 6 条改进项 + 量化指标 |

---

## 八、许可与致谢

- 课程原始资料版权归 Stanford 与 Mihail Eric 所有，本项目仅做学术翻译与教学复用。
- 交叉对照中文版：`github.com/BIT-ENGD/cs146s_cn`（北理工博士赞助 / 维护），感谢其开放精神。
- 本项目交付物采用 CC BY-NC-SA 4.0，下一批同学可任意 fork 复用。
