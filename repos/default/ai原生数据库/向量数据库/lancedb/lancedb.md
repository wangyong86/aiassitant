---
id: "ses_0fb782ba91237fca"
title: "lancedb"
createdAt: "2026-05-26T23:55:47.739Z"
updatedAt: "2026-06-01T11:19:52.945Z"
---

# lancedb

<!-- message:id=msg_a49c281ecc74f349 role=user createdAt=2026-05-26T23%3A57%3A03.673Z -->

## User

lancedb的产品形态，开源部分，解决的问题场景和典型应用。

<!-- /message -->
<!-- message:id=msg_93f532fb34d3a8fa role=assistant createdAt=2026-05-26T23%3A58%3A42.287Z runId=run_c29f65041d523cd0 status=succeeded -->

## Assistant

我按当前官方文档理解，LanceDB 现在不只是“一个向量数据库”，更准确的定位是：**面向 AI/ML 的多模态 Lakehouse + 向量检索层**。它把向量、原始多模态数据、元数据、索引、版本管理放在同一套开放表格式之上，而不是只做一个外置 embedding store。

资料口径主要来自 LanceDB 官方文档和 GitHub：LanceDB 文档说明它由 **LanceDB OSS** 和 **LanceDB Enterprise** 组成，二者都基于开源 **Lance format**；OSS 是嵌入式开源库，Enterprise 是分布式/托管的多模态 lakehouse。官方也把核心场景列为 search/retrieval、data curation、feature engineering、training 等。参考：LanceDB docs、Enterprise docs、Lance format docs、LanceDB GitHub。  
来源：<https://docs.lancedb.com/>、<https://docs.lancedb.com/enterprise>、<https://docs.lancedb.com/lance>、<https://github.com/lancedb/lancedb>

**1. 产品形态**

| 形态 | 定位 | 典型使用方式 | 适合阶段 |
|---|---|---|---|
| **Lance format** | 开源多模态 lakehouse 表格式，是 LanceDB 的底层存储基础 | 存储 vectors、images、videos、audio、text、metadata、indexes 等 | 数据底座、AI 数据湖、训练数据管理 |
| **LanceDB OSS** | 开源嵌入式向量/检索库，Apache 2.0 | 像 SQLite 一样嵌入 Python/TypeScript/Rust 应用进程，数据可放本地盘或对象存储 | 原型、本地开发、轻量生产、私有部署 |
| **LanceDB Enterprise / Managed / BYOC** | 商业化分布式版本，面向生产规模 | 远程服务、托管部署或 BYOC 部署到客户云账号/VPC | 大规模生产、高并发、合规、私有网络、自动运维 |
| **SDK / API 形态** | Python、TypeScript、Rust 等 SDK，Enterprise/Cloud 还可通过远程表/API 访问 | 应用直接 `connect()`，创建表、写入向量、检索、过滤、建索引 | 应用集成层 |

官方当前文档中强调的商业部署方式主要是 **Managed** 和 **BYOC**：Managed 由 LanceDB 管理基础设施；BYOC 安装到客户自己的云账号中，适合数据不能离开自己账号/VPC 的场景。来源：<https://docs.lancedb.com/enterprise/deployment>

**2. 开源部分包括什么**

LanceDB 的开源部分可以分两层看。

第一层是 **Lance format**。这是开源 lakehouse 格式，包含文件格式、表格式、catalog spec，面向多模态 AI 数据设计。它支持列式存储、Arrow 生态、版本管理、schema evolution、随机访问、向量索引、全文检索等能力。官方描述它适合构建搜索引擎、特征库、大规模 ML 训练、高性能 IO 和多模态数据管理。来源：<https://docs.lancedb.com/lance>、<https://github.com/lance-format/lance>

第二层是 **LanceDB OSS**。这是开源嵌入式 retrieval library，官方 FAQ 明确说 LanceDB OSS 是 Apache 2.0 许可的嵌入式检索库。它不要求你单独启动一个数据库服务，可以直接在应用进程、notebook、Docker 或 serverless function 里跑。来源：<https://docs.lancedb.com/faq/faq-oss>、<https://www.lancedb.com/lp/embedded-oss>

开源 OSS 侧核心能力大致包括：

| 能力 | 说明 |
|---|---|
| 向量检索 | 支持 similarity search，适合 embedding top-k 检索 |
| 全文检索 | 支持 FTS/BM25，能和向量检索组合做 hybrid search |
| SQL/过滤 | 可以对 metadata 做过滤、SQL 风格查询 |
| 多模态存储 | vectors、text、image、audio、video、PDF bytes、metadata 可放在同一表中 |
| 索引 | 支持 IVF、HNSW 组合、PQ/SQ/RQ 等索引/量化路线；OSS 中通常需要手动创建和维护索引 |
| 版本管理 | Lance 表有版本化能力，适合实验复现、数据 lineage、增量更新 |
| 云存储支持 | OSS 可配置本地文件系统、S3、GCS、Azure Blob、S3-compatible object store 等 |
| SDK | Python、TypeScript、Rust 是官方文档明确提到的主要 SDK |

