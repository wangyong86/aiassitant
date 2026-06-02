---
id: "ses_546960bed315f60a"
title: "当前AI需要一站式混合检索，先说明其主要场景，然后分析典型做法，最后给出理想方案。"
createdAt: "2026-05-20T09:54:34.616Z"
updatedAt: "2026-05-21T00:33:02.031Z"
---

# 当前AI需要一站式混合检索，先说明其主要场景，然后分析典型做法，最后给出理想方案。

<!-- message:id=msg_c1d42a8a3a150362 role=user createdAt=2026-05-20T09%3A54%3A34.616Z -->

## User

当前AI需要一站式混合检索，先说明其主要场景，然后分析典型做法，最后给出理想方案。

<!-- /message -->
<!-- message:id=msg_90cb2551bfe8f37b role=assistant createdAt=2026-05-20T09%3A56%3A16.780Z runId=run_49f15b106e0379bb status=succeeded -->

## Assistant

当前 AI 要做“一站式混合检索”，本质上不是只把向量检索、关键词检索、知识库问答拼在一起，而是要让用户在一个入口里完成“查得全、找得准、能解释、可追溯、可行动”的信息获取流程。下面按主要场景、典型做法、理想方案展开。

**一、主要场景**

1. **企业知识库问答**

这是最典型的场景。企业内部资料往往分散在文档、网页、FAQ、工单、邮件、会议纪要、代码仓库、数据库、对象存储中。用户希望像问 ChatGPT 一样直接提问，例如：

`​``text
上季度华东区销售下滑的主要原因是什么？
某客户合同里的 SLA 条款是什么？
这个报错以前有没有处理过？
`​``

这类问题既需要语义理解，也需要精确定位来源。单纯向量检索容易找相似但不一定准确的内容，单纯关键词检索又容易漏掉同义表达，因此需要混合检索。

2. **RAG 问答与智能客服**

在智能客服、政务问答、金融问答、医疗辅助问答等场景中，AI 不能凭空生成答案，必须基于可信资料回答。用户的问题通常表达不规范，例如：

`​``text
我这个保单能不能赔？
医保异地备案怎么弄？
系统提示 10031 是啥意思？
`​``

这些问题需要同时匹配业务术语、用户口语、历史案例、政策条款和结构化规则。混合检索可以把“语义相似”和“关键词命中”结合起来，提高召回率和答案可靠性。

3. **代码、日志、工单与运维排障**

技术团队经常需要跨代码仓库、日志平台、监控事件、知识库、历史工单进行检索。例如：

`​``text
最近支付回调超时和哪个发布有关？
这个 NullPointerException 以前出现过吗？
订单状态卡在 pending 的处理方案是什么？
`​``

这类场景有大量精确符号，如类名、函数名、错误码、接口路径、trace id。向量检索对自然语言友好，但对精确 token、异常栈、短字符串不稳定；关键词检索对这些内容更强。因此运维、研发、DevOps 场景尤其依赖混合检索。

4. **行业研究与投研分析**

投研、咨询、战略、市场研究中，用户需要从财报、公告、新闻、研报、电话会纪要、数据库指标中综合信息。例如：

`​``text
比较宁德时代和比亚迪最近两年的毛利率变化原因。
找出 AI 服务器液冷产业链的核心公司和依据。
`​``

这类问题既要查事实，也要做聚合、对比和推理。检索系统不仅要找到文档，还要抽取数据、识别时间、引用来源、处理冲突信息。

5. **法律、合规与审计**

法律合规场景对可追溯性要求很高。例如：

`​``text
这份合同有没有违反公司标准条款？
某类数据能否跨境传输？
过去类似案件法院怎么判？
`​``

这类问题需要准确匹配法条、合同条款、司法案例、内部制度，并且必须保留引用依据。混合检索不仅要召回相关材料，还要区分权威级别、发布时间、适用范围和冲突关系。

6. **多模态知识检索**

越来越多知识不只在文本中，还在图片、PPT、PDF 扫描件、视频、音频、图表、流程图中。例如：

`​``text
找出这批质检图片里和这个缺陷相似的案例。
会议里谁提到了预算调整？
这份 PDF 图表说明了什么趋势？
`​``

这时需要 OCR、ASR、图像 embedding、文本 embedding、结构化元数据检索一起工作，一站式混合检索会成为统一入口。

7. **个人知识管理与办公助手**

个人或团队场景中，信息分散在本地文件、网盘、聊天记录、邮件、日历、笔记系统中。用户希望问：

`​``text
上次讨论产品定价是在什么时候？
帮我找一下关于 A 项目的所有决策依据。
我之前收藏过一篇关于向量数据库选型的文章，在哪？
`​``

这里的核心是跨源检索、权限控制、时间线整理和上下文记忆。

---

**二、典型做法**

当前主流的一站式混合检索通常由几类能力组合而成。

1. **关键词检索**

典型技术包括倒排索引、BM25、TF-IDF、短语匹配、字段过滤。常见工具有 Elasticsearch、OpenSearch、Solr、Lucene 等。

优点是：

- 对专有名词、编号、错误码、函数名、合同编号非常准确。
- 支持过滤、排序、高亮、聚合。
- 工程成熟，性能稳定。
- 可解释性较好，知道命中了哪些词。

缺点是：

- 不擅长语义相似。
- 用户换一种说法可能搜不到。
- 对长问题、模糊问题、跨语言问题效果有限。
- 对“意图”理解弱。

例如用户问“离职赔偿怎么算”，文档里写的是“经济补偿金计算规则”，纯关键词检索可能召回不足。

2. **向量检索**

向量检索会把文本、图片、代码等内容转成 embedding，再用向量数据库做近邻搜索。常见组件包括 Milvus、FAISS、pgvector、Weaviate、Qdrant、Pinecone、Elasticsearch vector search 等。

优点是：

- 能理解语义相似。
- 适合自然语言问答。
- 可跨语言、跨表达方式匹配。
- 对非结构化文档友好。

缺点是：

- 对精确匹配不稳定。
- 对数字、版本号、错误码、短 token 容易失真。
- 召回结果有时“看起来相关但并不正确”。
- embedding 模型更新后可能需要重建索引。
- 可解释性比关键词检索弱。

例如用户搜“ERR_CONN_RESET”，向量检索可能找不到真正包含该错误码的文档，反而找到一些泛泛讨论网络错误的内容。

3. **结构化检索**

很多业务数据在数据库、表格、数据仓库、知识图谱里。例如订单、客户、交易、合同、指标、用户画像。典型做法包括：

- SQL 查询。
- Metadata filter。
- 知识图谱查询。
- API 调用。
- OLAP 指标查询。
- 权限过滤。

优点是结果准确、可计算、适合事实类问题。缺点是用户自然语言到查询语句的转换难度较高，而且结构化数据和非结构化文档之间经常割裂。

4. **多路召回**

实际系统通常不会只用一种召回方式，而是多路并行：

- BM25 召回关键词相关文档。
- 向量召回语义相关文档。
- 元数据过滤限定时间、部门、权限、文档类型。
- 热门文档或历史点击补充召回。
- 知识图谱召回实体关系。
- SQL/API 获取实时业务数据。

多路召回解决了单一路径的短板，但会带来结果融合问题：不同检索器的分数不可直接比较，召回结果可能重复、冲突、质量不一致。

5. **结果融合与重排序**

常见融合方法包括：

- 分数归一化后加权。
- Reciprocal Rank Fusion，即 RRF。
- 规则加权，例如标题命中加分、近期文档加分、权威来源加分。
- Cross-Encoder reranker 重排序。
- LLM rerank 或 LLM judge。
- 去重与多样性控制。

典型流程是先用便宜的检索方式召回较多候选，再用更贵但更准的模型重排。例如先召回 100 条，再 rerank 到前 10 条，最后交给大模型回答。

6. **RAG 生成答案**

在问答场景中，检索结果通常会进入 RAG 流程：

`​``text
用户问题
 -> 查询改写
 -> 多路召回
 -> 去重与重排
 -> 上下文压缩
 -> LLM 生成答案
 -> 引用来源
 -> 置信度与追问
`​``

这能让大模型基于外部知识回答，减少幻觉。但如果检索召回质量差，后面的生成也很难可靠。

7. **Agent 式检索**

更复杂的问题会使用 Agent。它不是一次检索就结束，而是多轮规划：

`​``text
先理解问题
再拆解子问题
分别查询文档、数据库、网页、API
对结果进行交叉验证
最后综合回答
`​``

例如“分析某公司过去三年利润下降原因”，系统可能需要查财报、新闻、行业数据、管理层解释、财务指标。这类任务适合 Agent，但成本、延迟和稳定性都更难控制。

---

**三、典型做法的问题**

当前很多混合检索系统看似“一站式”，实际只是“多个检索入口拼接”。主要问题有这些。

1. **检索链路割裂**

关键词、向量、数据库、知识图谱、网页搜索常常由不同系统维护。用户感知上是一个入口，但底层没有统一的查询理解、权限、排序、引用和评估体系。

结果是：有些问题适合文档检索，有些问题适合 SQL，有些问题需要两者结合，但系统不知道该走哪条路径。

2. **只重召回，不重答案质量**

很多系统把重点放在“召回多少文档”，但用户真正要的是“答案是否正确、完整、可解释”。如果检索结果没有经过筛选、压缩、事实核验，LLM 很容易把多个片段拼成一个看似合理但错误的回答。

3. **权限和安全容易后补**

企业检索必须处理：

- 用户权限。
- 部门权限。
- 文档密级。
- 数据脱敏。
- 审计日志。
- 外部模型调用边界。
- 多租户隔离。

很多系统先做检索效果，后补权限控制，最后会遇到严重风险。正确做法应该是权限前置，在索引、召回、重排、生成、引用各层都生效。

4. **评估体系不完整**

混合检索不能只看 Top-K 命中率。还要评估：

- 召回率。
- 精确率。
- MRR / NDCG。
- 答案正确性。
- 引用准确性。
- 幻觉率。
- 拒答正确率。
- 延迟。
- 成本。
- 权限泄漏率。
- 用户满意度。

没有评估体系，就无法判断加一个 reranker、换一个 embedding、调整 chunk 策略到底有没有收益。

5. **文档处理质量决定上限**

很多 RAG 项目失败不是模型差，而是文档处理差。典型问题包括：

- PDF 解析错乱。
- 表格丢失结构。
- 标题层级丢失。
- 图片没有 OCR。
- chunk 切分破坏语义。
- 元数据缺失。
- 同一文档多个版本混杂。
- 过期内容没有失效机制。

混合检索的上限往往由数据治理决定，而不是由某一个向量数据库决定。

6. **实时性不足**

企业知识变化很快。政策、合同、工单、指标、代码、日志都在更新。如果索引不能增量更新，或者文档版本没有管理，系统很容易回答过期内容。

7. **用户意图理解不足**

用户的问题可能是查找、问答、比较、总结、排障、生成报告、数据分析、追踪来源等不同任务。如果系统把所有问题都当作“检索文档”，体验会很差。

---

**四、理想方案**

理想的一站式混合检索应该是一个“统一检索与知识推理平台”，而不只是一个搜索框加向量库。可以分为六层。

---

**1. 统一数据接入层**

首先要把不同来源的数据统一纳入治理范围，包括：

- 文档：PDF、Word、PPT、Markdown、HTML。
- 表格：Excel、CSV、BI 报表。
- 数据库：MySQL、PostgreSQL、ClickHouse、数据仓库。
- 协作系统：飞书、钉钉、企业微信、Slack、Confluence、Notion。
- 工程系统：Git、Jira、工单、日志、监控。
- 多媒体：图片、音频、视频、扫描件。
- 外部数据：官网、政策网站、新闻、公告、API。

这层不只是同步文件，还要做版本管理、权限继承、数据血缘、更新时间记录、来源标识和变更检测。

理想状态是：所有知识对象都被标准化为统一的 `Knowledge Object`，包含正文、结构、元数据、权限、版本、来源和生命周期。

---

**2. 高质量文档理解与索引层**

文档进入系统后，需要进行深度解析：

- OCR 识别扫描件。
- ASR 处理音视频。
- 表格结构提取。
- 标题层级识别。
- 图片、图表、流程图理解。
- 代码块、公式、脚注、引用识别。
- 文档版本比对。
- 章节级、段落级、句子级 chunk 切分。
- 实体、时间、地点、组织、产品、指标抽取。
- 摘要和关键词生成。
- 多粒度 embedding 生成。

索引不应该只有一个向量索引，而应该至少包括：

- 倒排索引，用于关键词与精确匹配。
- 向量索引，用于语义召回。
- 结构化索引，用于元数据过滤和字段查询。
- 图谱索引，用于实体关系查询。
- 时序索引，用于时间相关问题。
- 权限索引，用于安全过滤。
- 多模态索引，用于图片、音频、视频检索。

这才是真正的混合检索基础。

---

**3. 查询理解与路由层**

用户输入后，系统不应立刻检索，而应先理解问题：

- 用户想查事实、找文档、做总结，还是分析数据？
- 问题里有哪些实体、时间、指标、条件？
- 是否需要精确匹配？
- 是否需要实时数据？
- 是否涉及权限敏感内容？
- 是否需要多步推理？
- 是否需要追问澄清？

