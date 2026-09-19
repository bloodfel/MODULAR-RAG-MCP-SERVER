# Phase 1：DEV_SPEC 精读指南（DeepSeek 带读版）

> 用法：打开 DeepSeek 网页版 → 把本文档第二节的 prompt 整段粘贴 → 附上 DEV_SPEC.md 文件（3214 行，可整份附上；如太长则按 Session 行号分段粘贴）。
> 读完回来后告诉我，我会出一份 30 分钟校准测验查漏补缺，然后正式进入 M1 开发。

---

## 一、课程表：9 个 Session，逐知识点覆盖

| Session | 章节（行号） | 知识点清单 |
|---|---|---|
| **S1 全局认知** | §1-2 (L18-166) | RAG 是什么/为什么需要；分块为何要语义感知；Hybrid Search 双路互补原理；粗排→精排两段式的成本逻辑；MCP 协议定位；Image-to-Text 多模态策略；可观测性与评估闭环概念 |
| **S2 摄取流水线·上** | §3.1.1 (L173-247) | 五层职责（Loader/Splitter/Transform/Embed/Upsert）；统一 Markdown 中间格式的设计意图；SHA256 前置去重与零成本增量更新；SQLite + WAL；RecursiveCharacterTextSplitter 切分原理 |
| **S3 摄取流水线·下** | §3.1.1 (L249-310) | LLM 三种增强（智能重组/元数据注入/图片 Captioning）；原子化+幂等+降级三个工程特性；chunk_id = hash(source+section+content)；Upsert 语义；双路向量化与批处理；DocumentManager 生命周期（跨 4 存储协调删除） |
| **S4 检索流水线** | §3.1.2 (L312-349) | Query 预处理（关键词提取/查询扩展）；**BM25 公式**（从 TF-IDF 推起）；Dense 向量与余弦相似度；**RRF 公式推导**（为什么用排名不用分数）；metadata 过滤前置/后置；Cross-Encoder vs Bi-Encoder；LLM Rerank；fallback 降级设计 |
| **S5 MCP 服务** | §3.2 (L351-436) | MCP 是什么（底层 JSON-RPC 2.0）；Host/Client/Server 三角色；tools/resources/prompts 三类能力；stdio 传输；能力协商握手；本项目 3 个 tool 的输入输出设计 |
| **S6 可插拔架构** | §3.3 (L437-628) | 抽象基类 ABC；工厂模式与注册表模式；依赖倒置原则；配置驱动设计（settings.yaml 加载与校验）；"接口先行"开发法；异常与降级约定 |
| **S7 观测与多模态** | §3.4-3.5 (L629-1079) | TraceContext/TraceCollector 设计；JSON Lines 结构化日志；Streamlit 六页面架构；on_progress 进度回调；Vision LLM captioning 全流程；图片存储与索引 |
| **S8 测试方案** | §4 (L1080-1257) | TDD 红绿重构循环；Unit/Integration/E2E 三层各自测什么；mock 与 contract test；golden test set；Ragas 四大指标（faithfulness / answer relevancy / context precision / recall）；hit_rate 与 MRR；性能测试与 CI 概念 |
| **S9 架构与排期** | §5-6 (L1258-3213 选读) | 整体架构图怎么读；目录结构约定；数据流图；配置文件全貌；任务格式与验收标准怎么读；阶段 A-I 概览；如何配合 auto-coder skill 开发 |

---

## 二、DeepSeek 带读 Prompt（整段复制）