需要注意：**自动索引、分布式查询、分布式缓存、企业级监控、私有网络/合规、托管运维** 这些能力主要在 Enterprise 侧。官方文档明确提到，OSS 的向量索引需要手动创建和维护，而 Enterprise 会自动管理索引；Enterprise 还提供低延迟分层缓存、横向扩展、监控、BYOC/Managed 部署等能力。来源：<https://docs.lancedb.com/indexing/vector-index>、<https://docs.lancedb.com/enterprise>

**3. 它解决的核心问题**

LanceDB 解决的不是单一“向量查找”问题，而是 AI 应用里经常出现的几类数据工程问题。

**问题一：embedding、原始数据、元数据分散在多套系统里。**  
传统 RAG 或多模态检索系统经常是：原文在对象存储，metadata 在 Postgres，向量在向量数据库，全文检索在 Elasticsearch/OpenSearch。这样会带来 ETL、同步、权限、版本一致性、回溯复现等成本。LanceDB 的思路是把 source data、metadata、embeddings 放在同一张 Lance 表里，再在这张表上做 vector search、full-text search、SQL filter 和 hybrid search。

**问题二：多模态数据不适合只用普通行列数据库或纯向量库管理。**  
图片、视频、音频、PDF、点云等数据往往很大，训练/检索时又需要和 embedding、标签、特征、版本绑定。Lance format 针对多模态数据、随机访问和列式读取做了优化，适合 AI 数据集管理。官方文档也提到它可以原生处理 images、audio、video、PDF bytes 等 blob 数据。来源：<https://docs.lancedb.com/tables/multimodal>

**问题三：训练和数据治理需要高效随机访问、版本化和增量特征工程。**  
ML 训练并不只是全表扫描，常见操作包括采样、shuffle、按标签过滤、增量加特征列、给历史数据补 embedding、对比不同数据版本。Lance 的版本化、schema evolution、zero-copy 数据演进，能减少反复重写大文件和重复搬运数据。官方把它列为 training、feature engineering、data curation 的核心场景。来源：<https://docs.lancedb.com/>、<https://docs.lancedb.com/lance>

**问题四：向量数据库从原型到生产的形态断层。**  
很多向量库在本地开发时要起服务、容器、集群；也有一些库适合本地但上生产困难。LanceDB OSS 的优势是嵌入式，启动成本低；当数据量、QPS、合规和运维要求上来时，可以迁移到 Enterprise/BYOC，底层格式和表抽象保持一致。

**问题五：对象存储上的 AI 数据检索。**  
很多企业 AI 数据已经在 S3/GCS/Azure Blob 上。LanceDB 是 disk-first / object-store-friendly 的设计，可以把对象存储作为持久层，再通过本地/远程计算层做检索、索引和缓存。Enterprise 进一步做计算存储分离、缓存、后台索引和 compaction。来源：<https://docs.lancedb.com/storage>、<https://docs.lancedb.com/enterprise/architecture>

**4. 典型应用场景**

**RAG / 企业知识库**  
把文档 chunk、embedding、原文、标题、权限、部门、时间、来源 URL 等放在 LanceDB 表里。查询时先向量召回，再结合 metadata filter、全文检索和 rerank。适合内部知识库、客服问答、法律/财务/研发文档检索。

**AI Agent 长期记忆**  
Agent 的交互历史、工具调用结果、用户偏好、任务上下文都可以 embedding 化后写入 LanceDB。之后按语义召回相关记忆，同时用 metadata 控制用户、项目、时间范围和权限。

**多模态搜索**  
比如以图搜图、文本搜图、视频片段检索、音频内容检索、商品图片检索。LanceDB 的特点是可以把图片/视频/PDF 原始 bytes 或路径、embedding、标签、结构化属性放在一起，减少多系统拼接。

**推荐系统 / 相似内容召回**  
商品、文章、短视频、音乐、广告素材都可以用 embedding 做相似召回，再按类目、价格、地区、库存、用户分群等 metadata 过滤。LanceDB 适合做候选集召回层，不一定替代完整的 ranking/recommender pipeline。

**数据集治理和训练数据管理**  
用于管理大规模图像、视频、点云、文本语料、embedding 数据集。常见操作包括筛选、去重、采样、打标签、添加新特征列、追踪版本、准备 fine-tuning 数据。它更接近“AI 数据湖表 + 检索引擎”，不只是线上 serving 的向量库。

**特征工程 / Embedding 管线**  
当模型升级或 embedding 模型更换时，可以在原表上新增 embedding 列，而不是重写所有原始数据。这样一张表里可以存在多个模型版本的向量列，便于 A/B、回溯和迁移。