然后进行查询改写和路由：

`​``text
原始问题：
“上个月支付失败最多的原因是什么，有没有相关工单？”

系统拆解：
1. 查询上个月支付失败统计数据。
2. 检索支付失败相关工单。
3. 检索近期发布记录或异常日志。
4. 综合生成原因分析。
`​``

理想的路由策略应该结合规则、轻量模型和大模型：

- 简单问题走低成本检索。
- 精确问题优先关键词和结构化查询。
- 语义问题优先向量召回。
- 复杂问题走 Agent 多步检索。
- 高风险问题加强引用和拒答机制。

---

**4. 多路召回、融合与重排层**

理想的召回架构应该支持并行检索：

`​``text
BM25 / keyword
vector search
metadata filter
SQL / API
knowledge graph
recent activity index
user history
multimodal search
web search, if allowed
`​``

然后进行统一融合：

- 去重：同一文档、同一段落、同一版本去重。
- 分数校准：不同召回器分数统一。
- 权威加权：制度、合同、官方文档优先。
- 时效加权：新版本优先，过期文档降权。
- 上下文扩展：命中片段前后章节补齐。
- 多样性控制：避免结果都来自同一文档。
- 交叉验证：关键事实尽量多源确认。
- Reranker 精排：用 cross-encoder 或 LLM 判断相关性。

最终交给大模型的不是“随便几个 chunk”，而是经过压缩、排序、冲突检测后的证据包。

---

**5. 可信生成与可追溯回答层**

生成层要解决的不只是“写得像人”，而是“回答可信”。理想能力包括：

- 每个关键结论都能引用来源。
- 引用精确到文档、章节、段落、表格或字段。
- 区分事实、推断和建议。
- 标出信息时间范围。
- 对冲突信息进行说明。
- 对证据不足的问题拒答或追问。
- 不把无关检索结果硬拼进答案。
- 支持答案展开、追问、下载报告、生成工单、调用业务动作。

例如回答不应该只是：

`​``text
支付失败主要是通道超时。
`​``

而应该是：

`​``text
上个月支付失败的主要原因是通道 A 超时，占失败量 43%。证据来自支付失败统计表 2026-04、工单 INC-2381 和 4 月 17 日发布后的监控告警。需要注意的是，4 月 29 日之后失败率已回落，因此该结论主要适用于 4 月中下旬。
`​``

这类回答才适合企业使用。

---

**6. 评估、反馈与持续优化层**

一站式混合检索必须可运营。需要建立离线和在线评估体系。

离线评估包括：

- 标准问题集。
- 标准答案。
- 标准证据。
- Top-K 召回率。
- NDCG / MRR。
- 答案正确率。
- 引用准确率。
- 拒答准确率。

在线评估包括：

- 用户点击。
- 用户追问。
- 点赞点踩。
- 答案采纳率。
- 人工纠错。
- 查询无结果率。
- 延迟和成本。
- 安全审计。

系统应能自动发现：

- 哪些问题经常搜不到。
- 哪些文档质量差。
- 哪些索引过期。
- 哪类问题需要新增工具。
- 哪些答案引用错误。
- 哪些业务词需要同义词维护。

这使混合检索从一次性项目变成持续进化的平台。

---

**五、推荐的目标架构**

可以用下面的方式概括理想方案：

`​``text
用户入口
  |
查询理解与任务规划
  |
检索路由
  |
多路召回
  |-- 关键词检索
  |-- 向量检索
  |-- 结构化查询
  |-- 知识图谱
  |-- 多模态检索
  |-- 实时 API
  |
融合、去重、重排、权限过滤
  |
证据包构建与上下文压缩
  |
大模型生成 / Agent 推理
  |
引用、校验、拒答、追问
  |
反馈、评估、持续优化
`​``

核心原则是：

- 用关键词检索保证精确性。
- 用向量检索保证语义召回。
- 用结构化查询保证事实和数据准确。
- 用知识图谱处理实体关系。
- 用 reranker 提升最终相关性。
- 用权限系统保证安全。
- 用引用和评估保证可信。
- 用 Agent 处理复杂任务，但不滥用 Agent。

---

**六、落地建议**

如果从零建设，不建议一开始就做庞大的全能系统，而应分阶段落地。

第一阶段：建立基础 RAG 和混合检索。

- 接入核心文档源。
- 做文档解析、chunk、embedding。
- 建立 BM25 + vector 双路召回。
- 加入 metadata filter。
- 增加 reranker。
- 输出带引用答案。
- 建立基础评估集。

第二阶段：增强企业可用性。

- 接入权限系统。
- 支持增量同步。
- 支持文档版本管理。
- 增加结构化数据查询。
- 建立反馈闭环。
- 优化无答案、冲突答案、过期答案处理。

第三阶段：升级为知识平台。

- 加入知识图谱。
- 加入多模态检索。
- 支持 Agent 多步任务。
- 支持跨系统行动，例如创建工单、生成报告、调用 API。
- 建立全链路监控和自动评估。
- 持续优化数据治理和业务词表。

---

**七、最终理想形态**

理想的一站式混合检索不是“搜文档”，而是“面向任务的知识操作系统”。

用户只需要提出自然语言问题，系统能自动判断应该查文档、查数据库、查日志、查图谱、查网页，还是调用业务 API；能把多源结果融合成可信证据；能生成可追溯答案；能在证据不足时主动追问；能在需要时继续执行后续动作。

简而言之：

`​``text
传统搜索：用户找资料。
普通 RAG：AI 根据资料回答。
一站式混合检索：AI 理解任务、寻找证据、验证事实、生成答案，并推动下一步行动。
`​``

真正的价值不在于“有向量库”或“有大模型”，而在于把数据治理、检索召回、语义理解、权限安全、证据追溯、评估反馈和业务行动统一起来。这样的一站式混合检索，才是当前 AI 在企业和专业场景中真正需要的基础设施。

<!-- /message -->
<!-- message:id=msg_93306b10a98e1d02 role=user createdAt=2026-05-20T10%3A02%3A32.421Z -->

## User

当前AI有很强的语义理解能力和本地执行能力，加上mcp，具有整合多种信息的能力。以RAG 问答与智能客服为例，有哪些能力是AI做不好，仍需要数据库管理系统来做的？它整合这些数据的接口是什么，应该如何分层。

<!-- /message -->
<!-- message:id=msg_fddb9ca432e8ba08 role=assistant createdAt=2026-05-20T10%3A04%3A52.539Z runId=run_dbf1f4a322aa125a status=succeeded -->

## Assistant

AI 加上本地执行能力和 MCP 之后，确实可以“调度”和“整合”很多信息源，但在 RAG 问答与智能客服里，AI 更适合作为**理解、路由、推理、生成和交互层**，不适合作为**数据事实、状态管理、权限控制、事务一致性和高性能查询层**。

换句话说：

`​``text
AI 擅长：理解用户意图、拆解问题、选择工具、组织证据、生成自然语言答案。
DBMS 擅长：可靠存储、精确查询、事务一致性、权限隔离、索引优化、数据治理、审计追踪。
`​``

两者不是替代关系，而是分工关系。

---

**一、AI 做不好，仍需要数据库管理系统做的能力**

**1. 精确事实存储与查询**

智能客服里有大量事实必须精确返回，例如：

`​``text
订单状态
物流轨迹
退款金额
保单状态
合同条款版本
用户等级
工单处理进度
库存数量
计费记录
优惠券有效期
`​``

这些信息不能靠 AI “理解后猜测”，必须来自数据库。

例如用户问：

`​``text
我这笔订单为什么不能退款？
`​``

AI 可以理解这是“退款资格判断”，但真正需要查：

`​``text
订单是否发货
是否超过售后期
商品是否支持七天无理由
是否已经使用优惠券
是否发生部分退款
是否命中风控规则
`​``

这些都应由数据库或业务系统返回确定结果。

AI 不适合直接承担这些职责，因为它：

- 不能保证每次都精确。
- 不能可靠维护最新状态。
- 不能天然处理并发更新。
- 不能替代强约束的数据模型。
- 不能保证金额、时间、状态判断零误差。

---

**2. 事务一致性**

客服系统经常不只是“查”，还要“办”：

`​``text
创建工单
修改收货地址
发起退款
补发优惠券
取消订单
升级人工客服
记录用户投诉
`​``

这些动作必须满足事务要求。

例如“发起退款”可能涉及：

`​``text
校验订单状态
冻结退款金额
写入退款申请
更新售后单状态
记录审计日志
通知支付系统
触发消息队列
`​``

这不是 AI 应该直接做的。AI 可以调用“申请退款”工具，但事务本身必须由数据库和业务服务保证。

DBMS 或业务系统需要保证：

- 要么全部成功，要么全部失败。
- 并发请求不会重复退款。
- 状态流转合法。
- 金额计算一致。
- 操作可回滚或可补偿。
- 每一步可审计。

AI 可以发起动作，但不能成为事务边界。

---

**3. 并发控制**

智能客服是高并发系统。多个用户、多个坐席、多个自动化流程可能同时操作同一条数据。

例如：

`​``text
用户在 App 申请退款
客服同时手动创建售后单
后台风控系统同时拦截该订单
仓库系统同时更新发货状态
`​``

这时需要数据库处理：

- 锁。
- MVCC。
- 乐观并发控制。
- 唯一约束。
- 幂等键。
- 状态机校验。
- 事务隔离级别。

AI 没有能力可靠处理这种并发一致性问题。它可以判断“应该查询售后状态”，但不能替代数据库的并发控制机制。

---

**4. 权限、租户隔离与数据安全**

智能客服涉及大量敏感数据：

`​``text
手机号
身份证号
地址
银行卡信息
订单记录
医疗记录
保单信息
企业合同
聊天记录
投诉内容
`​``

AI 不能自己决定“谁可以看什么”。权限必须前置到数据层和服务层。

DBMS / 数据服务需要处理：

- 行级权限。
- 字段级脱敏。
- 多租户隔离。
- 用户身份校验。
- 客服角色权限。
- 数据访问审计。
- 敏感字段加密。
- 数据出境限制。
- 最小权限访问。

例如普通客服可以看到：

`​``text
订单状态、物流信息、售后进度
`​``

但不能看到：

`​``text
完整身份证号、完整银行卡号、内部风控标签、其他用户数据
`​``

AI 不能被允许直接访问原始数据库后再“自觉隐藏”。正确做法是：AI 调用接口时，接口已经根据当前用户、客服角色、场景和权限返回可见数据。

---

**5. 索引、查询优化与高性能检索**

RAG 问答看起来是“AI 检索资料”，但底层仍然依赖各种索引系统：

`​``text
倒排索引
向量索引
结构化索引
时间索引
地理索引
图索引
全文索引
列式索引
缓存索引
`​``

例如客服系统中有几亿条工单、几千万个订单、几百万篇知识库文档。AI 不可能自己逐条读取再判断。

数据库和搜索系统需要完成：

- Top-K 检索。
- 条件过滤。
- 排序分页。
- 聚合统计。
- 权限过滤。
- 分区裁剪。
- 查询计划优化。
- 向量近邻搜索。
- BM25 关键词检索。
- 混合检索融合。

AI 负责提出“我要查什么”，DBMS 负责高效执行“怎么查”。

---

**6. 数据版本管理与时效性**

客服问答中，政策、价格、合同、规则经常变化。

例如：

`​``text
3 月之前的退款规则
4 月新版会员权益
某地区临时物流政策
某保险产品 2026 版条款
某活动期间的优惠规则
`​``

用户问“我这个订单能不能退”，不能只看最新规则，还要看：

`​``text
下单时间
活动时间
支付时间
发货时间
政策生效时间
用户所属地区
商品所属类目
`​``

这些需要数据库维护：

- 生效时间。
- 失效时间。
- 版本号。
- 发布状态。
- 审批状态。
- 历史快照。
- 数据血缘。
- 变更记录。

AI 可以理解“要按订单时间适用规则”，但具体版本匹配必须由数据系统保证。

---

**7. 数据质量、约束和治理**

AI 对脏数据很敏感。RAG 答错，很多时候不是模型差，而是底层数据质量差。

数据库系统需要保证：

- 主键唯一。
- 外键关系。
- 字段类型。
- 非空约束。
- 枚举约束。
- 状态机约束。
- 去重。
- 数据校验。
- 数据清洗。
- 生命周期管理。
- 过期数据下线。
- 冲突数据标记。

例如知识库里同时存在：

`​``text
旧版退款政策
新版退款政策
地方特殊政策
活动特殊政策
客服临时口径
`​``

AI 如果拿到这些材料，很容易混合生成一个看似合理但不准确的答案。DBMS 和知识管理系统应该提前标记权威级别、版本、适用范围和状态。

---

**8. 审计、合规与可追溯**

客服场景需要知道：

`​``text
谁查询了用户数据
AI 调用了哪个接口
返回了哪些字段
根据哪些知识库片段回答
是否执行了业务动作
是否修改了订单或工单
用户是否采纳答案
是否发生越权访问
`​``

这些都不能只存在于 AI 上下文里，而要进入审计系统。

数据库和日志系统需要记录：