```text
你是一位资深 RAG 工程师兼耐心导师。我正在通过一个开源项目「Modular RAG MCP Server」
深度学习 RAG 工程化。我的目标有两个：① 吃透每个知识点（面试导向，这个项目本身就是
按面试考点设计的）；② 之后 AI 会按照这份 DEV_SPEC 帮我写代码，我必须有能力 review
它写的每一行。所以请你【带我精读】这份文档，绝对不要总结式地带过。

我会把 DEV_SPEC.md 全文（约 3200 行）附给你。请严格按照下面的规则教学：

【教学规则】
1. 按 9 个 Session 的顺序进行（课程表见下）。每个 Session 内按"知识点"逐个讲，
   一次只讲一个知识点。每个知识点讲完出 1-2 道检验题，等我用自己的话回答、你纠正
   补充之后，才进入下一个知识点。我说"继续"才推进；我说"不懂"就换一种类比重讲。
2. 每个知识点固定六步讲：
   ① 是什么（大白话 + 一个生活化类比）
   ② 为什么需要它（如果没有它会出什么问题）
   ③ 怎么实现（关键机制，不回避公式和伪代码）
   ④ 在 DEV_SPEC 哪个小节体现（引用小节号或行号，带我回到原文）
   ⑤ 面试官会怎么问（高频问题 + 答题框架）
   ⑥ 检验题
3. 不跳过数学：BM25 要从 TF-IDF 推起并给出完整公式；余弦相似度、RRF 公式都要
   用具体的小数字例子算一遍。
4. 不跳过代码概念：ABC 抽象基类、工厂模式、注册表、配置校验、pytest mock 等，
   要讲到"在 Python 里具体怎么写"的粒度，给出短示例代码。
5. 每个 Session 结束时：给出该 Session 的知识点清单表格（知识点 / 一句话本质 /
   面试考点），外加 3 道综合复述题让我整体作答。
6. 你全程维护一个进度表（9 个 Session × 各自知识点的 checklist），每次我发消息
   时先简短显示当前进度，再继续教学。

【9 个 Session 课程表】（行号以 DEV_SPEC.md 为准）
S1 全局认知：§1-2 (L18-166) —— RAG 是什么/为什么；分块为何语义感知；Hybrid
   Search 双路互补；粗排→精排成本逻辑；MCP 协议定位；Image-to-Text 多模态；
   可观测性与评估闭环
S2 摄取流水线·上：§3.1.1 (L173-247) —— 五层职责（Loader/Splitter/Transform/
   Embed/Upsert）；统一 Markdown 中间格式；SHA256 前置去重与零成本增量更新；
   SQLite+WAL；RecursiveCharacterTextSplitter 切分原理
S3 摄取流水线·下：§3.1.1 (L249-310) —— LLM 三种增强（智能重组/元数据注入/
   图片 Captioning）；原子化+幂等+降级；chunk_id=hash(source+section+content)；
   Upsert 语义；双路向量化与批处理；DocumentManager 生命周期（跨 4 存储删除）
S4 检索流水线：§3.1.2 (L312-349) —— Query 预处理；BM25 原理与公式（从 TF-IDF
   推起）；Dense 向量与余弦相似度；RRF 公式推导（为何用排名不用分数）；metadata
   过滤前置/后置；Cross-Encoder vs Bi-Encoder；LLM Rerank；fallback 降级
S5 MCP 服务：§3.2 (L351-436) —— MCP 是什么（JSON-RPC 2.0）；Host/Client/Server
   角色；tools/resources/prompts；stdio 传输；能力协商；本项目 3 个 tool 设计
S6 可插拔架构：§3.3 (L437-628) —— ABC 抽象基类；工厂模式与注册表；依赖倒置；
   配置驱动；接口先行开发法；异常与降级约定
S7 观测与多模态：§3.4-3.5 (L629-1079) —— TraceContext/TraceCollector；JSON
   Lines 结构化日志；Streamlit 六页面；on_progress 回调；Vision LLM captioning
   全流程；图片存储与索引
S8 测试方案：§4 (L1080-1257) —— TDD 红绿重构；Unit/Integration/E2E 三层；
   mock 与 contract test；golden test set；Ragas 四大指标（faithfulness /
   answer relevancy / context precision / recall）；hit_rate 与 MRR；性能测试
S9 架构与排期：§5-6 (L1258-3213 选读) —— 整体架构图怎么读；目录结构约定；
   数据流；配置文件全貌；任务格式与验收标准怎么读；阶段 A-I 概览；如何配合
   auto-coder skill 进行 AI 驱动开发

【开始方式】
DEV_SPEC.md 在附件里。收到后：① 先展示 9-Session 进度表；② 然后直接开始 S1 的
第一个知识点。全程用中文教学。
```

---

## 三、给 DeepSeek 的使用提示

1. **附件**：DEV_SPEC.md 直接拖进对话框当附件（3214 行一般没问题）。如果它说太长，就按 Session 行号分段粘贴，比如学 S4 时只发 L312-349 附近的内容。
2. **节奏**：不要连点"继续"。每个检验题先自己写答案再发，答错的地方才是真正学到东西的地方。
3. **卡住了怎么办**：DeepSeek 讲得含糊或者你和它争论不出结果的知识点，**记到下面的"存疑清单"**，回来问我——这正是我该花钱的地方。

## 存疑清单（回来问我）

- （记录 DeepSeek 没讲清楚 / 你俩有分歧的知识点）

---

## 回来之后的流程

1. 我出 6-8 道跨 Session 校准题（30 分钟），定位薄弱点
2. 薄弱点定向补课
3. 正式进入 **M1：Stage A+B**（工程骨架 + 可插拔底座，AI 主导 + 逐任务讲解）