**本地 AI 应用 / 桌面应用 / Notebook 分析**  
OSS 嵌入式形态很适合个人知识库、代码语义搜索、本地 RAG、研究 notebook、小型 SaaS 初期版本。像 SQLite 一样直接嵌入应用，不需要单独维护 Milvus、Qdrant、Elasticsearch 这类服务。

**5. 和传统向量数据库的差异**

| 维度 | LanceDB 倾向 | 传统向量 DB 常见形态 |
|---|---|---|
| 核心定位 | 多模态 lakehouse + retrieval | 在线向量检索服务 |
| 存储模型 | 基于 Lance 开放表格式，disk/object-store first | 服务内部存储或外部存储抽象 |
| 部署 | OSS 嵌入式；Enterprise 托管/BYOC | 单机服务、集群、托管服务 |
| 数据对象 | 向量、metadata、原始多模态数据、索引、版本 | 主要是向量 + metadata |
| AI 训练数据 | 强调随机访问、版本、schema evolution、feature engineering | 通常不是核心重点 |
| 上手成本 | OSS 很低，不需要单独服务 | 通常要起服务/容器/集群 |
| 大规模生产 | 依赖 Enterprise 做分布式、缓存、运维 | 多数产品原生就是服务/集群形态 |

**6. 适合与不适合**

适合 LanceDB 的情况：

| 场景 | 原因 |
|---|---|
| RAG、agent memory、语义搜索 | 向量 + 全文 + metadata filter 都在一套表上 |
| 多模态数据检索 | 原始数据、embedding、标签、索引可以统一管理 |
| AI 数据集管理 | 支持版本、随机访问、schema evolution |
| 本地/嵌入式开发 | OSS 不需要单独数据库服务 |
| 对象存储上的 AI 数据湖 | Lance format 天然面向 lakehouse/object storage |
| 希望减少 ETL 和数据复制 | 不必把数据拆到多个系统维护 |

不一定最适合的情况：

| 场景 | 可能更适合 |
|---|---|
| 纯关系型 OLTP、高频事务更新 | PostgreSQL/MySQL 等 |
| 只是小规模 Postgres 应用加一点向量 | pgvector 可能更简单 |
| 强依赖复杂 keyword search、日志检索、聚合分析 | Elasticsearch/OpenSearch/ClickHouse 等 |
| 需要开箱即用的全托管大规模服务但不想采购 Enterprise | 可比较 Pinecone、Weaviate Cloud、Qdrant Cloud、Milvus/Zilliz 等 |
| 数据主要是结构化 BI 分析 | DuckDB、ClickHouse、Snowflake、BigQuery 等更直接 |

**7. 一句话总结**

LanceDB 的差异点在于：**它不是单纯把 embedding 存起来做 ANN 检索，而是把向量检索、多模态原始数据、metadata、全文检索、版本化和 AI 数据工程放到同一个开放 lakehouse 表格式上。** OSS 适合嵌入式、本地、轻量生产和私有部署；Enterprise/Managed/BYOC 负责生产规模下的分布式查询、自动索引、缓存、监控、安全和合规。

如果用产品视角概括：

| 层次 | 产品价值 |
|---|---|
| Lance format | 开放、多模态、面向 AI 的数据底座 |
| LanceDB OSS | 开发者友好的嵌入式向量/混合检索库 |
| LanceDB Enterprise | 面向企业生产的分布式多模态 lakehouse / retrieval platform |
| 典型应用 | RAG、Agent memory、多模态搜索、推荐召回、AI 数据集管理、训练数据访问、特征工程 |

<!-- /message -->
<!-- message:id=msg_e31ed54ee7e61845 role=user createdAt=2026-06-01T11%3A16%3A43.388Z -->

## User

lancedb的创始团队，发展过程，营收变化，他们说如何找到这个产品形态的，对当前的ai大方向，找具体赛道的逻辑

<!-- /message -->
<!-- message:id=msg_dcedcfa9bcef18d9 role=assistant createdAt=2026-06-01T11%3A19%3A52.945Z runId=run_68afbd2dd7ccf810 status=succeeded -->

## Assistant

下面按“可验证事实”和“我的归纳判断”分开说。**截至 2026-06-01 的公开资料里，LanceDB 没有披露正式财报或连续 ARR 数据**，所以营收部分只能用第三方估算和融资/客户侧信号辅助判断，不能当作审计口径。

**1. 创始团队**

LanceDB 的核心创始人是 **Chang She** 和 **Lei Xu**。

**Chang She：CEO / Co-founder**  
他的背景非常典型地指向“数据基础设施 + ML 数据”的交叉点：