- 请求 ID。
- 用户 ID。
- 客服 ID。
- 会话 ID。
- 工具调用参数。
- 数据返回摘要。
- 权限判断结果。
- 模型生成内容。
- 引用来源。
- 动作执行结果。
- 异常和拒答原因。

AI 可以生成解释，但系统必须能事后还原事实链路。

---

**9. 复杂聚合和确定性计算**

很多客服问题不是简单问答，而是计算。

例如：

`​``text
我这个月为什么多扣了 38 元？
我的退款金额怎么算？
我还有多少积分？
这个套餐升级后每月多少钱？
这个保单理赔金额是多少？
`​``

这些问题涉及：

- 计费规则。
- 汇总计算。
- 折扣叠加。
- 税费。
- 退款比例。
- 账期。
- 余额。
- 历史变更。
- 四舍五入规则。

AI 可以解释计算逻辑，但不应该自己临时算业务账。正确做法是调用计费系统、订单系统、保单系统或数据库函数，让确定性系统返回结果。

---

**10. 实时状态与事件流**

智能客服里很多数据是实时变化的：

`​``text
物流状态
排队状态
支付状态
风控状态
库存状态
客服在线状态
工单 SLA 倒计时
系统故障告警
`​``

这些需要数据库、缓存、消息队列、事件流系统维护。

AI 的上下文是临时的，不适合维护实时全局状态。AI 应该在需要时调用接口获取最新状态，而不是相信自己之前看到的上下文。

---

**二、AI 整合这些数据的接口是什么**

核心原则是：**不要让 AI 直接裸连数据库，而是让 AI 通过受控接口访问数据能力。**

在 MCP 场景下，MCP 可以作为 AI 与外部系统之间的标准工具协议，但 MCP 本身不是数据库，也不是权限系统。它更像是把后端能力暴露给 AI 的“工具接口层”。

可以分成几类接口。

---

**1. 检索类接口**

用于 RAG 和知识库问答。

典型接口：

`​``text
search_knowledge(query, filters, top_k)
hybrid_search(query, keyword, embedding, filters)
get_document(doc_id)
get_chunk(chunk_id)
get_policy_version(policy_id, effective_time)
search_faq(query, product, region, user_type)
search_ticket_cases(query, filters)
`​``

它们背后可能调用：

`​``text
Elasticsearch / OpenSearch
PostgreSQL full-text search
Milvus / Qdrant / pgvector
知识库系统
对象存储
文档管理系统
`​``

AI 不需要知道底层是 BM25、向量库还是 SQL，只需要拿到结构化返回：

`​``json
{
  "results": [
    {
      "id": "chunk_123",
      "title": "退款政策 2026 版",
      "content": "签收后 7 日内...",
      "source": "policy_refund_2026",
      "version": "2026-04-01",
      "authority": "official",
      "score": 0.87,
      "citation": {
        "document_id": "doc_456",
        "section": "3.2",
        "url": "..."
      }
    }
  ]
}
`​``

---

**2. 业务查询类接口**

用于查询订单、用户、工单、保单、账单等结构化业务数据。

典型接口：

`​``text
get_customer_profile(customer_id)
get_order(order_id)
get_order_status(order_id)
get_refund_eligibility(order_id)
get_invoice_status(invoice_id)
get_ticket(ticket_id)
list_customer_tickets(customer_id)
get_subscription(user_id)
get_billing_summary(user_id, month)
`​``

这类接口应该由业务服务提供，而不是让 AI 直接写 SQL 查生产库。

返回应该是领域对象，而不是数据库原始行：

`​``json
{
  "order_id": "O202605200001",
  "status": "delivered",
  "paid_amount": 299.00,
  "delivered_at": "2026-05-10T13:20:00Z",
  "refund_window_ends_at": "2026-05-17T23:59:59Z",
  "refund_supported": false,
  "reason": "超过 7 天无理由退货期限"
}
`​``

这样 AI 拿到的是可解释的业务事实，而不是自己拼规则。

---

**3. 业务动作类接口**

用于执行操作，必须强约束、幂等、可审计。

典型接口：

`​``text
create_ticket(customer_id, category, description)
update_ticket(ticket_id, fields)
escalate_to_human(ticket_id, reason)
submit_refund_request(order_id, reason)
cancel_order(order_id)
resend_invoice(invoice_id)
send_coupon(customer_id, campaign_id)
`​``

这类接口必须有：

- 权限校验。
- 参数校验。
- 幂等键。
- 风控校验。
- 操作确认。
- 审计日志。
- 可回滚或补偿机制。
- 人工审批机制。

AI 不应该直接修改数据库表。它只能调用有业务语义的动作接口。

---

**4. 规则与决策类接口**

很多客服判断不应该让 AI 自己推断，而应该由规则引擎或决策服务完成。

典型接口：

`​``text
check_refund_policy(order_id)
check_compensation_policy(customer_id, incident_id)
calculate_refund_amount(order_id)
calculate_subscription_price(user_id, plan_id)
check_claim_eligibility(policy_id, case_info)
get_next_best_action(customer_id, context)
`​``

这类接口背后可能是：

`​``text
规则引擎
风控系统
计费系统
权益系统
保单系统
促销系统
`​``

AI 负责解释返回结果：

`​``text
根据系统判断，这笔订单不支持自动退款，原因是签收时间已超过 7 天。不过商品仍在质保期内，可以为你创建售后维修工单。
`​``

---

**5. 上下文与会话接口**

智能客服需要保存会话上下文，但不能完全依赖模型上下文窗口。

典型接口：

`​``text
get_conversation(conversation_id)
append_message(conversation_id, message)
get_user_context(user_id)
save_case_summary(conversation_id, summary)
get_recent_interactions(customer_id)
`​``

这些数据通常存在：

`​``text
会话数据库
Redis
客服系统
CRM
工单系统
日志系统
`​``

AI 可以读取近期上下文，但长期记忆和会话状态应该由系统管理。

---

**6. 观测与评估接口**

RAG 和客服系统要持续优化，需要记录检索和回答质量。

典型接口：

`​``text
log_retrieval(query, results, selected_sources)
log_tool_call(tool_name, params, result_summary)
log_answer(answer, citations, confidence)
submit_feedback(conversation_id, rating, reason)
record_handoff(conversation_id, human_agent_id)
evaluate_answer(question, answer, references)
`​``

这些接口用于：

- 评估召回质量。
- 发现知识缺口。
- 追踪错误答案。
- 分析用户满意度。
- 审计 AI 工具调用。
- 优化知识库和检索策略。

---

**三、MCP 在这里的角色**

MCP 可以理解为 AI 调用外部能力的标准化通道。它不是让 AI 直接“拥有数据库”，而是让数据库、搜索系统、业务系统以工具形式暴露能力。

一个 MCP Server 可以封装：

`​``text
知识库检索工具
订单查询工具
工单创建工具
用户画像查询工具
退款资格判断工具
计费解释工具
日志查询工具
权限校验工具
`​``

AI 看到的是工具描述，例如：

`​``json
{
  "name": "get_refund_eligibility",
  "description": "Check whether an order is eligible for refund according to current business rules.",
  "input_schema": {
    "type": "object",
    "properties": {
      "order_id": { "type": "string" },
      "customer_id": { "type": "string" }
    },
    "required": ["order_id", "customer_id"]
  }
}
`​``

AI 调用：

`​``json
{
  "order_id": "O123456",
  "customer_id": "C888"
}
`​``

MCP Server 再去调用后端业务系统或数据库，并返回受控结果。

关键点是：

`​``text
AI 不直接访问表。
AI 不直接决定权限。
AI 不直接拼生产 SQL。
AI 不直接修改核心业务数据。
AI 通过 MCP 调用经过封装、校验、授权、审计的工具。
`​``

---

**四、应该如何分层**

比较合理的分层可以是下面这样。

`​``text
用户 / 客服入口
  |
AI 交互层
  |
任务理解与编排层
  |
MCP / 工具接入层
  |
领域服务与数据访问层
  |
数据库、搜索引擎、向量库、对象存储、消息系统
  |
数据治理、权限、审计、评估体系
`​``

展开来看。

---

**1. 用户交互层**

负责和用户对话。

包括：

`​``text
Web 聊天窗口
App 客服入口
企业微信 / 飞书 / Slack
电话客服转写
人工坐席工作台
`​``

这一层主要处理：

- 用户输入。
- 多轮对话。
- 展示答案。
- 展示引用。
- 展示操作按钮。
- 人工接管。
- 用户反馈。

这一层不直接查数据库。

---

**2. AI 应用层**

这一层是大模型所在位置，负责理解和生成。

主要能力：

`​``text
意图识别
问题改写
多轮上下文理解
工具选择
证据组织
自然语言回答
追问澄清
拒答
转人工
生成摘要
生成工单描述
`​``

例如用户问：

`​``text
我这个订单为什么退不了？
`​``

AI 应用层应该判断：

`​``text
这是售后/退款问题
需要订单号
需要查询订单状态
需要查询退款资格
可能需要解释政策
如果信息不足要追问
`​``

它不应该自己凭经验回答。

---

**3. 任务编排层**

这一层负责把用户问题变成可执行流程。

例如：

`​``text
问题：我这个订单为什么不能退款？

编排流程：
1. 获取用户身份。
2. 查询用户最近订单。
3. 如果订单不明确，追问用户选择订单。
4. 查询订单详情。
5. 调用退款资格判断。
6. 检索相关退款政策。
7. 生成解释。
8. 如果可走其他售后路径，提供创建工单入口。
`​``

这一层可以由 Agent、工作流引擎、规则编排或代码逻辑实现。

关键是：复杂业务不能完全交给 LLM 自由发挥。稳定流程应该产品化、服务化。

---

**4. MCP / 工具接口层**

这一层是 AI 与外部系统之间的边界。

它负责：

`​``text
工具注册
参数 schema
工具描述
权限上下文传递
参数校验
返回结构规范
调用审计
错误标准化
超时控制
重试控制
限流
`​``

MCP 工具应该是面向业务能力的，例如：

`​``text
get_order_status
check_refund_eligibility
search_policy
create_support_ticket
get_customer_recent_orders
calculate_refund_amount
handoff_to_human
`​``

不建议暴露成：

`​``text
execute_sql
read_table
update_database
`​``

除非是在内部分析场景，并且有只读权限、SQL 审查、查询沙箱和严格审计。

---

**5. 领域服务层**

这一层封装业务逻辑，是 AI 真正应该调用的后端能力。

例如：

`​``text
订单服务
用户服务
售后服务
工单服务
计费服务
权益服务
风控服务
知识库服务
搜索服务
权限服务
审计服务
`​``

它负责：

- 业务规则。
- 状态流转。
- 数据校验。
- 权限判断。
- 事务处理。
- 幂等控制。
- 返回领域对象。
- 屏蔽底层数据库细节。

AI 调用的是“查订单状态”，不是“查 orders 表”。

---

**6. 数据访问层**

这一层负责具体访问数据库和搜索系统。

包括：

`​``text
Repository
DAO
Query Service
Search Gateway
Vector Retrieval Gateway
Cache Gateway
Event Consumer
`​``

它处理：

- SQL 查询。
- 索引查询。
- 向量检索。
- 缓存读取。
- 分页排序。
- 数据转换。
- 查询优化。
- 连接池。
- 读写分离。
- 限流降级。

AI 不应该感知这一层的细节。

---

**7. 数据存储层**

这是 DBMS 和相关存储系统所在的位置。

不同数据用不同系统管理：

`​``text
用户、订单、工单：关系型数据库，如 PostgreSQL、MySQL、OceanBase
聊天记录、事件日志：PostgreSQL、MongoDB、ClickHouse、Kafka
知识库文档：对象存储 + 元数据库
全文检索：Elasticsearch、OpenSearch、Solr
向量检索：Milvus、Qdrant、pgvector、Weaviate
分析报表：ClickHouse、Doris、Snowflake、BigQuery
缓存状态：Redis
关系网络：Neo4j、JanusGraph 或关系型图建模
文件原文：S3、MinIO、OSS、NAS
`​``

RAG 和客服通常不是一个数据库能全部解决，而是多种存储组合。

---

**8. 数据治理与安全层**

这层应该贯穿所有层，而不是最后补上。

包括：

`​``text
身份认证
权限控制
字段脱敏
租户隔离
密级管理
数据加密
访问审计
数据血缘
版本管理
数据质量校验
生命周期管理
合规策略
`​``

尤其是 RAG 场景，权限过滤不能只在最后回答前做。它应该在：

`​``text
文档同步
索引构建
检索召回
重排序
上下文构建
答案生成
引用展示
日志记录
`​``

每个阶段都生效。

---

**9. 评估与观测层**

这一层用于判断系统是否真的可用。

需要监控：

`​``text
问题解决率
转人工率
首响时间
平均处理时长
召回命中率
答案正确率
引用准确率
幻觉率
拒答准确率
工具调用成功率
接口延迟
数据库慢查询
权限拦截次数
用户满意度
`​``

还要保存测试集：

`​``text
标准问题
标准答案
标准证据
标准工具调用路径
边界问题
拒答问题
权限测试问题
`​``

没有这一层，RAG 客服很难持续优化。

---

**五、推荐的整体架构**

可以这样抽象：

