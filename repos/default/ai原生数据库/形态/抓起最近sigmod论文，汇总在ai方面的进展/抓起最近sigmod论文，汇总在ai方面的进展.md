---
id: "ses_ee5af351ef379955"
title: "抓起最近sigmod论文，汇总在ai方面的进展"
createdAt: "2026-06-02T11:07:50.942Z"
updatedAt: "2026-06-02T11:11:12.585Z"
---

# 抓起最近sigmod论文，汇总在ai方面的进展

<!-- message:id=msg_783be9e07bd0be57 role=user createdAt=2026-06-02T11%3A08%3A48.387Z -->

## User

分析最近的sigmod论文，汇总ai方面的进展

<!-- /message -->
<!-- message:id=msg_aab89a2780bf234c role=assistant createdAt=2026-06-02T11%3A11%3A12.585Z runId=run_8729de43b5c73ff6 status=succeeded -->

## Assistant

**范围说明**

我把“最近”按截至 **2026-06-02** 能查到的最新 SIGMOD 论文来处理：重点看 **SIGMOD/PODS 2026 research track**，必要时用 SIGMOD 2025 做趋势参照。SIGMOD/PODS 2026 官方页面显示会议在 **2026-05-31 至 2026-06-05** 于 Bengaluru 举办；2026 SIGMOD accepted papers 分为 4 个 round，并对应 PACMMOD 2025/2026 的若干 issue。([2026.sigmod.org](https://2026.sigmod.org/))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

**一句话结论**

最近 SIGMOD 的 AI 方向已经从“把 ML 模型接到数据库上”明显转向三条主线：**数据库支撑大模型系统、LLM/Agent 改造数据管理工作流、向量数据库成为核心数据系统问题**。同时，论文更强调成本、延迟、鲁棒性、安全、可解释性和统计保证，而不是只报模型准确率。

**1. LLM 正在进入数据管理的核心交互层**

最明显的变化是，LLM 不再只是 Text-to-SQL 的一个模型组件，而是被放进数据分析、知识图谱问答、表格问答、可视化、数据集描述生成、文档抽取等完整工作流里。

2026 年 SIGMOD 里能看到一批相关工作：例如 `DeepEye-SQL`、`DIVER`、`PRISM`、`PLForge`、`ST-Raptor`、`AixelAsk`、`Chatty-KG`、`AutoDDG`、`MultiVis-Agent` 等，覆盖 Text-to-SQL、SQL/过程式 SQL 生成、半结构化表格 QA、知识图谱对话问答、数据集描述、多模态可视化。官方 accepted list 中明确列出了这些方向，如 Text-to-SQL、安全、KG 多智能体问答、表格 QA、可视化 agent 等论文。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

我的判断是：SIGMOD 社区对 LLM 的关注点已经从“自然语言能不能生成 SQL”升级到“生成结果如何可控、可验证、可复用、可降本”。例如 `PRISM` 直接关注 NL2SQL 的成本-准确率权衡，`Reliable Answers for Recurring Questions` 关注重复问题的可靠回答，`Are Your LLM-based Text-to-SQL Models Secure?` 关注 SQL injection/backdoor 风险。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

这说明 Text-to-SQL 方向正在工程化：下一步竞争点不是单 benchmark 分数，而是 **安全约束、模板/解码约束、证据链、值链接、交互式修正、执行一致性和成本控制**。

**2. 数据库开始反向服务 AI 基础设施**

另一个很强的趋势是：数据库系统论文开始直接处理 LLM serving、KV cache、训练数据面、分布式训练、模型管理等 AI 基础设施问题。

2026 年相关论文非常密集：`AlignedServe` 做高吞吐 LLM serving 编排，`KVDrive` 和 `Beluga` 处理长上下文推理中的 KV cache 管理，`HotPrefix` 做 prefix sharing 调度，`DepCache` 面向 GraphRAG 的 KV cache，`Efficient LLM Serving for Agentic Workflows` 直接从数据系统角度讨论 agentic workflow 的 LLM serving。官方列表中这些论文集中出现在 2026 round 1、round 3、round 4。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

训练侧也很明显：`TensorSocket` 关注深度学习训练的数据加载共享，`Hydraulis` 关注大 Transformer 训练的并行策略和数据分配，`DFLOP` 面向多模态 LLM 训练 pipeline 优化，`Mixtera` 则把 foundation model training 的数据平面作为系统问题。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

这背后的信号很清楚：AI 系统瓶颈正从“模型结构”外溢到 **数据加载、缓存层级、调度、近存/远存、CXL、GPU/SSD/RDMA 协同、agent 多请求复用**。SIGMOD 的优势恰好在这些系统问题上。

**3. 向量数据库和 embedding 数据管理已经成为主战场**

2026 SIGMOD 的向量检索相关论文数量非常多，且主题已经远超“做一个更快的 ANN index”。现在的问题包括：带过滤条件的向量检索、动态流式更新、向量索引合并、跨 embedding model 集成、分布式/RDMA/GPU/SSD 向量检索、多向量检索、安全访问控制、benchmark、predicate 与 vector query 的统一优化。

典型例子包括 `HARMONY`、`VecFlow`、`VecFlow-Chamfer`、`CoTra`、`GEM`、`CANDOR-Bench`、`Curator`、`DiskJoin`、`SAQ`、`Integrating Vector Databases across Embedding Models`、`Beyond Vector Search`、`HONEYBEE` 等。官方 accepted list 里能看到大量 ANNS、vector database、filtered-search、multi-vector search、RDMA、GPU、RBAC、benchmark 相关论文。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

这里的进展可以概括为四点：

1. **从单查询 ANN 到数据库语义查询**：过滤条件、谓词、权限、更新、join、multi-vector 都进入研究核心。
2. **从算法论文到系统论文**：GPU、RDMA、SSD、分布式架构成为性能来源。
3. **从静态 benchmark 到动态 open-world**：例如连续 ANNS benchmark 说明大家开始关心真实线上向量库的增删改和分布漂移。
4. **从一个 embedding 空间到多个 embedding 空间**：跨 embedding model 的集成会变成企业级 RAG/agent 系统的实际痛点。

我认为这是近两年 SIGMOD 最“产业可转化”的方向之一。

**4. AI for DB：自动调优、优化器和 learned components 更务实了**

传统 “ML for DB” 方向没有消失，但风格变了。早期常见的是 learned index、learned cardinality/cost model、RL knob tuning；最近论文更关注 **可迁移、可解释、可抗 workload drift、能和传统优化器共存**。

2026 年能看到 `AgentTune`、`MCTuner`、`SEFRQO`、`Reqo`、`Divo`、`GenRewrite`、`Can Large Language Models Be Query Optimizer for Relational Databases?`、`GenJoin`、`Rainbow`、`Are Learned DBMS Components Robust to Workload Drift?` 等。它们覆盖 knob tuning、RAG/query optimizer、学习型 cost model、LLM query rewrite、生成式 query optimizer、OOD workload 下的 index benefit 估计等。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

和 2025 相比，这个趋势更强。2025 已经有 `How Good are Learned Cost Models, Really?`、LLM 自动数据库调优、Text-to-SQL 自动验证修复、RAG cache、learned index tuning 等方向。([2025.sigmod.org](https://2025.sigmod.org/sigmod_papers.shtml)) 到 2026，研究重点明显从“能否用 AI 替换组件”变成“AI 如何作为数据库组件的一部分受控运行”。

我的判断：未来 DBMS 里的 AI 不会简单替代优化器，而是形成 **候选生成 + 代价估计 + 规则约束 + 执行反馈 + workload memory** 的混合架构。LLM 适合做搜索空间压缩、候选 rewrite、历史经验迁移；传统优化器仍然负责可验证的 cost/rule/plan 边界。

**5. AI 查询处理：把模型调用当成昂贵算子来优化**

一个很重要但容易被忽略的趋势是：SIGMOD 开始系统化处理“AI query”或“语义查询”的执行代价。也就是，查询里可能包含 LLM、视觉模型、embedding model、分类器、模糊 LIKE、unstructured data predicate，这些模型调用昂贵、随机、有误差，因此需要数据库式优化。

2026 年有 `100x Cost & Latency Reduction` 这类用轻量 proxy model 做 AI query approximation 的工作，也有 unstructured data join 的统计保证、LLM-powered data processing with guarantees、task cascades、language-model approximated LIKE predicate、AI/ML inference over joins、semantic/multimodal analytics 等方向。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

这类论文的共同思想是：不要把大模型当黑盒 API 随便调用，而要像数据库处理 join/order/filter 一样处理模型调用：

- 能用 cheap proxy 的地方先用 proxy。
- 能 cascade 的地方分层筛选。
- 能缓存和复用的地方缓存。
- 能给统计误差界的地方给 error guarantee。
- 能把模型推理下推、合并、factorize 的地方做执行计划优化。

这很可能成为“AI-native database”最核心的技术层之一。

**6. 数据质量、隐私、公平和安全成为 AI 数据系统的硬指标**

AI 相关数据管理不只是性能问题。2026 论文里有不少围绕 fairness、privacy、poisoning、backdoor、causal explanation、watermarking、data forgetting、DP synthetic data、privacy-preserving recommendation 的工作。例如 fair data preprocessing、poisoning/backdoor、differentially private tabular synthesis、privacy-preserving federated recommendation、data forgetting、watermarking、causal explanations 等。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

这说明 SIGMOD 对 AI 的理解仍然是“数据为中心”的：模型效果固然重要，但真正落地要解决 **数据来源、数据修复、访问控制、隐私预算、攻击面、审计、解释、遗忘和溯源**。

这也和 2025 的 `Provenance-Enabled Explainable AI`、LLM 数据清洗、数据增强、mislabeled data subset selection 等方向连续。([2025.sigmod.org](https://2025.sigmod.org/sigmod_papers.shtml))

**7. 图、时序和多模态数据仍是 AI+DB 的重要载体**

SIGMOD 2026 里图相关 AI/ML 论文也很多，包括 GNN training、spectral GNN benchmark、graph neural architecture search、temporal graph learning、GraphRAG cache、knowledge graph QA、subgraph neural optimizer 等。时序方向则有 anomaly detection、time-series clustering、cleaning、compression 等。([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))([2026.sigmod.org](https://2026.sigmod.org/sigmod_papers.shtml))

这些方向的共同点是：AI 应用的数据越来越复杂，不只是二维表。数据库系统需要同时支持 **关系、图、向量、文本、表格、文档、视频、时空轨迹、日志**。这也是为什么“multi-modal analytics”“semantic query”“GraphRAG”“semi-structured table QA”在 2026 里显得很突出。

**和 2025 相比的变化**

SIGMOD 2025 已经有不少 AI 相关论文：ML query processing、deep transfer learning model search、LLM 视频数据管理、learned cost model、RAG cache、Text-to-SQL 验证、CXL embedding serving、LLM 自动调优、GNN out-of-core training、AI-assisted UDF inlining、LLM 数据清洗、embedding KG serving 等。([2025.sigmod.org](https://2025.sigmod.org/sigmod_papers.shtml))

但 2026 的变化更明显：

- **LLM 从应用点扩展为系统层问题**：serving、KV cache、agent workflow、GraphRAG cache、训练数据面都进入 SIGMOD。
- **向量数据库从“检索算法”升级为完整 DBMS 问题**：权限、过滤、动态更新、分布式、GPU、benchmark、跨 embedding model。
- **AI for DB 从模型替代走向 hybrid architecture**：优化器、调优器、rewrite、cost model 都在强调可靠性、解释性和 workload drift。
- **成本和保证更重要**：很多论文标题或主题直接出现 cost、latency、guarantees、robustness、security、privacy。
- **Agent 化开始出现但仍偏早期**：`AgenticScholar`、`AgentTune`、`MultiVis-Agent`、`Chatty-KG` 等说明 agent 已进入数据库论文，但重点还在 orchestration、tool use、可靠性和系统支撑，而不是完全自治。

**值得重点跟的方向**

如果你想从 SIGMOD 这些论文里提炼未来 1-2 年的 AI 数据系统机会，我建议优先看这几类：

1. **LLM serving + KV cache + agent workflow**
   
   这是大模型应用成本的核心。长上下文、multi-agent、GraphRAG 会让 KV cache、prefix reuse、dependency-aware caching、memory tiering 变成高价值系统问题。

2. **Vector DB 2.0**
   
   重点不是再做一个 HNSW，而是 filtered vector search、hybrid search、predicate pushdown、multi-vector、dynamic update、cross-embedding federation、security/RBAC、GPU/RDMA/SSD execution。

3. **LLM-native query processing**
   
   把 LLM/embedding/vision model 当作数据库算子，做 cost-based planning、cascade、proxy approximation、statistical guarantee、caching、batching、reuse。这是语义查询和非结构化数据分析落地的关键。

4. **AI-assisted optimizer and tuning**
   
   不要押注“LLM 直接当优化器”。更靠谱的是让 LLM/agent 做历史经验迁移、候选计划生成、配置搜索引导、query rewrite，再由传统优化器和执行反馈约束。

5. **可信 AI 数据管理**
   
   Text-to-SQL 安全、数据投毒、隐私保护、可解释性、数据遗忘、水印、因果解释都会变成企业级 AI 系统必须补齐的能力。

**我的总体判断**

最近 SIGMOD 的 AI 进展可以概括为：**AI 正在数据库化，数据库也正在 AI 基础设施化**。

“AI 数据库”不再只是“数据库里加向量索引”或“自然语言查表”。更完整的形态会是：

- 底层有向量、图、关系、多模态统一存取；
- 中间有 cost-based semantic query engine；
- 上层有 LLM/agent 做自然语言、workflow、数据发现和自动调优；
- 旁路有安全、隐私、可解释、审计和成本控制；
- 系统层专门优化训练、推理、KV cache、RAG cache、GPU/SSD/RDMA/CXL 等资源。

所以，如果按研究和工程价值排序，我会把 **向量数据库系统化、LLM serving 数据系统化、LLM-native 查询优化、可信 Text-to-SQL/Agent、AI-assisted DB tuning** 作为当前 SIGMOD AI 方向最值得跟的五条主线。

<!-- /message -->