- 早期是 **pandas 核心贡献者之一**，也有早期 Parquet 相关经验。
- 曾任 **Cloudera** 工程管理岗位，做 Hadoop / 数据基础设施相关工作。
- 后来在 **Tubi TV** 做 VP Engineering，负责过 ML stack、实验平台、推荐/个性化等系统。
- TechCrunch 也提到他是 pandas 共同创建/早期核心贡献者之一，并在 Tubi、Cloudera 做过数据工具和基础设施。来源：TechCrunch 对 LanceDB 的报道，L130-L134。  
  <https://techcrunch.com/2024/05/15/lancedb-which-counts-midjourney-as-a-customer-is-building-databases-for-multimodal-ai/>

**Lei Xu：CTO / Co-founder**  
Lei Xu 的背景更偏底层存储、分布式系统、Hadoop/HDFS 和自动驾驶数据基础设施：

- PhD，研究方向偏分布式存储。
- Apache Hadoop HDFS PMC / committer。
- 曾在 **Cloudera** 做 HDFS / Ozone 等相关工作。
- 曾在 **Cruise** 做机器学习平台、数据基础设施，接触自动驾驶的大规模多模态数据。
- Wayfinder 的公司页明确列出 Lei Xu 是 LanceDB CTO/co-founder，并提到他在 Cruise 做 ML Platform，在 Cloudera 做 HDFS，是 Hadoop HDFS PMC 成员。来源：Wayfinder，L55-L60。  
  <https://www.wayfinder.com/company/lancedb>

这两个人的组合很重要：  
**Chang She 代表数据科学/数据工程/ML 产品化经验，Lei Xu 代表分布式存储和大规模多模态数据系统经验。** 这不是一个“看到向量数据库热就冲进去”的团队，而是从多年数据系统经验里抽象出新 AI workload 的基础设施缺口。

**2. 发展过程**

可以粗略分成四个阶段。

**阶段一：从多模态数据格式问题开始，不是从“向量数据库”开始**

公开资料显示，LanceDB 的根不是“我要做一个 Pinecone/Qdrant 竞品”，而是两位创始人先看到一个更底层的问题：  
**AI 数据不再只是结构化表格，而是 embedding、图片、视频、音频、PDF、字幕、metadata、训练样本、索引混在一起。传统 Parquet、Spark、pandas、Arrow、WebDataset 等组合不够顺。**

Chang She 在 O’Reilly 访谈里说，他在 Tubi 做视频点播/流媒体时，处理个性化 embedding、视频资产、图片资产、音频、字幕文本等“AI data”，这些东西“不太适合传统数据栈”，这是他开始 LanceDB 的灵感来源。来源：O’Reilly 访谈，L392-L398。  
<https://www.oreilly.com/radar/podcast/generative-ai-in-the-real-world-chang-she-on-data-infrastructure-for-ai/>

官方 Series A 公告里，Chang She 也回顾说，四年前他和 Lei 问的问题是：**为什么处理 embeddings、images、video 仍然比表格数据难这么多？** 他们的判断是行业建立在“上一代数据”的基础设施上，所以要从 first principles 重新做，放弃把 Parquet / WebDataset 当作核心基础。来源：LanceDB 官方 Series A 公告，L455-L457。  
<https://www.lancedb.com/blog/series-a-funding>

所以最早的核心是 **Lance format**：一个面向 AI/multimodal 数据的开放列式格式和表格式。

**阶段二：用开源 Lance format + LanceDB OSS 打开发者入口**

LanceDB 之后把 Lance format 上面的检索/数据库能力包装成 LanceDB OSS，形成一个很强的开发者入口：

- 本地嵌入式，不需要先部署一个独立数据库服务。
- Python / TypeScript / Rust SDK。
- 支持 vector search、full-text search、hybrid search、metadata filter。
- 像 SQLite / DuckDB 一样适合 notebook、本地应用、serverless、早期 RAG 项目。
- 同时底层数据格式可以连接到 S3 / object storage，为之后企业级扩展留路径。

这一步非常关键：  
**他们没有先卖一个重型企业平台，而是先用低摩擦 OSS 进入开发者工作流。**

Talk Python To Me 的访谈把 LanceDB 描述为一个 developer-friendly open source database for AI，基于 Lance format，重点处理 text、images、videos、PDFs、embedding vectors 等多模态数据；并且 OSS 用于 local prototypes 和 moderate-scale production，Cloud/Enterprise 用于 billions vectors / high throughput。来源：Talk Python To Me，L18-L33、L106-L117。  
<https://talkpython.fm/episodes/show/488/multimodal-data-with-lancedb>

**阶段三：2024 年从 OSS/开发者采用进入商业化验证**

2024 年 5 月，LanceDB 宣布总融资达到 **1100 万美元**，其中包括 **800 万美元 seed round**，由 CRV、Essence VC、Swift Ventures 等参与。VentureBeat 报道中还提到 LanceDB 的商业产品包括开源数据库、hosted serverless solution、enterprise product。来源：VentureBeat，L16-L21。  
<https://venturebeat.com/ai/lancedb-raises-11m-to-build-an-open-source-multimodal-ai-database-for-app-development/>