`​``text
[用户]
  |
  v
[聊天入口 / 坐席工作台]
  |
  v
[AI 应用层]
- 意图理解
- 多轮对话
- 工具选择
- 答案生成
- 引用展示
  |
  v
[任务编排层]
- 查询改写
- 流程控制
- 澄清问题
- 多工具调用
- 人工接管判断
  |
  v
[MCP / Tool Gateway]
- 工具 schema
- 参数校验
- 权限上下文
- 调用审计
- 错误处理
  |
  v
[领域服务层]
- 订单服务
- 用户服务
- 工单服务
- 售后服务
- 知识库服务
- 计费服务
- 权限服务
  |
  v
[数据访问与检索层]
- SQL 查询
- 全文检索
- 向量检索
- 混合召回
- 缓存读取
- 事件查询
  |
  v
[数据存储层]
- RDBMS
- 搜索引擎
- 向量数据库
- 对象存储
- 日志库
- 缓存
- 数据仓库
`​``

横向贯穿：

`​``text
权限安全
审计日志
数据治理
质量评估
监控告警
成本控制
`​``

---

**六、一个具体例子**

用户问：

`​``text
我买的手机屏幕坏了，能不能免费维修？
`​``

合理流程不是 AI 直接回答，而是：

`​``text
1. AI 理解：这是售后维修 / 质保问题。
2. AI 追问或识别订单：需要知道是哪一笔订单。
3. 调用 get_recent_orders，获取用户近期手机订单。
4. 用户选择订单。
5. 调用 get_order_detail，获取购买时间、商品型号、保修状态。
6. 调用 check_warranty_eligibility，判断是否在保修期、是否支持免费维修。
7. 调用 search_policy，检索对应商品的质保政策。
8. AI 生成回答，并引用政策。
9. 如果符合条件，调用 create_repair_ticket。
10. 全流程写入审计日志。
`​``

其中 AI 做的是：

`​``text
理解问题
组织流程
调用工具
解释结果
生成自然语言答复
`​``

数据库和业务系统做的是：

`​``text
存储订单
查询状态
校验质保
执行规则
创建工单
保证事务
记录审计
`​``

---

**七、接口设计原则**

在 RAG 问答和智能客服里，AI 整合数据的接口应该遵循这些原则。

**1. 面向业务能力，而不是面向数据库表**

推荐：

`​``text
check_refund_eligibility(order_id)
get_customer_orders(customer_id)
create_support_ticket(...)
search_policy(...)
`​``

不推荐：

`​``text
select * from orders
select * from refund_rules
update tickets set ...
`​``

---

**2. 返回结构化事实，而不是一大段不可控文本**

推荐返回：

`​``json
{
  "eligible": false,
  "reason_code": "REFUND_WINDOW_EXPIRED",
  "reason_text": "订单已超过 7 天无理由退货期限",
  "policy_reference": {
    "document_id": "refund_policy_2026",
    "section": "2.1"
  },
  "alternative_actions": ["REPAIR_SERVICE", "HUMAN_AGENT"]
}
`​``

这样 AI 才能稳定解释。

---

**3. 所有工具都要带权限上下文**

每次调用都应该知道：

`​``text
当前用户是谁
当前客服是谁
当前租户是谁
当前会话是什么
当前渠道是什么
允许访问哪些字段
允许执行哪些动作
`​``

工具调用不应该只传业务参数，还要带安全上下文。

---

**4. 写操作必须确认和幂等**

例如：

`​``text
create_refund_request
cancel_order
send_coupon
change_address
`​``

这些接口必须：

- 有幂等键。
- 有操作确认。
- 有权限校验。
- 有业务状态校验。
- 有审计日志。
- 必要时需要人工审批。

AI 不应该在用户没有明确授权时直接执行高影响动作。

---

**5. 检索结果必须带来源和版本**

RAG 接口返回的内容必须包含：

`​``text
文档 ID
标题
章节
版本
发布时间
生效时间
来源系统
权威级别
权限标签
召回分数
引用位置
`​``

否则 AI 即使答对了，也无法证明依据。

---

**6. 工具返回应区分事实、规则和建议**

例如：

`​``json
{
  "facts": {
    "order_status": "delivered",
    "delivered_at": "2026-05-10"
  },
  "rule_result": {
    "refund_supported": false,
    "reason": "超过退款期限"
  },
  "suggested_actions": [
    {
      "action": "create_repair_ticket",
      "label": "申请维修"
    }
  ]
}
`​``

这样 AI 不会把建议误说成事实。

---

**八、理想分工**

最终分工可以总结为：

`​``text
AI 是理解与编排层。
MCP 是工具协议与接入层。
业务服务是规则与动作层。
DBMS 是事实、状态、一致性和查询层。
搜索/向量库是非结构化知识召回层。
审计与治理系统是安全可信层。
`​``

更简洁地说：

`​``text
AI 负责“问什么、怎么解释、下一步做什么”。
数据库负责“事实是什么、状态是否一致、谁能访问、操作是否成功”。
MCP 负责“把 AI 的意图转换成受控的工具调用”。
`​``

---

**九、最关键的边界**

在 RAG 问答与智能客服中，最重要的边界是：

`​``text
AI 可以决定调用哪个工具，但不能绕过工具直接操作数据。
AI 可以解释业务结果，但不能替代业务规则引擎。
AI 可以组织证据，但不能伪造数据来源。
AI 可以发起业务动作，但不能成为事务系统。
AI 可以利用上下文，但不能成为长期状态存储。
AI 可以辅助判断，但高风险场景必须有人审或规则系统兜底。
`​``

所以，即使 AI 语义理解很强、本地执行能力很强、还能通过 MCP 连接外部系统，数据库管理系统仍然是 RAG 问答与智能客服的基础设施核心。AI 把数据“用起来”，DBMS 保证数据“是真的、准的、新的、安全的、可追溯的、可并发访问的”。

<!-- /message -->
<!-- message:id=msg_18c71f77a23b23b5 role=user createdAt=2026-05-20T11%3A01%3A24.586Z -->

## User

看来mcp基于一个桥接的地位，也就是那个中间层，但它的接口似乎很多，这个问题如何解决？和数据库的交互本来sql能力很强，但暴露出业务接口就很多，还有权限如何透传

<!-- /message -->
<!-- message:id=msg_0da96e33c9233b21 role=assistant createdAt=2026-05-20T11%3A03%3A37.207Z runId=run_cc3e41b9483eada6 status=succeeded -->

## Assistant

你的判断是对的：**MCP 处在桥接层，但如果把所有数据库表、所有业务动作都一股脑暴露成 MCP tool，接口会爆炸，权限也会变得不可控。**

解决思路不是“给 AI 暴露更多接口”，而是把 MCP 从“接口清单”升级成一个**受控能力网关**：

`​``text
AI
  |
MCP / Tool Gateway
  |
领域能力层 / 查询能力层 / 权限策略层
  |
业务服务 / 数据服务 / DBMS / 搜索系统
`​``

核心原则是：

`​``text
AI 不应该直接面对数据库复杂性。
MCP 不应该等于数据库 API。
MCP tool 应该表达“受控能力”，而不是暴露“底层资源”。
`​``

---

**一、接口太多的问题，本质是什么**

MCP 接口多，通常来自三种误区。

第一种是把每个数据库表都包装成工具：

`​``text
get_user
get_order
get_order_item
get_refund
get_coupon
get_invoice
get_ticket
get_ticket_log
...
`​``

这样很快就会变成“工具版数据库”，AI 要自己理解表关系、字段含义、业务规则，风险很高。

第二种是把每个业务动作都做成细粒度工具：

`​``text
check_order_status
check_refund_window
check_product_returnable
check_coupon_used
check_payment_status
check_logistics_status
...
`​``

这样工具数量也会爆炸，而且 AI 需要自己编排复杂业务判断。

第三种是直接暴露通用 SQL：

`​``text
execute_sql(sql)
`​``

这看起来接口最少，但权限、安全、性能、注入、误查、越权、慢查询、写操作风险都会集中爆发。

所以要区分三类接口：

`​``text
低层接口：表、SQL、索引、存储。
中层接口：领域对象、查询视图、规则判断。
高层接口：面向任务的业务能力。
`​``

MCP 最适合暴露中高层能力，而不是底层表。

---

**二、不要按表设计 MCP tool，要按“任务域”设计**

RAG 问答和智能客服里，用户不是来查表的，而是来解决问题的。

所以 MCP 工具不应该围绕数据库表设计，而应该围绕任务域设计。

例如不要这样设计：

`​``text
get_orders_table
get_refunds_table
get_products_table
get_refund_rules_table
get_order_items_table
`​``

更合理的是：

`​``text
get_customer_context
get_order_summary
check_refund_eligibility
explain_billing_issue
search_policy
create_support_ticket
handoff_to_human
`​``

它们背后可以查很多表，但 AI 不需要知道。

例如：

`​``text
check_refund_eligibility(order_id, customer_id)
`​``

背后可能查：

`​``text
orders
order_items
payments
refunds
products
product_categories
refund_rules
promotions
risk_flags
logistics_events
`​``

但 MCP 返回的是业务判断结果：

`​``json
{
  "eligible": false,
  "reason_code": "REFUND_WINDOW_EXPIRED",
  "reason_text": "订单已超过 7 天无理由退货期限",
  "facts": {
    "delivered_at": "2026-05-10T13:20:00Z",
    "refund_window_days": 7,
    "refund_window_ended_at": "2026-05-17T23:59:59Z"
  },
  "policy_reference": {
    "document_id": "refund_policy_2026",
    "section": "2.1"
  },
  "alternative_actions": [
    "create_repair_ticket",
    "handoff_to_human"
  ]
}
`​``

这样 AI 只负责解释，不负责乱推断。

---

**三、可以用“三层工具模型”控制接口数量**

建议把 MCP tools 分成三层，不要全都暴露给模型。

`​``text
L1：任务级工具
L2：领域查询工具
L3：受控数据查询工具
`​``

---

**1. L1：任务级工具，给 AI 优先使用**

这是最推荐暴露给 AI 的接口，数量应该少，但能力强。

例如客服系统可以只有几十个核心工具：

`​``text
resolve_refund_question
resolve_warranty_question
resolve_billing_question
resolve_logistics_question
resolve_account_question
create_support_ticket
handoff_to_human
search_customer_policy
get_customer_case_summary
`​``

这些工具是“面向任务”的。

例如：

`​``text
resolve_refund_question(customer_id, order_id?, question)
`​``

它内部完成：

`​``text
识别订单
查询订单
判断退款资格
查询政策
生成结构化事实
返回建议动作
`​``

AI 拿到结果后再组织语言。

优点：

- 工具数量少。
- AI 编排压力小。
- 权限容易统一控制。
- 业务逻辑稳定。
- 适合高频客服场景。

缺点：

- 后端建设成本较高。
- 灵活性不如自由 SQL。
- 新业务需要新增或扩展任务工具。

---

**2. L2：领域查询工具，给复杂问题和 Agent 使用**

这类工具不是按表，而是按领域对象。

例如：

`​``text
get_customer_profile
get_customer_recent_orders
get_order_detail
get_ticket_history
get_billing_summary
get_refund_status
get_policy_by_context
search_similar_cases
`​``

这类工具适合 AI 多步编排，但仍然屏蔽底层表结构。

例如：

`​``text
get_order_detail(order_id)
`​``

返回的不是 orders 表原始字段，而是订单领域模型：

`​``json
{
  "order_id": "O123",
  "status": "delivered",
  "items": [
    {
      "sku": "PHONE-001",
      "name": "某手机",
      "category": "mobile_phone",
      "warranty_months": 12,
      "returnable": true
    }
  ],
  "payment": {
    "paid_amount": 3999,
    "currency": "CNY"
  },
  "logistics": {
    "delivered_at": "2026-05-10T13:20:00Z"
  }
}
`​``

优点：

- 比 L1 灵活。
- 比直接 SQL 安全。
- 有利于复用。
- 适合复杂对话和多工具组合。

缺点：

- AI 仍需要一定编排能力。
- 需要设计领域模型。

---

**3. L3：受控数据查询工具，只给低风险分析场景使用**

有些场景确实需要 SQL 能力，比如运营分析、内部数据问答、报表探索。

但这类工具不能简单暴露：

`​``text
execute_sql(sql)
`​``

而应该是受控查询接口：

`​``text
query_dataset(dataset_id, dimensions, metrics, filters, time_range, limit)
`​``

或者：

`​``text
run_approved_query(query_template_id, params)
`​``

或者在严格环境下：

`​``text
generate_sql(intent) -> review_sql -> execute_readonly_sql
`​``

重点是：AI 可以生成查询意图，但执行层要做约束。

例如不要让 AI 直接查任意表：

`​``sql
SELECT * FROM users;
`​``

而是让它只能查授权数据集：

`​``json
{
  "dataset": "customer_support_orders",
  "metrics": ["order_count", "refund_count"],
  "dimensions": ["reason_code"],
  "filters": {
    "tenant_id": "T001",
    "created_at": {
      "from": "2026-05-01",
      "to": "2026-05-20"
    }
  },
  "limit": 100
}
`​``

由数据服务生成 SQL 并执行。

---

