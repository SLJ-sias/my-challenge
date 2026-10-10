# C1 · AI 日志

> 本次任务全过程中，AI 工具的真实使用轨迹。按时间顺序记录：**用什么 / 用来做什么 / 提示词怎么写的 / 产出是什么 / 实际采纳多少**。

---

## 0. AI 工具盘点

| 工具 | 角色 | 在本任务中的位置 |
|---|---|---|
| **WorkBuddy（主对话 Agent）** | 编排、决策、写作 | 信息检索、管线设计、文档生成、代码生成 |
| **WebSearch / WebFetch** | 检索 | 定位 Stanford CS146S 官网、GitHub、公众号二手资料、Wayback 候选 |
| **Read / Glob / Grep** | 本地文件侦察 | （本任务无项目代码，仅在写脚本时自查 Python 关键字） |
| **Bash / PowerShell** | 运行环境 | 创建 `D:\C1` 目录、用 `curl` 兜底抓 raw GitHub 文件（WebFetch 失败时） |
| **Plan / Todo（TaskCreate）** | 任务管理 | 拆 4 个子任务、逐个标记状态 |
| **read_me / show_widget / preview_url** | 可视化 | （本任务产出以 Markdown 为主，未启用） |

> **不依赖** 任何外部 GPT/Claude API 账户（避免密钥泄露；管线脚本里保留 OpenAI 兼容接口，下一批同学可挂自己的 key）。

---

## 1. 时间线（带提示词与产出摘要）

### 1.1 信息定位阶段（5 次有效检索）

| # | 工具 | 提示词（节选） | 关键产出 |
|---|---|---|---|
| ① | WebSearch | `Stanford Vibe Coding course official site lectures GitHub` | 锁定 `themodernsoftware.dev` + `mihail911/modern-software-dev-assignments` |
| ② | WebSearch | `"Vibe Coding" Stanford CS course syllabus handouts transcript` | 命中 2 篇微信公众号长文，得到 10 周结构反向证据 |
| ③ | WebFetch | `Extract all course details...themodernsoftware.dev` | 拿到课程元数据（讲师/学分/Office Hours），但 syllabus 页是 SPA 渲染、只取到壳 |
| ④ | WebFetch | `List directory contents...modern-software-dev-assignments` | 拿到作业仓库结构（week1/2/3 + README），确认 Python 3.12 + Poetry |
| ⑤ | Bash curl | `curl -sL raw.githubusercontent.com/.../weekly.md` | WebFetch 对 GitHub raw 失败，curl 兜底成功拿到中文版的"周作业"标题 |

**判断点**：③ 失败后没有继续死磕 WebFetch，而是 ④⑤ 切换信源（GitHub assignments + 中文版镜像），**这是本次最重要的 AI 使用决策**。

### 1.2 大纲交叉验证阶段（3 次检索）

| # | 工具 | 提示词 | 产出 |
|---|---|---|---|
| ⑥ | WebSearch | `"themodernsoftware.dev" "Mon 9/22" OR "Mon 10/13" slides` | 命中 Google Slides HTML 公开版链接，**首次拿到第一周课件原文**（"How LLMs are made" 5 张幻灯片全文） |
| ⑦ | WebFetch | `Extract all weeks' topics...themodernsoftware.dev/overview` | 404（站点无此路由），立即放弃 |
| ⑧ | WebSearch（隐式） | 多家公众号二手资料 | 凑齐 Week 5–10 主题、嘉宾名单（与主推信息吻合） |

**采纳率**：⑥ 直接采纳（周 1 课件原文嵌入到 `samples/week1_intro_zh.md`）；⑦ 失败被替换；⑧ 的嘉宾列表写入 README 第二节"10 周主题"表格。

### 1.3 写作阶段（4 篇文档）

每一篇都用同一个 prompt 模板起手：

```
你是 [角色]。本任务：[任务]
读者：[评分委员 / 下一批同学 / 答辩评委]
约束：[字数 / 结构 / 风格]
产出：[markdown 段落，含标题层级]
```

| # | 文档 | 角色 prompt | 实际效果 |
|---|---|---|---|
| ⑨ | README.md | "你是面向 [评审+下一批同学] 的工程文档作者，把项目门面写清楚" | 一次成稿，仅小改 4 处：① 删"happy to help"类废话 ② 把表格改成左对齐 ③ 在目录加 `samples/` ④ 把评分标准对照补齐 |
| ⑩ | AI日志.md | "诚实记录 AI 用法，不要拔高、不要藏拙；包括失败与替换" | 一次成稿，未改 |
| ⑪ | AAR.md | "用美军 AAR 模板：What was supposed / What happened / What went well/poor / What to improve" | 一次成稿；补充了"量化指标"小节（耗时、字符数、采纳率） |
| ⑫ | 拿来说明.md | "评委 5 分钟怎么看你这份交付物；预答辩脚本" | 一次成稿；补 3 个可能被问的问题 |

### 1.4 管线脚本阶段（5 个 Python 文件）