TechCrunch 同期报道提到它的客户包括 **Midjourney、Character.ai、WeRide、Airtable** 等，并说明 LanceDB 的收入模式是卖 fully managed versions of its open source software，附加硬件加速、治理控制等能力；同时 OSS 当时约有 **60 万次/月下载**。来源：TechCrunch，L142-L143。  
<https://techcrunch.com/2024/05/15/lancedb-which-counts-midjourney-as-a-customer-is-building-databases-for-multimodal-ai/>

这说明 2024 年时它已经不是纯社区项目，而是进入了商业开源公司的典型路径：

| 层次 | 作用 |
|---|---|
| Lance format | 开放数据底座，降低锁定感 |
| LanceDB OSS | 开发者采用入口 |
| Cloud / Enterprise | 商业化承接生产规模需求 |
| 大客户 | 反向推动产品从“向量库”扩展到“AI 数据平台” |

**阶段四：2025 年明确升级为 Multimodal Lakehouse**

2025 年 6 月 24 日，LanceDB 宣布 **3000 万美元 Series A**，由 Theory Ventures 领投，CRV、Y Combinator、Databricks Ventures、RunwayML、Zero Prime、Swift 等参与。来源：LanceDB 官方公告，L432-L452。  
<https://www.lancedb.com/blog/series-a-funding>

同一天/同阶段，他们把产品叙事从“vector database for multimodal AI”进一步升级为 **Multimodal Lakehouse**。官方博客说，LanceDB Enterprise 的产品套件变成四块：

- Search
- Exploratory Data Analysis
- Feature Engineering
- Training

也就是说，Search 只是其中一部分。来源：LanceDB Multimodal Lakehouse 博客，L477-L487。  
<https://www.lancedb.com/blog/multimodal-lakehouse>

这就是产品形态的关键演变：

| 时间 | 产品认知 | 产品形态 |
|---|---|---|
| 起点 | 传统数据格式不适合 AI 多模态数据 | Lance format |
| 早期 | 开发者需要快速做 embedding / RAG / search | LanceDB OSS 嵌入式向量数据库 |
| 商业化 | 企业需要生产级检索、治理、扩展 | Cloud / Enterprise |
| 2025 后 | AI 团队不只需要 search，还需要探索、特征工程、训练数据管理 | Multimodal Lakehouse |

**3. 营收变化：公开信息很有限**

严格说：**LanceDB 没有公开披露官方 ARR、收入、毛利、客户数、净留存等 SaaS 核心财务指标。**

能看到的只有几类信号：

**融资信号**

| 时间 | 事件 | 金额/信息 |
|---|---|---|
| 2021/2022 左右 | 公司创立 / YC W22 相关背景 | CRV 页面显示 December 2021 founded；多家数据库显示 2022 founded。来源：CRV，L15-L23。 |
| 2024-05 | Seed / total funding | VentureBeat 报道总融资 1100 万美元，其中 800 万美元 seed。 |
| 2025-06-24 | Series A | 官方宣布 3000 万美元 Series A。 |

CRV 页面显示 LanceDB 的时间线为 **December 2021 founded**，**August 2023 partnered**。来源：CRV，L15-L26。  
<https://www.crv.com/companies/lancedb>

**商业客户信号**

TechCrunch 2024 年列出的客户包括 Midjourney、Character.ai、WeRide、Airtable。官方 2025 年 Series A 公告又提到 Runway、Midjourney、Character.ai 等客户在“tens of billions of vectors / petabytes of training data”规模上使用 LanceDB Enterprise。来源：LanceDB 官方公告，L457。  
<https://www.lancedb.com/blog/series-a-funding>

这说明它的商业收入大概率主要来自：

- Enterprise license / subscription
- Managed / Cloud
- BYOC / private deployment
- 大规模索引、缓存、治理、安全、支持服务

**第三方营收估算**

GetLatka 页面估计 LanceDB 2024 年 revenue 为 **230 万美元**，并称 2024 年 6 月约 **150 万美元**，2024 年 12 月约 **230 万美元**。来源：GetLatka，L61-L70。  
<https://getlatka.com/companies/lancedb.com>

但这个数据要非常谨慎使用，因为同一页面还错误地写了 LanceDB “raised $0 / bootstrapped”，这和公开融资事实冲突；页面自己也声明这些数字是估算、未被公司验证。来源：GetLatka，L163-L188。  
<https://getlatka.com/companies/lancedb.com>

所以我会这样判断：

- **官方可确认：** 已经商业化，收入来自 managed/enterprise，2024 年已有知名付费/生产客户，2025 年完成 3000 万美元 A 轮。
- **第三方估算：** 2024 年收入可能在百万美元级别，GetLatka 给出约 230 万美元估算。
- **不可确认：** 2025/2026 ARR、增长率、毛利、净留存、客户数，没有可靠公开口径。