**四、SQL 很强，为什么不直接给 AI 用 SQL**

SQL 的确很强，但问题是：**SQL 是数据库能力，不是业务安全边界。**

直接暴露 SQL 会遇到几个问题。

**1. 权限粒度不够业务化**

数据库权限通常是：

`​``text
库
表
列
行
视图
角色
`​``

但客服权限往往是：

`​``text
这个客服能不能看这个客户？
这个租户能不能看这笔订单？
这个渠道能不能看手机号？
这个场景能不能看风控标签？
这个用户是否授权查看历史记录？
`​``

这些不是简单 SQL 权限能完整表达的。

---

**2. AI 容易写出危险查询**

即使只读，也可能有风险：

`​``sql
SELECT * FROM customers LIMIT 10000;
`​``

或者：

`​``sql
SELECT phone, address, id_card FROM users;
`​``

或者复杂 join 导致慢查询：

`​``sql
SELECT ...
FROM orders o
JOIN order_items i
JOIN refunds r
JOIN tickets t
JOIN logs l
...
`​``

这会带来：

`​``text
越权
隐私泄漏
慢查询
数据库压力
结果过大
字段误解
业务规则绕过
`​``

---

**3. 数据库字段不是业务语义**

比如数据库里可能有字段：

`​``text
status = 3
type = 2
flag = 1
risk_code = R17
`​``

AI 可能不知道真实含义。

更严重的是，同一个字段在不同系统里含义不一致：

`​``text
order.status
refund.status
ticket.status
payment.status
`​``

直接 SQL 会让 AI 过度依赖底层实现细节。

---

**4. SQL 不能替代业务规则**

例如用户问：

`​``text
我能不能退款？
`​``

这不是一条 SQL 能可靠解决的问题。

它涉及：

`​``text
订单状态
支付状态
物流状态
商品类目
活动规则
售后规则
地区政策
历史退款
风控规则
人工审批状态
`​``

正确答案应该来自：

`​``text
check_refund_eligibility
`​``

而不是 AI 临时拼 SQL。

---

**五、但 SQL 能力仍然可以保留，只是要放在正确层**

比较好的做法是：**SQL 不直接暴露给 AI，而是沉到数据服务层。**

架构可以是：

`​``text
AI
  |
MCP tools
  |
Query Service / Domain Service
  |
SQL Builder / ORM / Policy Engine
  |
DBMS
`​``

AI 调用：

`​``text
get_order_summary(order_id)
`​``

服务内部执行 SQL：

`​``sql
SELECT ...
FROM orders
JOIN payments
JOIN logistics
WHERE order_id = ?
  AND tenant_id = ?
`​``

AI 调用：

`​``text
query_support_metrics(...)
`​``

数据服务内部生成受控 SQL：

`​``sql
SELECT reason_code, COUNT(*)
FROM support_tickets
WHERE tenant_id = ?
  AND created_at BETWEEN ? AND ?
GROUP BY reason_code
LIMIT 100;
`​``

也就是说：

`​``text
SQL 仍然存在，而且很重要。
但 SQL 是后端实现细节，不是 AI 的公开接口。
`​``

---

**六、如何减少 MCP 接口数量**

可以用几个办法。

---

**1. 按领域聚合，而不是按动作碎片化**

不要设计成：

`​``text
get_order_status
get_order_payment
get_order_logistics
get_order_items
get_order_refund
get_order_coupon
`​``

可以聚合成：

`​``text
get_order_context(order_id, purpose)
`​``

例如：

`​``json
{
  "order_id": "O123",
  "purpose": "refund_explanation"
}
`​``

服务根据 purpose 返回必要字段：

`​``json
{
  "order": {...},
  "payment": {...},
  "logistics": {...},
  "refund": {...},
  "policy_refs": [...]
}
`​``

这样工具数量减少，返回也更贴近任务。

---

**2. 用“意图参数”替代多个相似工具**

例如不要有：

`​``text
search_refund_policy
search_warranty_policy
search_shipping_policy
search_invoice_policy
`​``

可以设计成：

`​``text
search_policy(query, policy_domain, context)
`​``

其中：

`​``json
{
  "policy_domain": "refund",
  "context": {
    "product_category": "mobile_phone",
    "region": "CN-SH",
    "order_time": "2026-05-01"
  }
}
`​``

---

**3. 用资源模型统一 CRUD**

对于低风险对象，可以用统一资源接口，而不是每个对象一个工具。

例如：

`​``text
get_resource(resource_type, resource_id, view)
list_resources(resource_type, filters, view, limit)
`​``

但 resource_type 必须是白名单：

`​``text
order
ticket
invoice
refund_request
policy_document
`​``

view 也必须是白名单：

`​``text
summary
detail
customer_visible
agent_visible
audit
`​``

例如：

`​``json
{
  "resource_type": "order",
  "resource_id": "O123",
  "view": "customer_visible"
}
`​``

这比裸 SQL 安全，比为每个对象写工具更少。

---

**4. 用查询模板替代自由查询**

对于统计和分析类问题：

`​``text
run_query_template(template_id, params)
`​``

例如：

`​``text
refund_reason_distribution
ticket_volume_by_category
top_complaint_products
billing_dispute_trend
`​``

AI 只能选择模板和参数，不能任意写 SQL。

---

**5. 建立 Tool Catalog，而不是一次性暴露所有工具**

MCP tool 不一定要全量暴露给模型。可以先有一个工具目录检索能力：

`​``text
search_tools(task_description)
get_tool_schema(tool_name)
call_tool(tool_name, args)
`​``

或者由编排层根据意图动态挂载工具。

例如：

`​``text
用户问退款，只暴露退款、订单、政策、工单相关工具。
用户问发票，只暴露发票、订单、计费、工单相关工具。
`​``

这样可以降低模型选择工具的复杂度，也减少误调用。

---

**七、权限如何透传**

权限透传要解决的是：**AI 调工具时，后端怎么知道“谁在问、代表谁问、允许看什么、允许做什么”。**

不能只靠 AI 在参数里写：

`​``json
{
  "user_id": "U123"
}
`​``

因为这可以被提示注入或模型误填影响。

正确做法是：权限上下文由系统注入，不由模型生成。

---

**1. 建立 Auth Context**

每次 MCP 调用都必须携带系统可信的上下文：

`​``json
{
  "auth_context": {
    "tenant_id": "T001",
    "end_user_id": "U123",
    "agent_id": "A456",
    "channel": "web_chat",
    "session_id": "S789",
    "roles": ["customer"],
    "scopes": [
      "order:read:own",
      "ticket:create:own",
      "policy:read"
    ],
    "data_region": "CN",
    "locale": "zh-CN"
  }
}
`​``

这个 `auth_context` 应该来自：

`​``text
登录态
网关
客服系统
IAM
会话服务
`​``

而不是来自用户自然语言，也不是来自模型生成。

---

**2. MCP Server 不做最终信任，后端服务也要校验**

权限不能只在 MCP 层校验一次。更稳妥是多层校验：

`​``text
API Gateway：认证、租户、基础 scope
MCP Gateway：工具级权限、参数校验、审计
Domain Service：业务权限、对象归属、状态校验
DBMS：行级权限、列级权限、视图隔离
`​``

例如 AI 调：

`​``text
get_order_detail(order_id = O999)
`​``

每一层都要约束：

`​``text
这个 session 是否有效？
当前用户是否能调用 get_order_detail？
O999 是否属于当前用户或当前客服可服务范围？
返回字段是否需要脱敏？
是否需要记录审计？
`​``

不能因为 MCP 已经校验了，领域服务就默认信任。

---

**3. 权限最好使用“能力 token”或“下游 token 交换”**

比较成熟的做法是：AI 调用工具时，MCP Server 不直接拿超级权限访问后端，而是使用当前用户/客服的权限上下文换取下游访问 token。

流程：

`​``text
用户登录
  |
生成用户会话 token
  |
AI 发起工具调用
  |
MCP Gateway 使用会话 token / service token 做 token exchange
  |
获得 scoped downstream token
  |
调用领域服务
  |
领域服务按 token 权限返回数据
`​``

这个 token 应该是短期、限域、可审计的，例如：

`​``text
tenant:T001
subject:U123
actor:AI_ASSISTANT
scopes:order:read:own,ticket:create:own
expires_in:60s
`​``

如果是客服代表用户操作，还要区分：

`​``text
subject：数据主体，通常是客户
actor：操作者，可能是客服或 AI
delegation：是否代理操作
`​``

例如：

`​``json
{
  "subject": "customer:U123",
  "actor": "agent:A456",
  "assistant": "ai:customer-service-bot",
  "delegation": "agent_assisted",
  "scopes": ["order:read", "ticket:create"]
}
`​``

这样审计时能知道：

`​``text
谁的数据被访问？
是谁发起的？
AI 是否参与？
代表谁操作？
`​``

---

**4. 使用 Policy Decision Point / Policy Enforcement Point**

可以把权限抽象成：

`​``text
PDP：策略决策点
PEP：策略执行点
`​``

例如：

`​``text
PDP：OPA、Cedar、Casbin、自研权限中心
PEP：API Gateway、MCP Gateway、领域服务、DB 查询层
`​``

调用流程：

`​``text
MCP 工具调用
  |
PEP 收集上下文：
- subject
- actor
- tenant
- resource
- action
- environment
  |
请求 PDP 判断：
allow / deny / mask / require_approval
  |
执行或拒绝
`​``

策略例子：

`​``text
客户只能查看自己的订单。
普通客服只能查看当前会话客户的订单。
高级客服可以查看本租户订单，但手机号要脱敏。
风控标签只有风控角色可见。
退款金额超过阈值需要人工确认。
`​``

PDP 返回不一定只是允许/拒绝，也可以返回数据处理要求：

`​``json
{
  "decision": "allow",
  "field_masks": {
    "phone": "partial",
    "id_card": "hidden",
    "risk_label": "hidden"
  },
  "max_rows": 20,
  "audit_level": "high"
}
`​``

---

**5. 字段级脱敏不要交给 AI**

不要让后端返回完整数据，然后要求 AI “不要展示”。

错误做法：

`​``json
{
  "phone": "13812345678",
  "id_card": "310101199001011234"
}
`​``

然后提示词里写：

`​``text
不要泄露用户隐私。
`​``

正确做法是接口直接返回已脱敏数据：

`​``json
{
  "phone": "138****5678",
  "id_card": null
}
`​``

或者根据权限返回：

`​``json
{
  "phone": {
    "value": "138****5678",
    "masked": true,
    "mask_reason": "FIELD_PERMISSION_DENIED"
  }
}
`​``

AI 只能看到它应该看到的数据。

---

**八、推荐的接口分层方式**

可以采用下面这个分层。

`​``text
[AI / Agent]
  |
  | 只看到少量任务级工具和领域工具
  v
[MCP Tool Gateway]
  - tool catalog
  - schema validation
  - dynamic tool exposure
  - auth context injection
  - audit logging
  - rate limit / timeout
  |
  v
[Capability Layer]
  - refund capability
  - billing capability
  - warranty capability
  - logistics capability
  - ticket capability
  - policy search capability
  |
  v
[Domain Service Layer]
  - order service
  - customer service
  - payment service
  - refund service
  - policy service
  - ticket service
  |
  v
[Data Access Layer]
  - SQL builder
  - repository
  - search gateway
  - vector retrieval gateway
  - cache gateway
  |
  v
[Storage Layer]
  - RDBMS
  - search engine
  - vector database
  - object storage
  - Redis
  - event log
`​``

横向贯穿：

`​``text
IAM / Policy Engine / Audit / Observability / Data Governance
`​``

这里最重要的是多了一层：

`​``text
Capability Layer
`​``

它专门解决“接口太多”的问题。

它把一堆细碎服务能力组合成少量业务能力。

例如：

`​``text
refund capability
`​``

内部可以调用：

`​``text
order service
payment service
logistics service
refund service
policy service
risk service
ticket service
`​``

但对 MCP 只暴露：

`​``text
resolve_refund_question
check_refund_eligibility
submit_refund_request
`​``

---

**九、一个实用的 MCP tool 设计范式**

每个工具最好符合这个结构：

`​``text
工具名：动词 + 业务对象 / 任务
输入：业务必要参数 + 可选上下文
权限：声明需要哪些 scope
输出：事实 + 规则结果 + 引用 + 可执行动作
审计：记录调用原因、参数摘要、结果摘要
`​``

例如：

`​``json
{
  "name": "resolve_refund_question",
  "description": "Resolve a customer's refund-related question using order facts, refund rules, and policy references.",
  "required_scopes": [
    "order:read:own",
    "refund:check",
    "policy:read"
  ],
  "input_schema": {
    "type": "object",
    "properties": {
      "order_id": {
        "type": "string"
      },
      "question": {
        "type": "string"
      }
    },
    "required": ["question"]
  }
}
`​``

注意这里没有让模型传：

`​``text
tenant_id
user_id
role
permission
`​``

这些由系统注入。

返回：