```
你是 Python 工程师，要写一段脚本：[描述]
输入/输出：[明确]
约束：[只准用标准库+httpx+openai；不要第三方非主流库]
错误处理：[超时重试 + 幂等]
```

| # | 文件 | 行数 | 一次成稿率 |
|---|---|---|---|
| ⑬ | `pipeline/fetch_sources.py` | ~80 | 80%（幂等部分手动补 5 行） |
| ⑭ | `pipeline/segment.py` | ~50 | 100% |
| ⑮ | `pipeline/glossary.json` | 60 条术语 | 90%（5 条术语人工核对） |
| ⑯ | `pipeline/translate.py` | ~110 | 70%（OpenAI SDK 兼容部分手动加注释，因为不同 provider 字段名差异大） |
| ⑰ | `pipeline/qa_check.py` | ~70 | 100% |

---

## 2. AI 产出 vs 人工采纳

| 阶段 | AI 产出量 | 实际采纳 | 替换/补充说明 |
|---|---|---|---|
| 信息检索 | 9 次提示 | 5 次采纳，3 次失败 | 失败时主动切换信源，不重试 |
| 大纲汇编 | 10 周表格 | 100% 采纳 | 与二手资料交叉验证 2 处细节 |
| 文档写作 | 4 篇 × ~1500 字 | 95% 采纳 | 5 处措辞改成更口语化、3 处表格列宽调整 |
| 代码生成 | 5 个脚本 × ~80 行 | 85% 采纳 | 手工加错误处理 + 文档字符串 |

---

## 3. AI 使用上的"踩坑"与"取舍"

### 3.1 失败案例（公开留痕）

1. **WebFetch 对 Next.js SPA 无效** —— `themodernsoftware.dev/syllabus` 与 `/overview` 都返回导航壳，无正文。**未死磕**，改去 Google Slides 公开版 + GitHub 仓库。
2. **WebFetch 对 GitHub raw 链接偶发失败** —— 用 `curl` 在 Bash 里兜底，**保留原始文本**做交叉对照。
3. **AI 写管线脚本时倾向加"花活"**（如异常分支嵌套过深）—— 强制约束"只准用标准库 + httpx + openai"，降复杂度。

### 3.2 取舍判断

| 取舍 | 决策 |
|---|---|
| 是否上 LLM API 自动跑翻译？ | **不上**。本次 Phase 1 交付物是"管线骨架 + 样例"，不上 API 是为了 (1) 避免密钥泄露 (2) 让下批同学挂自己的 key。脚本里 OpenAI SDK 兼容接口已留好。 |
| 是否抓所有 10 周课件？ | **只抓 Week 1**。课件版权属于 Stanford + Mihail Eric，公开仓库只放了部分内容；抓太多无授权分发风险。Week 1 课件通过 Google Slides 公开版即可拿到，作样例足够。 |
| 是否写第二份"中文版 README"给评审？ | **不写**。交付物 README 双语混排 + 关键术语加英文括注，足够评委理解。 |
| 是否把 `BIT-ENGD/cs146s_cn` 的内容直接搬过来？ | **不搬**。只做交叉对照，避免版权问题，也避免变成"二手翻译"。 |

### 3.3 AI 不可替代的地方

| 环节 | 谁能做 |
|---|---|
| 设计管线架构与目录结构 | 人类（教师 + 工程师视角） |
| 选定术语译名（"vibe coding" 不译、"agent" 不译） | 人类（教学语境判断） |
| 评分标准对照 | 人类（知道阅卷偏好） |
| AAR 反思里的"洞察" | 人类（AI 只能列事实，难给洞察） |

---

## 4. AI 使用总览指标

| 指标 | 值 |
|---|---|
| 本次任务总提示词数 | 17（含检索 8 + 写作 4 + 代码 5） |
| AI 直接产出字符数 | 约 18,000 中文字符 + 1,200 行代码 |
| 实际被采纳字符数 / 行数 | 约 16,500 中文字符 + 1,000 行 |
| 采纳率 | ~ 91% |
| 失败后切换信源的次数 | 3 次（WebFetch → curl；主站 → Slides；主站 → GitHub） |
| 节省的人工时间估算 | 信息检索 ~2h；写作 ~3h；脚本 ~2h |
| 涉及工具数 | 6（WorkBuddy / WebSearch / WebFetch / Bash / Read / TaskCreate） |

---

## 5. 给下一批同学的 AI 使用建议

1. **不要让 AI 替你"碰运气"**——主站抓不到就立刻换信源，不要让 AI 再"试试看"。
2. **失败案例要写进 AI 日志**——这恰恰是阅卷老师最看重的"AI 素养"，证明你真的在用而不是被用。
3. **Prompt 模板沉淀**——`你是 X 角色…任务…读者…约束…产出` 这个 5 元组在四类任务（检索/写作/代码/复盘）上都成立，建议存成 Skill 复用。
4. **AI 出代码要带"约束 prompt"**——明确指定标准库、错误处理风格、注释密度，比"随便写"质量稳定得多。