如果从融资节奏和客户质量看，它更像是一个 **“早期但已有高质量 enterprise pull 的商业开源基础设施公司”**，而不是一个已经披露规模化 ARR 的成熟 SaaS。

**4. 他们是怎么找到这个产品形态的**

我认为 LanceDB 找到产品形态的路径可以概括为：  
**先从创始人亲身遇到的底层数据痛点出发，再借 AI 时代的向量检索需求找到分发入口，最后被大客户的生产问题拉成更大的平台。**

具体有五步。

**第一步：不是看“向量数据库热”，而是看“AI 数据形态变了”**

他们的原始问题不是“怎么做 ANN 检索”，而是：

- embeddings 不像普通字段；
- 图片、视频、音频、PDF 太大；
- 多模态数据需要随机访问；
- 模型训练、检索、数据探索、特征工程都要访问同一批数据；
- Parquet / WebDataset / Spark / pandas 组合起来能做，但很笨重；
- 数据经常在 data lake、vector DB、search engine、feature store 之间复制。

这就是所谓的 **workload shift**：  
上一代数据基础设施服务的是 BI、batch analytics、structured table；新一代 AI workload 需要处理多模态、embedding、原始对象、metadata、在线检索和离线训练的混合需求。

**第二步：先做“底层格式”而不是只做应用层**

Lance format 是他们的战略核心。因为如果只是做一个向量库，很容易被：

- Postgres + pgvector
- OpenSearch / Elasticsearch
- Qdrant / Milvus / Weaviate / Pinecone
- 云厂商内置向量检索
- LLM framework 自带 vector store

挤压成一个功能点。

但如果控制的是 **AI 数据的底层格式和表抽象**，它就可以往多个方向延展：

- 检索
- 数据集管理
- 特征工程
- 训练数据加载
- 多模态分析
- object storage 上的数据系统

这也是它后来能从 vector DB 升级成 multimodal lakehouse 的基础。

**第三步：用向量数据库作为 go-to-market wedge**

尽管底层愿景更大，但市场教育不能一开始就太抽象。所以 LanceDB 早期对外很容易被理解为：

> 一个开源、嵌入式、低摩擦的 vector database / retrieval library。

这个切入点好在：

- RAG 爆发，需求真实存在。
- 开发者可以几分钟上手。
- OSS 分发成本低。
- 和 LangChain、LlamaIndex、agent framework 集成容易。
- 小项目可以免费用，大规模生产自然转 enterprise。

这是一种很典型的基础设施创业路径：  
**用一个明确、紧迫、低门槛的开发者需求切进去，但底层架构为更大的系统留空间。**

**第四步：被生产客户拉宽产品边界**

当客户从 demo / RAG prototype 进入生产，问题会从“能不能查向量”变成：

- embeddings 怎么生成和更新？
- 原始文档、图片、视频在哪里？
- metadata 和权限怎么跟向量保持一致？
- 训练集怎么版本化？
- 模型升级后怎么 backfill 特征？
- 10B/100B vectors 怎么控制成本？
- S3/object storage 上如何低延迟检索？
- agent 带来 10 倍、100 倍、1000 倍 QPS 后怎么撑？

O’Reilly 访谈里 Chang She 明确说，单纯向量数据库太窄；即便只是 text embeddings，embedding 创建、metadata、原始文档、serving 都在 vector DB 之外；企业里常见流程是 data lake 离线生成 embedding，然后复制到 vector DB，产生同步、搬运、成本和复杂度。来源：O’Reilly，L400-L407。  
<https://www.oreilly.com/radar/podcast/generative-ai-in-the-real-world-chang-she-on-data-infrastructure-for-ai/>

这就是他们产品从“vector DB”扩展到“Multimodal Lakehouse”的根本原因。

**第五步：把大方向重新定义成“AI data lifecycle”**

LanceDB 不再只说“我帮你做 vector search”，而是说：

> AI 团队需要在一个统一数据层上完成 search、exploration、feature engineering、training。

官方 Series A 公告说，传统 data lake 擅长 tabular/BI，但不适合 AI workloads、online serving、多模态数据；vector database 擅长 online vector search，但不适合 feature engineering、data exploration、training。结果是工程师要拼很多工具。来源：LanceDB 官方公告，L471-L474。  
<https://www.lancedb.com/blog/series-a-funding>

所以他们找到的产品形态不是“更快的向量库”，而是：

> **围绕 AI 数据全生命周期的一体化数据基础设施。**

**5. 对当前 AI 大方向的判断**

我把 LanceDB 的判断归纳成一句话：

> **AI 的核心瓶颈会从模型能力本身，逐步转向数据、上下文、检索、训练数据流、成本和生产工程。**

具体看，他们押的是几个趋势。

**趋势一：多模态成为默认，不是小众场景**