`​``json
{
  "answer_facts": {
    "order_status": "delivered",
    "delivered_at": "2026-05-10T13:20:00Z",
    "paid_amount": 299.00
  },
  "decision": {
    "refund_supported": false,
    "reason_code": "REFUND_WINDOW_EXPIRED",
    "reason_text": "超过 7 天无理由退货期限"
  },
  "citations": [
    {
      "source": "refund_policy_2026",
      "section": "2.1",
      "effective_from": "2026-04-01"
    }
  ],
  "suggested_actions": [
    {
      "action": "create_repair_ticket",
      "requires_confirmation": true
    }
  ]
}
`​``

AI 的回答就基于这个结构化结果。

---

**十、什么时候可以给 AI SQL 能力**

不是绝对不能给，而是要分场景。

**不建议给自由 SQL 的场景：**

`​``text
面向终端用户的客服
涉及个人隐私
涉及金融、医疗、合同、订单
涉及生产库
涉及写操作
涉及多租户
涉及强合规
`​``

**可以有限给 SQL 能力的场景：**

`​``text
内部分析助手
只读数据仓库
脱敏数据集
低敏统计数据
开发测试环境
有 SQL 审核器
有查询限流
有结果行数限制
有审计
`​``

即便给，也建议是：

`​``text
语义层查询 > SQL 模板 > 只读 SQL > 自由 SQL
`​``

优先级从高到低：

`​``text
query_metric(dataset, metrics, dimensions, filters)
run_approved_query(template_id, params)
run_readonly_sql(sql)
execute_any_sql(sql)
`​``

生产客服系统里，通常只应该到前两层。

---

**十一、推荐的权限透传流程**

一个完整流程可以这样设计：

`​``text
1. 用户登录客服入口，获得 session。
2. 用户向 AI 提问。
3. AI 判断需要调用 resolve_refund_question。
4. 编排层发起 MCP tool call。
5. MCP Gateway 从会话系统读取 auth_context，而不是让 AI 填。
6. MCP Gateway 校验此会话是否允许调用该 tool。
7. MCP Gateway 将 auth_context 转成下游 scoped token。
8. Capability Service 收到 token。
9. Capability Service 调用 Policy Engine 判断对象级权限。
10. Domain Service 查询 DB 时自动带 tenant_id、subject_id、scope。
11. DB 层使用视图、RLS 或查询条件进一步限制。
12. 返回结果前做字段脱敏。
13. MCP Gateway 记录工具调用审计。
14. AI 只拿到脱敏、授权、结构化结果。
15. AI 生成回答和引用。
`​``

可以概括为：

`​``text
身份不由 AI 声明。
权限不由 AI 判断。
脱敏不由 AI 执行。
SQL 不由 AI 直接控制。
审计不依赖 AI 自觉记录。
`​``

---

**十二、最终建议**

最实用的方案是：

`​``text
1. MCP 暴露少量任务级工具，覆盖高频客服场景。
2. MCP 暴露适量领域查询工具，支持复杂对话编排。
3. 不直接暴露数据库表。
4. 不默认暴露自由 SQL。
5. 对分析类问题使用语义数据集、查询模板或只读受控 SQL。
6. 权限上下文由系统注入，通过 token exchange 透传到领域服务。
7. 权限校验分布在 MCP Gateway、领域服务和 DBMS 多层。
8. 字段级脱敏在后端完成，不交给模型。
9. 工具目录动态暴露，按当前意图和权限选择可用工具。
10. 所有工具调用、数据访问、业务动作都写审计。
`​``

一句话总结：

`​``text
MCP 不应该成为“万能数据库入口”，而应该成为“AI 可调用的受控业务能力网关”。
`​``

SQL 仍然是数据库交互的核心能力，但它应该藏在领域服务、查询服务、语义层和权限策略之后。AI 通过 MCP 表达意图，后端用 SQL、索引、规则引擎和事务系统完成可靠执行。权限则通过系统注入的 auth context、短期 scoped token、策略引擎、字段脱敏和数据库行列级控制逐层透传。

<!-- /message -->
<!-- message:id=msg_83e1a297fd9b416d role=user createdAt=2026-05-20T11%3A18%3A25.298Z -->

## User

看起来mcp应该面向业务设计，而非底层系统。而现在的agent无非是换了一种方法来实现业务，具有对应关系。而ai的上述独特的能力在于能代替人决策，请澄清这一论断，并结合palantir说明这一范式的转变

<!-- /message -->
<!-- message:id=msg_73498936d79a77fe role=assistant createdAt=2026-05-20T11%3A20%3A28.682Z runId=run_d3ddec41f3394480 status=succeeded -->

## Assistant

你的判断基本成立，但“AI 的独特能力在于能代替人决策”这句话需要澄清。更准确的说法是：

`​``text
AI 不是简单替代人类决策，而是把一部分原本由人完成的“理解、判断、权衡、编排、执行建议”转移到机器系统中。
但最终责任、边界、目标函数、权限和高风险裁决，仍然需要由组织制度和人来定义。
`​``

也就是说，AI 的价值不是“它像人一样拍板”，而是它能在受控业务语义和工具边界内，把过去依赖人工判断的中间环节自动化。

---

**一、MCP 应该面向业务，而不是面向底层系统**

你前面的结论是对的。MCP 如果只是把数据库、搜索引擎、文件系统、ERP、CRM、工单系统包装成一堆底层接口，它只是“系统适配器”。这会导致：

`​``text
接口多
语义弱
权限难控
AI 不知道业务含义
业务规则被绕过
审计困难
执行风险大
`​``

真正合理的 MCP tool 不应该长这样：

`​``text
query_orders_table
query_refunds_table
query_customer_table
execute_sql
search_index
update_ticket_row
`​``

而应该长这样：

`​``text
check_refund_eligibility
resolve_billing_dispute
get_customer_service_context
search_applicable_policy
create_support_ticket
submit_refund_request
handoff_to_human
`​``

也就是说，MCP 应该暴露的是**业务能力**，不是底层系统能力。

底层系统仍然存在：

`​``text
数据库
搜索引擎
向量库
对象存储
消息队列
规则引擎
ERP
CRM
工单系统
`​``

但它们应该藏在业务能力之后。

正确分层是：

`​``text
AI / Agent
  |
MCP / Tool Gateway
  |
业务能力层 Capability Layer
  |
领域服务层 Domain Services
  |
数据访问层 Data Access
  |
数据库 / 搜索 / 向量库 / 外部系统
`​``

其中 MCP 的职责不是“暴露所有接口”，而是把 AI 的意图接入到受控业务能力中。

---

**二、Agent 确实和传统业务系统有对应关系**

你说“现在的 agent 无非是换了一种方法来实现业务，具有对应关系”，这个判断也基本正确。

传统业务系统里有这些东西：

`​``text
用户界面
业务流程
状态机
规则引擎
服务接口
数据库事务
权限系统
审计日志
人工审批
报表分析
`​``

Agent 时代并没有把这些东西消灭，而是换了一种组织方式。

对应关系大致是：

| 传统业务系统 | Agent / AI 系统中的对应物 |
|---|---|
| 页面菜单 | 自然语言入口 / 多模态入口 |
| 用户点击按钮 | Agent 调用 tool / action |
| 固定工作流 | Agent 编排 / 动态流程规划 |
| 业务规则 | 规则工具 / 函数 / policy engine |
| SQL 查询 | 领域查询工具 / 语义查询 / 受控数据接口 |
| 人工判断 | 模型推理 / 推荐 / 分类 / 优先级判断 |
| 人工审批 | human-in-the-loop / confirmation gate |
| 系统写入 | Action / function-backed action / API writeback |
| 权限系统 | auth context / policy enforcement / scoped token |
| 操作日志 | tool call audit / reasoning trace / action lineage |
| BI 报表 | AI 分析、解释、模拟、建议 |

所以 Agent 不是凭空出现的新物种。它仍然依赖业务对象、规则、流程、权限、数据和动作。

但 Agent 和传统系统的关键差异在于：

`​``text
传统系统要求人适应软件流程。
Agent 系统让软件根据人的意图动态组织流程。
`​``

传统客服系统中，客服要知道去哪几个页面查订单、物流、售后、政策、工单。Agent 系统中，用户只说：

`​``text
为什么我这个订单不能退款？
`​``

Agent 自动完成：

`​``text
识别问题
找订单
查订单状态
查物流状态
查退款规则
查适用政策
判断是否可退
给出解释
建议下一步动作
必要时创建工单
`​``

这不是业务逻辑消失了，而是业务逻辑从“人通过 UI 逐步操作”变成“AI 通过工具和业务能力动态编排”。

---

**三、AI “代替人决策”这句话哪里对，哪里不对**

这句话有一半对，一半容易误导。

**对的部分：AI 确实可以替代一部分人类判断**

例如在 RAG 问答和智能客服里，过去很多判断由客服完成：

`​``text
用户到底想问什么？
是否需要追问？
该查哪个系统？
这条政策是否适用？
这个问题是否要转人工？
用户情绪是否异常？
这个工单属于哪个分类？
是否满足退款条件？
应该推荐维修、退款还是补偿？
`​``

这些判断里，有相当一部分可以由 AI 自动完成。

尤其是以下类型，AI 很适合做：

`​``text
意图识别
语义匹配
信息抽取
多源信息综合
相似案例检索
初步分类
优先级判断
风险提示
下一步动作建议
自然语言解释
`​``

例如：

`​``text
用户：“我买的手机屏幕坏了，才用了三个月，怎么处理？”
`​``

AI 可以判断：

`​``text
这是售后维修问题
可能涉及质保
需要订单信息
需要商品类目
需要判断是否人为损坏
需要查询质保政策
可能可以创建维修工单
`​``

这确实是在替代过去客服的一部分判断劳动。

---

**不对的部分：AI 不应该无边界替代最终责任决策**

“决策”不是一个单一动作。它至少包括五层：

`​``text
1. 识别情况
2. 汇总证据
3. 生成可选方案
4. 选择方案
5. 承担后果
`​``

AI 可以越来越多地承担前四层中的部分工作，但第五层，也就是**责任归属**，不能简单交给 AI。

例如：

`​``text
是否给用户赔偿 500 元？
是否拒绝保险理赔？
是否冻结账户？
是否标记用户为欺诈？
是否取消大客户订单？
是否调整生产计划？
是否改变供应链分配？
`​``

这些不是单纯技术问题，而是涉及：

`​``text
法律责任
商业风险
伦理约束
组织授权
客户关系
监管合规
品牌后果
`​``

所以更准确的表述应该是：

`​``text
AI 可以在受控范围内自动化“操作性决策”和“建议性决策”。
AI 不应无边界替代组织的“责任性决策”和“高风险裁决”。
`​``

---

**四、应把“决策”分成四类**

为了避免泛泛而谈，可以把企业里的决策分成四类。

**1. 确定性决策**

这类决策规则明确、数据明确、风险较低。

例如：

`​``text
订单是否超过 7 天无理由退货期
优惠券是否过期
用户是否有某项权益
物流是否已签收
发票是否已开具
`​``

这类不应该靠 AI “判断”，而应该由数据库、规则引擎、业务服务返回结果。

AI 的角色是解释：

`​``text
系统判断这笔订单已经超过退货期，因为签收时间是 2026-05-10，退货窗口截止到 2026-05-17。
`​``

这里 AI 不是真正决策者，业务系统才是。

---

**2. 概率性判断**

这类适合 AI 参与。

例如：

`​``text
用户真实意图是什么
这个投诉是否高风险
这条知识库是否相关
这个问题是否需要转人工
这个用户是否有流失风险
这段对话是否包含强烈不满情绪
`​``

这类判断过去依赖客服经验，现在 AI 可以很好地辅助甚至自动化。

但通常要保留：

`​``text
置信度
解释
兜底策略
人工复核
错误反馈机制
`​``

---

**3. 权衡型决策**

这类涉及多个目标之间的取舍。

例如：

`​``text
给不给补偿
优先处理哪个客户
库存不足时怎么分配
供应链中断时如何调整生产
客服资源不足时如何排队
`​``

AI 可以提出方案、模拟影响、排序建议，但未必应该直接拍板。

它需要结合：

`​``text
成本
SLA
客户价值
风险
公平性
政策约束
历史案例
当前运营目标
`​``

这正是 Palantir 一类平台强调的“决策操作系统”问题。

---

**4. 高风险责任决策**

这类必须谨慎。

例如：

`​``text
金融授信
医疗诊断
保险拒赔
执法行动
军事目标选择
大规模裁员
重大合同违约判断
`​``

AI 可以辅助，但通常不应独立完成最终决策。至少需要：

`​``text
人类审批
明确授权
审计链路
可解释证据
政策约束
可申诉机制
模拟与回滚
`​``

所以，“AI 代替人决策”应理解为：

`​``text
AI 替代的是大量低到中风险、重复性、信息密集型、判断密集型的决策劳动。
AI 不替代组织对目标、规则、权限、责任和高风险后果的承担。
`​``

---

**五、结合 Palantir：它说明的不是普通 RAG，而是“决策中心化”**

Palantir 的范式很值得拿来说明这个转变，因为它一直强调的不是“聊天机器人”，而是把 AI 接到真实业务运营上。

根据 Palantir 官方文档，AIP 的定位是把生成式 AI 连接到运营中；它强调 governed access、agent、evaluation、automation，以及 ontology-driven functions and agents。也就是说，它不是只让模型回答问题，而是让模型在治理框架内调用业务对象、逻辑和动作。参考 Palantir AIP capabilities 文档：  
https://www.palantir.com/docs/foundry/platform-overview/aip-capabilities