官方博客说，现代企业已经有大量“多模态数据”，即使它们不这么叫：PDF、slide deck、合同、销售电话、邮件、dashboard、图片、音频、结构化 metadata。来源：LanceDB Multimodal Lakehouse 博客，L451-L463。  
<https://www.lancedb.com/blog/multimodal-lakehouse>

这很重要。很多人一听多模态，只想到 Midjourney、Runway、自动驾驶、机器人。但 LanceDB 的判断是：  
**企业知识本身就是多模态的，只是过去没有被 AI 系统充分消费。**

**趋势二：数据规模不是线性增长，而是数量级变化**

LanceDB 官方 Series A 公告说，他们从客户那里学到三点：

- 数据种类从 tabular 扩展到 embeddings、documents、images、video。
- 多模态数据量比表格数据大 **3-9 个数量级**。
- LLM/VLM 让数据生成速度不再受人工动作限制。来源：LanceDB 官方公告，L460-L463。  
<https://www.lancedb.com/blog/series-a-funding>

这解释了为什么他们强调 object storage、disk-first、lakehouse、compute-storage separation。  
因为如果数据量真的是 10B vectors、petabytes 级别，全放内存/NVMe/专用服务内部存储，成本会非常难看。

**趋势三：agent 会放大底层数据系统压力**

O’Reilly 访谈里，Chang She 说 2025/2026 的核心驱动因素之一是 scale，而 scale 的主要来源是 agents：数据量、query throughput、性能和延迟要求都被放大。他还说一些客户的规模可能比一两年前大 1000 倍。来源：O’Reilly，L455-L466。  
<https://www.oreilly.com/radar/podcast/generative-ai-in-the-real-world-chang-she-on-data-infrastructure-for-ai/>

更具体地，他举例说，agent 可以自动跑大量 feature variant 实验，导致底层数据基础设施负载上升；从几年前普通 RAG 的 hundreds QPS，走向 agent 场景里的 100K QPS。来源：O’Reilly，L468-L474。  
<https://www.oreilly.com/radar/podcast/generative-ai-in-the-real-world-chang-she-on-data-infrastructure-for-ai/>

所以 LanceDB 对 AI 大方向的判断不是“每个人都会有聊天机器人”这么浅，而是：

> 当 agent 真的进入企业生产，会制造大量检索、读写、特征实验、上下文装载、权限控制、临时数据环境需求。底层数据系统会成为瓶颈。

**趋势四：模型越来越强，但企业价值落地越来越依赖“数据可用性”**

模型层竞争太激烈，且很多能力会商品化。企业真正难的是：

- 数据能不能被模型安全访问；
- 多模态数据能不能统一管理；
- retrieval 是否准确、低延迟、可控成本；
- training / fine-tuning 数据是否可追踪、可版本化；
- agent 是否能在权限边界内拿到正确上下文；
- 数据 pipeline 是否能从 notebook 平滑进入 production。

这也是为什么 LanceDB 的口号从 database/search 往 “Build better models faster” 和 multimodal lakehouse 靠。

**6. 找具体 AI 赛道的逻辑：从 LanceDB 可以抽象出一套方法**

如果把 LanceDB 当案例看，它给 AI 创业/选赛道的启发不是“去做向量数据库”，而是下面这套逻辑。

**逻辑一：不要从大词开始，要从 workload breakage 开始**

“大方向”是 AI、多模态、agent、RAG、模型训练。  
但大方向太宽，不能直接创业。要问：

- 哪个工作流因为 AI 变得明显更痛？
- 以前的工具为什么不够？
- 这个痛点是 demo 痛，还是 production 痛？
- 数据量、频率、成本、风险是否在放大？
- 用户现在是否已经在用 3-5 个工具拼接？

LanceDB 找到的是：  
**AI 多模态数据生命周期打破了传统 data stack。**

对应到其他赛道，也可以这样看：

| 大方向 | 找赛道时要问的具体问题 |
|---|---|
| Agent | agent 增多后，哪个系统的 QPS、权限、审计、状态管理被打爆？ |
| Multimodal | 图片/视频/音频/PDF 进入企业流程后，哪个数据层无法承载？ |
| AI coding | 代码生成之后，测试、review、安全、迁移、依赖管理哪里成为瓶颈？ |
| AI sales/support | 模型能生成话术后，数据质量、工作流集成、合规、CRM 写回哪里最痛？ |
| AI training | 算力贵之后，数据加载、去重、采样、版本化、评测哪里影响 GPU 利用率？ |

**逻辑二：优先找“AI 增长带来的二阶问题”**

一阶问题是：用 AI 做某个功能。  
二阶问题是：大量人用 AI 之后，某个基础设施/流程开始崩。

LanceDB 押的是二阶问题：

- embedding 多了，向量库不够；
- 多模态数据多了，Parquet/WebDataset 不够；
- agent 多了，检索 QPS 和上下文系统不够；
- 训练数据多了，传统数据湖不够；
- 企业上生产了，权限、治理、成本、部署方式不够。

二阶问题通常比一阶应用更适合基础设施公司，因为它：

- 生命周期更长；
- 客户付费能力更强；
- 更接近预算；
- 技术壁垒更高；
- 不容易被一个模型 API 更新直接吃掉。

**逻辑三：找“已有预算 + 新技术破坏旧方案”的交叉点**

LanceDB 不只是新技术产品，它也吃到了旧预算：

- data infrastructure budget
- ML platform budget
- search/retrieval budget
- cloud infrastructure cost reduction
- enterprise AI platform budget
- open-source support / enterprise license budget

这是很关键的商业判断。  
如果一个 AI 赛道只有“用户觉得酷”，但没有旧预算可替代，也没有新预算明确形成，商业化会很难。

好的赛道通常有这个结构：

> 新 AI workload 出现 → 旧系统勉强能做但成本/体验/规模不行 → 用户已经为旧系统付钱 → 新产品用更贴合 AI 的架构替代旧拼接方案。

LanceDB 替代/整合的旧拼接方案大概是：

- S3 + Parquet/WebDataset
- Spark/Ray preprocessing
- Elasticsearch/OpenSearch
- vector DB
- feature store
- training data loader
- metadata DB
- ad hoc scripts

**逻辑四：用一个“窄入口”进入，但架构要能长成平台**

LanceDB 的入口是 vector search / embedded vector DB。  
但它底层是 Lance format，所以能长成 multimodal lakehouse。

这是 AI 赛道选择里很重要的一点：

| 不好的入口 | 好的入口 |
|---|---|
| 一开始就讲大平台，用户不知道为什么现在要买 | 先解决一个高频痛点 |
| 只做一个 feature，后面没有扩展面 | 入口小，但底层数据/工作流位置关键 |
| 靠 prompt wrapper，模型升级容易被替代 | 控制数据、权限、工作流、系统集成或评测闭环 |
| 只服务 demo | 能自然进入 production scale |

所以具体赛道可以用这个问题筛：

> 这个产品有没有一个足够简单的 wedge？这个 wedge 背后有没有一个更大的 system-of-record 或 workflow-control 位置？

LanceDB 的答案是：  
wedge 是 OSS vector DB；更大位置是 AI 数据 lakehouse / lifecycle layer。

**逻辑五：选择“模型越强，我越重要”的赛道**

很多 AI 应用会被模型进步吞掉。  
更稳的赛道是：模型越强、调用越多、agent 越多、数据越多，你的产品越重要。

LanceDB 就是这个逻辑：

- 模型越多模态 → 数据格式越重要。
- RAG 越多 → retrieval 越重要。
- agent 越多 → QPS、权限、上下文、缓存越重要。
- 训练越频繁 → 数据版本、特征工程、data loading 越重要。
- 企业越上生产 → governance、BYOC、安全、成本越重要。

这类赛道更有“顺风性”。

**逻辑六：开源/开发者产品适合基础设施，但商业化必须落到企业痛点**

LanceDB 的 OSS 做得轻，是为了 adoption；但钱不是从个人 notebook 来的，而是从：

- 大规模数据；
- 生产 SLA；
- 自动索引；
- 分布式查询；
- BYOC；
- enterprise security；
- governance；
- support；
- training / feature engineering 工作流；

这些企业级需求来。

所以如果做 AI infra，不要只看 GitHub stars。要问：

- OSS 用户中谁会自然长大成付费客户？
- 免费版和付费版边界是否清晰？
- 企业是否有必须付费的 scale/security/governance 痛点？
- 数据是否在用户侧足够核心，迁移之后是否有粘性？

**7. 对 LanceDB 的一句话判断**

LanceDB 的真正定位不是“向量数据库公司”，而是：

> **用开源 Lance format 作为 AI 多模态数据底座，用 OSS LanceDB 获取开发者，用 Enterprise/Cloud/BYOC 承接大客户的搜索、特征工程、数据探索和训练数据管理需求。**

它找到赛道的路径是：

1. 创始人从 Tubi、Cruise、Cloudera、pandas/Hadoop 经验里看到传统数据栈处理 AI 数据很差。
2. 先做 Lance format，解决多模态 AI 数据的底层表示、随机访问、版本和索引问题。
3. 用 vector DB/RAG 作为开发者切入点。
4. 随着客户进入生产，把产品从 search 扩展到 multimodal lakehouse。
5. 押注 AI 的长期瓶颈会从“模型能不能生成”转向“数据能不能被准确、低成本、合规、规模化地使用”。

如果把这个案例抽象成选 AI 赛道的方法，就是：

> **不要追 AI 热词；找 AI 让旧基础设施失效的地方。优先选数据量、调用量、复杂度、合规要求都会随 AI 使用增长而放大的环节。用一个窄而急的痛点进入，用底层架构占住更大的工作流位置。**

<!-- /message -->