Palantir 对 Ontology 的定义也很关键。官方文档明确说，Ontology 不是简单表示数据，而是表示企业中的决策。它把相关的数据、逻辑和动作整合成 AI 可以访问的计算环境。参考 Palantir 平台概览：  
https://www.palantir.com/docs/foundry/platform-overview/overview

这和我们讨论的 MCP 设计高度一致。

普通 MCP 设计可能是：

`​``text
AI -> tool -> 某个系统 API
`​``

Palantir 式的设计更像是：

`​``text
AI -> Ontology tools/actions -> 业务对象 + 业务逻辑 + 权限 + 动作 + 审计
`​``

重点不是“接了多少系统”，而是这些系统被统一进一个业务语义层。

---

**六、Palantir Ontology 对 MCP 设计的启发**

Palantir 的 Ontology 可以理解为一个企业级业务语义层。它把企业中的东西抽象为：

`​``text
业务对象：订单、客户、设备、供应商、工厂、库存、合同、任务
业务关系：客户拥有订单，订单关联物流，设备属于产线，供应商影响产品
业务逻辑：规则、模型、预测、优化算法、分析模板
业务动作：调拨库存、调整计划、创建任务、写回 ERP、触发审批
权限治理：谁能看，谁能改，什么场景可用，如何审计
`​``

Palantir 官方文档里提到，Ontology 会把 operational process 的 nouns and verbs 建模成对人和 agent 都可读的形式；其中 Actions 定义企业中的“verbs”，也就是可执行动作，并控制人类操作者或 AI agent 如何让决策落地。参考 AIP 架构文档和平台概览：  
https://www.palantir.com/docs/foundry/architecture-center/aip-architecture  
https://www.palantir.com/docs/foundry/platform-overview/overview

这正好对应我们前面说的：

`​``text
MCP tool 不应该是数据库接口。
MCP tool 应该是业务动作和业务查询。
`​``

也就是：

`​``text
不是 query_table("orders")
而是 get_order_context(order_id, purpose)

不是 update_table("refunds")
而是 submit_refund_request(order_id, reason)

不是 execute_sql(sql)
而是 resolve_refund_question(customer_id, order_id, question)
`​``

---

**七、从“数据中心”到“决策中心”的范式变化**

传统企业软件大多是数据中心或流程中心。

`​``text
数据中心范式：
先建表
再做应用
再做报表
人看报表后决策
人去系统里执行
`​``

`​``text
流程中心范式：
先定义流程
再定义节点
人按流程操作
系统记录状态
异常由人处理
`​``

AI + Agent + MCP + Ontology 之后，范式变成：

`​``text
决策中心范式：
围绕业务决策建模
把相关数据、规则、模型、工具、动作、权限组织在一起
AI 理解当前情况
AI 调用工具获取证据
AI 生成方案或执行低风险动作
人类负责监督、授权和高风险裁决
系统记录决策、动作和结果
结果反过来改进下一次决策
`​``

这个变化的核心不是 UI 变成聊天框，而是企业系统的组织单位变了。

过去的组织单位是：

`​``text
表
页面
接口
流程
报表
`​``

现在更重要的组织单位变成：

`​``text
业务对象
业务能力
决策场景
可执行动作
证据链
权限上下文
反馈闭环
`​``

---

**八、Palantir 的关键不是“有 AI”，而是“AI 接上了运营语义”**

很多企业做 AI 客服或 RAG，会停留在：

`​``text
知识库文档
向量检索
问答生成
引用来源
`​``

这解决的是“回答问题”。

但 Palantir 代表的方向更进一步：

`​``text
AI 不只是回答问题，而是参与运营决策。
`​``

例如供应链中断场景，不只是问：

`​``text
哪个供应商出了问题？
`​``

而是继续做：

`​``text
哪些产品受影响？
哪些客户订单受影响？
有哪些替代供应商？
改生产计划会影响哪些产线？
库存如何调拨？
成本和交付风险如何变化？
是否需要通知客户？
是否要写回 ERP / WMS / 生产计划系统？
`​``

Palantir 的官方 Ontology 文档中有类似供应链例子：数据系统被综合成业务语义对象和关系，agent 可以在相同安全策略下查询、推荐和行动；方案可以先放在 scenario 中模拟和审查，再交给人类最终确认或写回操作系统。参考：  
https://palantirfoundation.org/docs/foundry/ontology/why-ontology

这说明的范式变化是：

`​``text
RAG 是“知识问答”。
Agent 是“任务编排”。
Ontology + Actions 是“决策执行环境”。
`​``

---

**九、AI 决策能力的真正边界：从 human-in-the-loop 到 human-on-the-loop**

过去常说 human-in-the-loop，也就是每一步都有人参与。

`​``text
AI 给建议
人确认
系统执行
`​``

但如果系统越来越成熟，会变成 human-on-the-loop：

`​``text
AI 自动处理常规情况
人监控整体运行
异常、高风险、低置信度情况交给人
`​``

再进一步，在低风险、高确定性的场景中，可以是 limited autonomy：

`​``text
AI 在授权边界内自动执行
系统记录审计
人事后抽检和优化策略
`​``

这也是 Palantir 文档中“from augmentation to automation”所暗示的方向：AI 注入业务流程可以被控制和评估，从辅助逐渐走向自动化。参考 AIP 架构文档：  
https://www.palantir.com/docs/foundry/architecture-center/aip-architecture

所以真正的变化不是一句“AI 替代人”，而是：

`​``text
人从一线操作员，逐步变成目标设定者、策略制定者、授权者、监督者和异常处理者。
AI 从问答工具，逐步变成受控业务执行者。
`​``

---

**十、MCP、Agent、Ontology 三者的关系**

可以这样理解：

`​``text
MCP 是连接协议。
Agent 是执行主体。
Ontology / 业务语义层是行动地图。
数据库和业务系统是事实与执行基础设施。
权限和审计是边界。
`​``

如果只有 MCP，没有业务语义层：

`​``text
AI 面对一堆工具
不知道业务结构
容易误用接口
权限复杂
效果不稳定
`​``

如果只有 Agent，没有受控工具：

`​``text
AI 只能聊天
不能可靠行动
无法进入生产业务
`​``

如果只有数据库，没有 Agent：

`​``text
数据在那里
人还要自己查、自己判断、自己执行
`​``

如果有 Ontology / Capability Layer：

`​``text
AI 看到的是业务世界：
客户、订单、合同、库存、设备、风险、政策、动作、约束、权限。
`​``

这时 Agent 才能真正参与业务。

所以理想结构是：

`​``text
用户意图
  |
Agent 理解和规划
  |
MCP 调用受控工具
  |
业务语义层 / Ontology / Capability Layer
  |
领域服务、规则、模型、动作
  |
数据库、搜索、向量库、外部系统
  |
审计、评估、反馈
`​``

---

**十一、这对企业软件建设意味着什么**

这个范式变化会改变企业系统设计方法。

**过去：先设计应用**

`​``text
我要做一个客服系统
我要做一个订单系统
我要做一个工单系统
我要做一个报表系统
`​``

**现在：先设计决策场景**

`​``text
用户为什么不能退款？
设备是否需要提前维修？
供应链中断如何重排产能？
这个客户是否应该优先处理？
这笔交易是否有风险？
这个项目是否会延期？
`​``

然后围绕每个决策场景组织：

`​``text
需要哪些数据？
需要哪些规则？
需要哪些模型？
需要哪些动作？
谁有权限看？
谁有权限执行？
什么时候需要人工确认？
如何审计？
如何评估效果？
结果如何反馈？
`​``

这就是从“应用软件”转向“决策软件”。

---

**十二、对“Agent 只是换一种方法实现业务”的进一步澄清**

这句话在工程层面是对的。

因为最后还是要落到：

`​``text
查数据
算规则
调接口
改状态
写日志
走审批
发通知
生成报告
`​``

但它在产品范式上又不完全对。

传统业务系统里，流程是开发者提前写死的：

`​``text
用户必须按照系统设计的路径走。
`​``

Agent 系统里，流程可以根据当前上下文动态生成：

`​``text
系统根据用户意图、业务状态、权限、风险和工具可用性，临时组织一条解决路径。
`​``

这就是差异。

举例：

`​``text
用户：我上个月账单不对，而且客服一直没处理。
`​``

传统系统可能需要客服手动进入：

`​``text
账单系统
订单系统
支付系统
优惠券系统
工单系统
投诉系统
客户画像
知识库
`​``

Agent 可以动态规划：

`​``text
1. 查用户身份和账单摘要
2. 识别异常扣费项
3. 查相关订单和优惠券
4. 查历史工单
5. 判断是否超 SLA
6. 检索账单政策
7. 给出解释
8. 如果确实异常，建议退款或补偿
9. 必要时升级人工
`​``

这仍然是在实现业务，但实现方式从“人驱动软件”变成“AI 驱动业务能力”。

---

**十三、最终结论**

可以把你的论断整理成一个更严谨的版本：

`​``text
MCP 不应该面向底层系统，而应该面向业务能力设计。
Agent 不是脱离传统业务系统的新东西，而是用自然语言、推理和工具调用重新组织业务流程。
AI 的独特能力不是无条件替代人类决策，而是在业务语义、权限、规则和审计约束下，自动化一部分原本由人完成的理解、判断、权衡和编排。
Palantir 所代表的范式，是从数据中心、应用中心、流程中心，转向以 Ontology 为核心的决策中心：把数据、逻辑、动作、权限和反馈统一建模，让 AI agent 能在真实运营中安全地建议、模拟、执行和学习。
`​``

一句话概括：

`​``text
过去的软件帮助人做决策。
现在的 AI agent 开始参与决策。
未来的企业平台要管理“人和 AI 如何共同决策并把决策写回现实世界”。
`​``

而 Palantir 的启示是：真正重要的不是把 LLM 接到数据库，也不是把所有 API 包成 MCP tool，而是建立一个企业级业务语义层，把“对象、规则、动作、权限、审计、反馈”统一起来。只有这样，AI 的决策能力才不会停留在聊天和建议，而能进入可治理、可执行、可追责的业务运营。

<!-- /message -->
<!-- message:id=msg_7f9a1342d57246fd role=user createdAt=2026-05-21T00%3A31%3A00.020Z -->

## User

举一个实际的例子，用于说明palantir如何定义语义层及其他关键概念和能力，实现受控决策

<!-- /message -->
<!-- message:id=msg_e345234d1e8355d7 role=assistant createdAt=2026-05-21T00%3A33%3A02.031Z runId=run_16ec24033bb6ccc5 status=succeeded -->

## Assistant

下面用一个“制造企业供应链断供处置”的例子说明。这个例子和 Palantir 官方文档里的 Onyx 医疗设备制造商场景一致：某供应商突然无法供应用于生产外科口罩的关键原材料，企业需要判断哪些订单受影响、是否调整产能、是否替换材料、是否改物流路线，以及哪些动作可以自动执行、哪些必须人工审批。Palantir 文档把 Ontology 描述为面向企业决策的模型，核心由 `Data / Logic / Action / Security` 组成，并强调它不是单纯的数据模型，而是把人和 AI 的决策接到真实运营上。参考：[Why create an Ontology?](https://palantirfoundation.org/docs/foundry/ontology/why-ontology)

**1. 先定义语义层：不是表，而是业务世界**

传统系统里可能有这些底层系统：

`​``text
ERP：采购订单、供应商、物料主数据
MES：生产计划、工单、产线状态
WMS：库存、库位、批次、可用量
TMS：运输计划、承运商、交付时间
CRM：客户订单、客户等级、合同 SLA
QMS：质量记录、材料合规认证
数据仓库：历史需求、成本、收入、毛利
`​``

如果直接把这些系统暴露给 AI，它看到的是一堆表、字段和 API。Palantir 的做法是先在 Ontology 中定义业务对象，也就是企业里的“名词”：

`​``text
Supplier：供应商
Material：原材料
InventoryLot：库存批次
Plant：工厂
ProductionLine：产线
WorkOrder：生产工单
Product：成品
CustomerOrder：客户订单
Shipment：发运任务
Contract：合同
RiskEvent：风险事件
Scenario：模拟方案
Decision：决策记录
`​``

然后定义对象之间的业务关系：

`​``text
Supplier supplies Material
Material is used in Product BOM
InventoryLot stores Material
Plant has ProductionLine
ProductionLine executes WorkOrder
WorkOrder produces Product
CustomerOrder requires Product
Shipment fulfills CustomerOrder
Contract defines SLA for CustomerOrder
RiskEvent affects Supplier / Material / Plant
`​``

这样 AI 和业务人员看到的不是 `purchase_order_line.qty_remaining`、`material_master.mtrl_cd`、`prod_sched.stat` 这种字段，而是：

`​``text
供应商 A 供应材料 M。
材料 M 被产品 P 使用。
产品 P 关联 183 个客户订单。
其中 27 个订单属于高优先级客户。
如果不调整计划，预计 11 个订单会违反 SLA。
`​``

这就是语义层的第一层价值：**把底层数据变成企业能理解、AI 也能安全操作的业务对象和关系**。Palantir 文档中也强调 Ontology 会把运营过程中的 “nouns and verbs” 建模成对人和 agent 都可读的形式，参考：[AIP architecture overview](https://palantirfoundation.org/docs/foundry/architecture-center/aip-architecture)。

**2. 把数据接入 Ontology，但保留来源、权限和血缘**

每个业务对象背后仍然来自真实系统：

`​``text
Supplier 来自 ERP 供应商主数据。
Material 来自 ERP 物料主数据和 BOM。
InventoryLot 来自 WMS。
WorkOrder 来自 MES。
CustomerOrder 来自 CRM / OMS。
Shipment 来自 TMS。
质量认证来自 QMS。
历史需求来自数据仓库。
`​``

Ontology 不是把所有数据复制成一个大表，而是把多个系统的数据映射成统一业务对象。

例如 `Material` 对象可以有这些属性：

`​``json
{
  "material_id": "MAT-8831",
  "name": "Meltblown Fabric Grade A",
  "approved_suppliers": ["SUP-A", "SUP-B"],
  "current_available_qty": 42000,
  "daily_consumption": 8000,
  "regulatory_category": "medical_grade",
  "substitution_allowed": true,
  "restricted_fields": ["unit_cost", "supplier_contract_terms"]
}
`​``

关键是，属性不是全部对所有人可见。生产主管可以看库存和产能，采购经理可以看供应商和合同，销售团队可能只能看订单风险和预计交付时间，AI Agent 也只能在被授予的范围内读取这些对象。

这对应 Palantir 强调的安全模型：权限不只是角色，还包括数据标记、用途、行列级限制、SSO 组、运行时策略和工具调用控制。参考：[Why create an Ontology? - Security](https://palantirfoundation.org/docs/foundry/ontology/why-ontology)。

**3. 定义 Logic：把规则、模型、优化器接进语义层**

只有对象还不够。断供处置不是“查资料”，而是要算影响、做权衡、生成方案。

所以 Ontology 里还要绑定逻辑资产，例如：

`​``text
calculate_material_shortage(material_id, horizon_days)
计算未来 N 天材料缺口。

find_at_risk_orders(material_id)
找出受该材料影响的客户订单。

forecast_demand(product_id, horizon_days)
预测未来需求。

simulate_reallocation(material_id, allocation_plan)
模拟材料重新分配后的订单影响。

optimize_production_schedule(constraints)
在产能、库存、SLA、成本之间求一个生产计划。

check_material_substitution(material_id, substitute_id)
判断替代材料是否满足质量、法规和客户合同要求。

calculate_revenue_at_risk(order_set)
计算受影响收入、毛利和违约风险。
`​``

这些逻辑可以来自不同地方：

`​``text
SQL / Spark 计算
Python 模型
优化算法
规则引擎
机器学习模型
外部 SaaS API
历史案例检索
LLM 分析工具
`​``

Palantir 文档把这一点称为把异构逻辑资产接入 Ontology，使人和 AI agent 都能在同一业务上下文中使用这些逻辑。参考：[Why create an Ontology? - Logic](https://palantirfoundation.org/docs/foundry/ontology/why-ontology)。

这里的重点是：**AI 不自己临时发明业务规则**。AI 可以调用 `optimize_production_schedule`，但约束、目标函数、SLA、法规校验、库存计算仍然由确定性逻辑和业务系统控制。

**4. 定义 Action：把“建议”变成受控动作**

接下来要定义业务动作，也就是企业里的“动词”。

例如可以定义这些 Action Types：

`​``text
CreateExpeditePurchaseOrder
创建紧急采购单。

ReallocateInventory
把某批库存从一个工厂调拨到另一个工厂。

ChangeProductionSchedule
调整某条产线的生产计划。

ApproveMaterialSubstitution
批准某个替代材料用于特定产品。

CreateCustomerRiskTask
给客户经理创建风险沟通任务。

UpdateShipmentPriority
调整发运优先级。

NotifyCustomer
向客户发送交付风险通知。

CommitScenarioToERP
把已审批方案写回 ERP / MES / WMS / TMS。
`​``

每个 Action 都不是裸 API，而是带参数、校验、权限、审批和副作用的业务动作。

例如 `ChangeProductionSchedule` 可以定义为：

`​``json
{
  "action_type": "ChangeProductionSchedule",
  "input": {
    "plant_id": "PLANT-02",
    "production_line_id": "LINE-7",
    "work_order_id": "WO-10029",
    "new_start_time": "2026-05-22T08:00:00Z",
    "new_quantity": 120000,
    "reason_code": "SUPPLIER_DISRUPTION"
  },
  "validations": [
    "line_must_be_available",
    "material_must_be_allocated",
    "quality_release_must_exist",
    "no_contractual_priority_violation"
  ],
  "approval_rules": [
    "if revenue_impact > 500000 then require_supply_chain_director",
    "if medical_grade_substitution then require_quality_approval"
  ],
  "side_effects": [
    "writeback_to_MES",
    "notify_warehouse",
    "update_customer_order_risk",
    "create_audit_event"
  ]
}
`​``

Palantir 的 Action Type 文档说明，Action 是对对象、属性和链接进行变更的事务性定义，并可以包含校验、规则和副作用；同一套 Action 逻辑可以在多个应用中复用。参考：[Action types overview](https://www.palantir.com/docs/foundry/action-types/overview)。

这就是受控决策的关键：**AI 可以建议或触发动作，但动作本身由 Ontology Action 定义，受权限、参数、规则、审批和审计控制。**

**5. 用 Scenario 做模拟：先沙箱推演，再写回现实**

断供场景下，AI Agent 可能提出一个方案：

`​``text
把材料 M 的现有库存优先分配给高 SLA 客户订单。
将普通订单延后 2 天。
从供应商 B 紧急采购替代材料。
把工厂 2 的产线 7 从产品 X 切换到外科口罩。
对 12 个可能延期客户生成客户经理跟进任务。
`​``

这个方案不能马上写回 ERP。正确做法是先进入 Scenario：

`​``text
Scenario A：按收入优先分配
Scenario B：按 SLA 风险优先分配
Scenario C：按客户等级优先分配
Scenario D：采购替代材料并调整生产计划
`​``

每个 Scenario 都计算后果：

`​``text
受影响订单数
违约订单数
预计收入风险
额外采购成本
加急运输成本
产线切换损失
库存耗尽时间
客户 SLA 风险
法规或质量风险
`​``

Palantir 文档里的 Onyx 例子明确提到，模拟输出可以作为 Ontology scenarios 暂存，团队可以在提交前安全分析方案影响。参考：[Why create an Ontology? - Building simulations and designing solutions](https://palantirfoundation.org/docs/foundry/ontology/why-ontology)。

这一步体现了范式变化：

`​``text
普通 RAG：回答“哪些订单受影响？”
Agent：建议“可以怎么处理？”
Ontology + Scenario：把建议变成可计算、可比较、可审批、可回滚的业务方案。
`​``

**6. AI Agent 的角色：不是自由决策者，而是受控参谋和执行编排者**

业务人员可以问：

`​``text
供应商 A 断供 10 天，优先保障高价值客户订单。给我一个不违反质量规则的生产和调拨方案。
`​``

AI Agent 的执行路径可以是：

`​``text
1. 识别这是供应链风险处置任务。
2. 查询 RiskEvent、Supplier、Material、InventoryLot、WorkOrder、CustomerOrder。
3. 调用 find_at_risk_orders 找出受影响订单。
4. 调用 calculate_revenue_at_risk 计算风险。
5. 调用 check_material_substitution 检查替代材料。
6. 调用 optimize_production_schedule 生成候选计划。
7. 创建 Scenario A / B / C。
8. 对比每个 Scenario 的 SLA、成本、收入和质量风险。
9. 推荐一个方案。
10. 如果涉及高影响动作，交给人审批。
11. 审批后调用 Action 写回 ERP / MES / WMS / TMS。
12. 记录决策链、工具调用、审批人和写回结果。
`​``

Palantir AIP 文档强调 AIP 是把 AI 接到数据和运营中，并支持生产级工作流、agents、functions、评估、审计和资源治理。参考：[AIP overview](https://www.palantir.com/docs/foundry/aip/overview)。

所以 AI Agent 并不是“想查什么就查什么、想改什么就改什么”。它面对的是 Ontology 暴露出来的受控对象、函数和动作。

**7. 权限如何控制：对象权限 + 工具权限 + 动作提交条件**

在这个例子里，可以这样设权限：

`​``text
生产主管：
可查看产线、工单、库存数量。
可提交 ChangeProductionSchedule，但超过 24 小时延期需上级审批。

采购经理：
可查看供应商、采购价格、合同条款。
可提交 CreateExpeditePurchaseOrder。

质量负责人：
可查看材料认证和质量记录。
可审批 ApproveMaterialSubstitution。

销售团队：
可查看客户订单风险和预计交付时间。
不能查看供应商成本和内部毛利。

AI Agent：
可读取供应链风险对象、库存摘要、订单风险摘要。
可运行模拟。
可创建建议方案。
不能直接提交高影响写回动作。
只能在人工审批后调用 CommitScenarioToERP。
`​``

这意味着，AI Agent 的能力不是由提示词约束，而是由平台权限约束：

`​``text
能看哪些对象？
能看哪些属性？
能调用哪些函数？
能触发哪些 Action？
哪些 Action 需要审批？
哪些 Action 只能在 Scenario 中模拟？
哪些结果必须脱敏？
哪些工具调用必须记录高等级审计？
`​``

Palantir 文档提到，agent 的工具使用也受同一安全架构控制，且工具调用依赖对底层对象、属性和链接的访问权限。参考：[Why create an Ontology? - Security](https://palantirfoundation.org/docs/foundry/ontology/why-ontology)。

**8. 审计和评估：决策不是一次性回答，而是可追踪链路**

一次受控决策完成后，系统要留下完整记录：

`​``text
谁提出了问题？
AI Agent 调用了哪些对象和函数？
使用了哪个模型版本？
读取了哪些数据版本？
生成了哪些 Scenario？
每个 Scenario 的计算结果是什么？
AI 推荐了哪个方案？
谁审批了方案？
审批时看到了哪些证据？
哪些 Action 被执行？
写回了哪些系统？
结果是否改善了 SLA / 成本 / 客户风险？
后续是否需要复盘？
`​``

Palantir AIP 架构文档强调端到端 observability，包括 AI workflow、agentic process、action logging、链式执行追踪和资源使用监控；AIP Evals 用于测试、调试和比较 agent 表现。参考：[AIP architecture overview](https://palantirfoundation.org/docs/foundry/architecture-center/aip-architecture)。

这就是和普通“AI 问答”的区别：

`​``text
普通问答只留下答案。
受控决策留下数据、逻辑、方案、动作、审批、写回和结果。
`​``

**9. 把它映射回 MCP / Agent 架构**

如果不用 Palantir，而是自己建设类似架构，那么对应关系大致是：

`​``text
Palantir Ontology
≈ 企业业务语义层 / Capability Layer

Object Type
≈ 业务对象，例如订单、库存、供应商、设备、客户

Link Type
≈ 业务关系，例如订单依赖库存、供应商供应物料

Function / Logic
≈ 规则引擎、优化器、预测模型、SQL 模板、服务函数

Action Type
≈ 受控业务动作，例如创建工单、调整计划、提交退款、写回 ERP

Scenario
≈ 沙箱模拟、what-if 分支、待审批方案

AIP Agent
≈ 在语义层上工作的业务 Agent

Security / Governance
≈ IAM、策略引擎、行列权限、工具权限、审批、审计

MCP
≈ Agent 调用这些对象、函数和动作的协议层
`​``

所以 MCP 不应该直接暴露：

`​``text
query_erp_table
update_mes_table
select_customer_orders
execute_sql
`​``

而应该暴露 Ontology / Capability 层中的能力：

`​``text
find_at_risk_orders
simulate_reallocation
compare_scenarios
submit_schedule_change_for_approval
commit_approved_plan
`​``

**10. 这个例子说明的核心范式**

Palantir 的关键不是“给 LLM 接数据库”，而是把企业运营抽象成一个可治理的决策环境：

`​``text
Data：哪些事实和状态参与决策？
Logic：如何计算、预测、优化、校验？
Action：决策如何变成真实系统里的动作？
Security：谁能看、谁能算、谁能建议、谁能执行、谁负责？
`​``

在供应链断供例子里，AI 不是裸奔式地替人拍板，而是在 Ontology 中：

`​``text
基于授权数据理解现状；
调用确定性逻辑和模型计算影响；
在 Scenario 中生成和比较方案；
按权限触发 Action；
高风险动作交给人审批；
审批后写回业务系统；
全程记录审计和结果反馈。
`​``

一句话概括：

`​``text
Palantir 的语义层把“企业有什么、它们如何关联、可以做什么、谁能做、做了以后写回哪里”统一建模；AI Agent 不是绕过业务系统做决策，而是在这个语义层和治理框架内参与决策。
`​``

这就是从“AI 查数据并回答”转向“AI 在受控业务语义层中建议、模拟、审批、执行和复盘”的范式变化。

<!-- /message -->
