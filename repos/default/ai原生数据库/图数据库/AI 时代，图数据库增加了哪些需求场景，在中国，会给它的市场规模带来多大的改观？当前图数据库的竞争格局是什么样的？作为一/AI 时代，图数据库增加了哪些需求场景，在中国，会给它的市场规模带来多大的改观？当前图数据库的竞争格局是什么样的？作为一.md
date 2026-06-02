---
id: "ses_0bdae31742a2d9ea"
title: "AI 时代，图数据库增加了哪些需求场景，在中国，会给它的市场规模带来多大的改观？当前图数据库的竞争格局是什么样的？作为一"
createdAt: "2026-05-25T21:48:29.723Z"
updatedAt: "2026-06-02T00:25:10.977Z"
---

# AI 时代，图数据库增加了哪些需求场景，在中国，会给它的市场规模带来多大的改观？当前图数据库的竞争格局是什么样的？作为一

<!-- message:id=msg_a9738d922dffdbdc role=user createdAt=2026-05-25T21%3A52%3A10.351Z -->

## User

AI 时代，图数据库增加了哪些需求场景，在中国，会给它的市场规模带来多大的改观？当前图数据库的竞争格局是什么样的？作为一家新进公司，突破点和定位应该怎么找？

<!-- /message -->
<!-- message:id=msg_3b83ee41c868154e role=error createdAt=2026-05-25T21%3A55%3A10.403Z runId=run_98585952ee0b2a09 status=failed -->

## Error

Agent run timed out after 180s. The run metadata keeps the CLI stderr tail for diagnostics.

<!-- /message -->
<!-- message:id=msg_0ebc9adba2e170ca role=user createdAt=2026-05-31T23%3A49%3A04.082Z -->

## User

以age这个pg的图插件为例，说明图数据库的架构包括哪些方面，如何考虑在pg里实现一个图数据库的架构问题，技术路径如何选择

<!-- /message -->
<!-- message:id=msg_8cec34b38078528f role=assistant createdAt=2026-05-31T23%3A53%3A04.963Z runId=run_77805e4039f41267 status=succeeded -->

## Assistant

下面以 Apache AGE 这个 PostgreSQL 图扩展为例，把问题拆成三层讲：**图数据库本身的架构要素**、**AGE 在 PG 里是怎么做的**、**如果自己在 PG 里实现图数据库，技术路径怎么选**。

**一、图数据库架构包括哪些方面**

图数据库不是“多几张点边表”这么简单。完整架构通常至少包括这些层：

1. **图模型层**

   先决定支持哪种图模型：

   - 属性图 Property Graph：点、边、标签、属性。Neo4j、AGE、NebulaGraph 更接近这一类。
   - RDF/语义图：三元组、URI、本体、SPARQL。更适合语义网、知识本体、推理。
   - 多模型：关系 + 图 + JSON + 向量等混合模型。

   AGE 采用的是属性图模型。官方文档里定义图由 vertex 和 edge 组成，每个点和边都有属性 map，edge 是两个 vertex 之间的有向连接。([age.apache.org](https://age.apache.org/age-manual/master/intro/graphs.html))

2. **数据定义与元数据层**

   图数据库需要管理：

   - graph：一个数据库里可以有多个图。
   - label：点标签、边标签。
   - id：点边全局或图内唯一标识。
   - schema：是否强 schema、弱 schema、schema optional。
   - catalog：记录图、标签、物理表、序列、索引、约束。

   这层决定后续查询优化和运维能力。没有稳定元数据，查询编译器、权限、迁移、统计信息都很难做好。

3. **存储层**

   图存储有几种典型路线：

   - **关系表存储**：点表、边表，边表里有 `start_id`、`end_id`。PG 内实现最自然。
   - **邻接表/CSR 存储**：面向遍历优化，适合多跳、图算法。
   - **KV/LSM 存储**：按点 ID、边方向、边类型组织 key，适合大规模分布式图。
   - **列式/分析型存储**：适合 PageRank、社区发现、批量图分析。
   - **混合存储**：事务数据走行存，分析图走列式或内存快照。

   AGE 选择的是关系表存储，这使它能最大化复用 PostgreSQL 的 MVCC、WAL、事务、索引、备份和权限能力。

4. **查询语言层**

   常见选择：

   - Cypher / openCypher：开发者友好，图模式匹配表达力强。
   - Gremlin：遍历式 API，灵活但工程集成风格不同。
   - SPARQL：RDF 生态。
   - SQL/PGQ：SQL 标准里的 property graph query 方向。
   - 自定义 DSL：适合垂直场景，但生态成本高。

   AGE 支持 openCypher 风格查询，并通过 `cypher(graph_name, $$ ... $$)` 函数嵌入 SQL。官方 README 也强调它支持 SQL 与 openCypher 混合查询。([github.com](https://github.com/apache/age))

5. **查询编译与执行层**

   图查询和 SQL 查询最大的不同在于 traversal：

   - 点匹配：`MATCH (n:Person)`
   - 边扩展：`MATCH (a)-[r]->(b)`
   - 多跳路径：`MATCH p=(a)-[*1..5]->(b)`
   - 路径去重、环检测、最短路径、可达性。
   - 属性过滤、标签过滤、方向过滤。
   - 图模式和关系表 join 的混合优化。

   一个成熟图数据库需要有图感知的 parser、semantic analyzer、rewriter、planner、executor 和 cost model。AGE 官方架构说明里明确提到 Cypher 查询会经过 parsing、transform、planner/optimizer、executor，并运行在 PostgreSQL 事务/cache/storage 层之上。([age.apache.org](https://age.apache.org/overview/))

6. **索引与统计层**

   图查询性能主要取决于：

   - 点 label 索引。
   - 边 label 索引。
   - 边的 `start_id` / `end_id` 索引。
   - 属性索引。
   - 组合索引：如 `label + property`、`start_id + edge_type`。
   - 图统计信息：每个 label 的点数、边数、度分布、属性选择率、路径选择率。
   - 多跳路径的代价估算。

   PG 插件路线的难点不在“建索引”，而在“让优化器知道图查询应该如何利用这些索引”。

7. **事务与一致性层**

   图数据库必须回答：

   - 点边写入是否 ACID？
   - 边引用的点是否必须存在？
   - 删除点时边如何处理？
   - 多跳遍历过程中看到的是哪个快照？
   - 并发写图时路径查询是否稳定？
   - 大图批量导入是否影响在线查询？

   AGE 的优势是直接复用 PostgreSQL 的事务系统，官方说明也强调 Cypher 查询运行在 PostgreSQL 既有 ACID 事务系统上。([age.apache.org](https://age.apache.org/overview/))

8. **算法与分析层**

   图数据库通常还需要：

   - PageRank。
   - 最短路径。
   - 连通分量。
   - 社区发现。
   - 相似度。
   - 路径枚举。
   - 风险传播、影响力传播。
   - 知识图谱推理或规则计算。

   这层 AGE 本体不是特别强，更多需要外部算法库、SQL 函数、物化视图、批处理或专门的图计算引擎配合。

9. **生态与工程层**

   包括：

   - 驱动 SDK。
   - 可视化工具。
   - 导入导出。
   - 权限。
   - 多租户。
   - 监控。
   - 备份恢复。
   - 版本升级。
   - 与 SQL、BI、AI/RAG 系统集成。

   AGE 有 AGE Viewer 作为图可视化工具，README 提到它提供 graph/table 结果探索能力。([github.com](https://github.com/apache/age))

**二、AGE 在 PostgreSQL 里的架构做法**

AGE 的核心思路可以概括为：

> 用 PostgreSQL 扩展机制提供 Cypher 入口，把图数据落到 PostgreSQL 普通表里，用 PG 的事务、存储、索引、执行器承载图能力。

它不是另起一个独立图存储引擎，也不是直接把 PostgreSQL 改造成专用图数据库。它更像一个 **Cypher 到 PG 执行体系的图扩展层**。

**1. 扩展入口**

AGE 作为 PostgreSQL extension 安装。PG 的扩展机制本身就是把类型、函数、操作符、索引支持、catalog 对象等打包成 extension，通过 `CREATE EXTENSION` 加载。PostgreSQL 官方文档说明，一个 extension 至少包含 control 文件和 SQL 安装脚本，`CREATE EXTENSION` 会创建对应 SQL 对象并记录其归属。([postgresql.org](https://www.postgresql.org/docs/current/extend-extensions.html?utm_source=openai))

AGE 安装后，典型使用方式是：

`​``sql
CREATE EXTENSION age;
LOAD 'age';
SET search_path = ag_catalog, "$user", public;
`​``

然后创建图：

`​``sql
SELECT * FROM ag_catalog.create_graph('graph_name');
`​``

AGE 文档说明，`create_graph` 位于 `ag_catalog` namespace，调用后会自动创建图所需表。([age.apache.org](https://age.apache.org/age-manual/master/intro/graphs.html))

**2. 图与表的映射**

AGE 的物理存储设计很关键。

创建一个 graph 后，AGE 会为每个 graph 创建一个 PostgreSQL namespace/schema。图的名字和 namespace 会记录在 `ag_catalog.ag_graph` 表中。([age.apache.org](https://age.apache.org/age-manual/master/intro/graphs.html))

每个 graph namespace 下会有两张基础父表：

`​``text
_ag_label_vertex
_ag_label_edge
`​``

它们是所有点 label 和边 label 表的父表。AGE 文档说明，新增 vertex label 会生成对应 label 表；edge label 也是类似逻辑；用 Cypher 创建点边时也会自动创建这些 label 表。([age.apache.org](https://age.apache.org/age-manual/master/intro/graphs.html))

可以把 AGE 的映射理解成：

`​``text
PostgreSQL database
  └── ag_catalog
        ├── ag_graph       -- graph 元数据
        └── ag_label       -- label 元数据
  └── graph_name schema
        ├── _ag_label_vertex
        ├── _ag_label_edge
        ├── Person         -- vertex label table
        └── FATHER_OF      -- edge label table
`​``

点通常包含：

`​``text
id
properties
`​``

边通常包含：

`​``text
id
start_id
end_id
properties
`​``

AGE 的 `agtype` 文档里也定义了 vertex 的 `id / label / properties`，edge 的 `id / start_id / end_id / label / properties`。([age.apache.org](https://age.apache.org/age-manual/master/intro/types.html?utm_source=openai))

这个设计的好处是：

- 图数据仍然是 PostgreSQL relation。
- label 可以映射为表。
- edge 的 `start_id`、`end_id` 可以走 B-tree 索引。
- 属性可以放在 `agtype` 里，类似 JSONB 风格。
- 事务、WAL、备份、权限、VACUUM、复制等能力都能复用 PG。

代价是：

- 多跳 traversal 本质上会变成多次 join / recursive expansion。
- 对高度动态的路径查询，PG 原生优化器并不天然理解图遍历代价。
- 属性全塞进 `agtype`，灵活性强，但 schema、统计信息、属性索引优化会变复杂。
- 大规模深度遍历不如原生邻接存储或专门图引擎。

**3. 查询入口：Cypher 作为 SQL 函数**

AGE 的查询通常长这样：

`​``sql
SELECT *
FROM cypher('graph_name', $$
    MATCH (a:Person)-[r:FATHER_OF]->(b:Person)
    RETURN a, r, b
$$) AS (a agtype, r agtype, b agtype);
`​``

也就是说，Cypher 查询不是 PG SQL parser 的一等语法，而是被包在 `cypher()` 函数调用里。这个选择非常现实：

- 不需要 fork PostgreSQL 改 SQL grammar。
- 和 SQL 可以自然组合。
- 扩展安装和升级成本低。
- PG 客户端仍然走普通 SQL 协议。

AGE 文档也展示了 Cypher 可以放在 CTE、JOIN、子查询等 SQL 结构里；例如 Cypher 可以用于 CTE，也可以作为 JOIN 的一部分。([age.apache.org](https://age.apache.org/age-manual/master/advanced/advanced.html))

这就是 AGE 的一个重要产品定位：**不是替代 SQL，而是在 SQL 里嵌入图查询**。

**4. 查询处理管线**

AGE 官方 overview 把架构分成几步：

`​``text
Cypher parser
  -> query transform
  -> planner / optimizer
  -> executor
  -> PostgreSQL transaction / cache / storage
`​``

官方说明里提到：

- 解析嵌入在 `cypher` function calls 里的 Cypher 查询。
- 将 Cypher query 转换成 Query tree，并作为 subquery node 挂载。
- planner/optimizer 理解部分 graph operations 并生成 graph operation 相关 plan nodes。
- executor 执行这些 graph operation plan nodes。
- 查询运行在 PG 既有事务系统上。([age.apache.org](https://age.apache.org/overview/))

这说明 AGE 不是简单字符串拼 SQL，而是有自己的 parser / transform / planner / executor 扩展层。

不过从架构定位看，它仍然高度依赖 PostgreSQL 的执行和存储体系。AGE README 也说它的基本原则是创建一个同时处理关系模型和图模型的 single storage，让用户可以同时使用 ANSI SQL 和 openCypher。([github.com](https://github.com/apache/age))

**5. 类型系统：agtype**

AGE 引入了自定义类型 `agtype`。官方文档说，AGE 返回的唯一数据类型是 `agtype`，它是 JSON 的超集，并且是自定义 JSONB 实现。([age.apache.org](https://age.apache.org/age-manual/master/intro/types.html))

`agtype` 承载：

- scalar：integer、float、numeric、boolean、string、null。
- composite：list、map。
- graph entity：vertex、edge、path。
- 属性 map。
- Cypher 结果值。

这层非常重要，因为 Cypher 的类型系统和 PostgreSQL 类型系统不完全一致。比如：

- Cypher 有 vertex、edge、path。
- Cypher null 语义与 SQL null 接近但表达位置不同。
- 属性 map 是动态结构。
- path 是点边序列，不是普通 SQL row。

如果在 PG 内实现图数据库，是否设计一个类似 `agtype` 的统一图值类型，是核心架构决策之一。

**6. SQL + Cypher 混合能力**

AGE 的优势之一是混合查询。例如：

`​``sql
SELECT id,
       graph_query.name = t.name AS names_match
FROM schema_name.sql_person AS t
JOIN cypher('graph_name', $$
    MATCH (n:Person)
    RETURN n.name, id(n)
$$) AS graph_query(name agtype, id agtype)
ON t.person_id = graph_query.id;
`​``

AGE 文档明确说明 Cypher 可以用于 CTE、JOIN、`IN`、`EXISTS` 等 SQL 场景，也可以同时查询多个 graph。([age.apache.org](https://age.apache.org/age-manual/master/advanced/advanced.html))

但是也有边界。例如 AGE 文档指出，Cypher 不能直接写在 SQL expression 中，必须在 `FROM` 子句里；如果放进 subquery，则可以像 SQL 查询一样工作。([age.apache.org](https://age.apache.org/age-manual/master/advanced/advanced.html))

另外，AGE 不支持在 Cypher 里直接写 SQL，但可以通过用户自定义函数间接调用 SQL；文档也说明 set-returning function 当前不支持。([age.apache.org](https://age.apache.org/age-manual/master/advanced/sql_in_cypher.html))

这暴露了 PG 插件路线的一个典型取舍：

- 优点：复用 SQL 生态，混合查询强。
- 缺点：Cypher 和 SQL 不是完全同一语法层的一等公民，存在表达边界。

**三、如果在 PG 里实现图数据库，架构问题怎么考虑**

如果你要自己做一个 “PG 里的图数据库”，不要一开始就想“我要做 Neo4j in PostgreSQL”。更务实的方式是先回答几个架构问题。

**1. 目标到底是图查询、图分析，还是图存储？**

三者差别很大：

| 目标 | 典型需求 | 架构重点 |
|---|---|---|
| 图查询 | 业务系统里查关系、路径、邻居、规则 | SQL/Cypher 集成、事务、索引 |
| 图分析 | PageRank、社区发现、批量算法 | 列式/内存/批计算/并行 |
| 图存储 | 海量点边、高并发读写 | 分区、分布式、邻接存储 |
| 知识图谱 | 实体关系、语义、推理、RAG | schema、本体、实体解析、检索 |
| AI/RAG 增强 | GraphRAG、关系约束、上下文扩展 | 图 + 向量 + 文档混合检索 |

如果目标是“让已有 PG 用户低成本获得图查询能力”，AGE 路线很好。

如果目标是“千亿边、深度遍历、图算法”，AGE 这种纯 PG relation 存储会吃力，需要专用存储或外部图计算。

**2. 图数据要不要和关系数据共存？**

这是 PG 内实现最核心的理由。

适合放 PG 内：

- 已有核心业务数据就在 PG。
- 图关系是业务数据的补充表达。
- 强事务、一致性、审计、权限很重要。
- 图查询经常要 join 关系表。
- 数据规模中等，主要是 1 到 3 跳关系查询。
- 希望降低运维复杂度。

不适合只靠 PG 内插件：

- 频繁 5 到 10 跳甚至更深 traversal。
- 图算法是主负载。
- 超大规模边表。
- 要低延迟毫秒级复杂路径查询。
- 要分布式图存储和计算。
- 查询模式高度不确定且依赖路径枚举。

**3. 选 Cypher、SQL/PGQ 还是 SQL 函数 API？**

可以分三档：

**第一档：函数式 API**

例如：

`​``sql
SELECT graph_neighbors('g', vertex_id, 'KNOWS');
SELECT graph_shortest_path('g', from_id, to_id);
`​``

优点：

- 实现快。
- 对 PG 友好。
- 适合垂直场景。
- 不需要复杂 parser。

缺点：

- 表达力弱。
- 生态弱。
- 不像真正图数据库。

适合 MVP 或行业场景，例如风控、供应链、组织关系。

**第二档：Cypher 嵌入 SQL，类似 AGE**

例如：

`​``sql
SELECT *
FROM cypher('g', $$
  MATCH (a:User)-[:PAY]->(b:Merchant)
  RETURN a, b
$$) AS (a agtype, b agtype);
`​``

优点：

- 开发者熟悉。
- 图表达力强。
- 可复用 openCypher 生态。
- 不需要改 PG SQL 语法。

缺点：

- 需要自己做 parser、transform、语义分析。
- 和 SQL optimizer 的融合有限。
- Cypher 不是 PG 语法的一等公民。

适合做通用图数据库插件。

**第三档：SQL/PGQ 或深度 SQL 集成**

目标是让图查询成为 SQL 里的标准组成部分，而不是函数参数里的字符串。

优点：

- 长期标准化方向好。
- 和 SQL optimizer 理论上融合更深。
- 对企业客户更友好。

缺点：

- 工程复杂度更高。
- PG core 未必提供足够直接的扩展点。
- 可能需要 fork 或大量 hook。

适合长期产品，不适合早期快速验证。

**4. 存储模型怎么选？**

PG 内有几种技术路径。

**路线 A：AGE 式 label table**

`​``text
graph schema
  ├── Person(id, properties)
  ├── Company(id, properties)
  ├── KNOWS(id, start_id, end_id, properties)
  └── WORKS_AT(id, start_id, end_id, properties)
`​``

优点：

- 每个 label 一张表，天然可按 label 裁剪。
- `start_id`、`end_id` 容易建索引。
- 查询计划更容易利用 PG relation。
- 权限、统计、VACUUM、分区都比较自然。
- 和 AGE 思路一致，风险低。

缺点：

- label 很多时表数量膨胀。
- 跨 label 查询需要 union/append。
- 动态属性统计差。
- 多跳 traversal 仍然是 join/recursive 逻辑。

适合多数 PG 图插件。

**路线 B：统一点表 + 统一边表**

`​``text
vertices(id, labels, properties)
edges(id, start_id, end_id, type, properties)
`​``

优点：

- 表数量少。
- 模型简单。
- 多租户和动态 label 好处理。
- DDL 少。

缺点：

- label 过滤依赖列/数组/JSON 索引。
- 大表膨胀快。
- 不同 label 的属性差异难优化。
- 高频 label 查询可能性能不如分表。

适合 schema 很动态、label 数量很多但查询模式不重的场景。

**路线 C：基础表 + 邻接索引表**

`​``text
vertices(id, label, properties)
edges(id, start_id, end_id, type, properties)
adj_out(start_id, type, end_id, edge_id)
adj_in(end_id, type, start_id, edge_id)
`​``

优点：

- 遍历性能更好。
- 适合多跳 expansion。
- 可以专门优化邻居访问。
- 可以和 PG 表事务保持一致。

缺点：

- 写入多维护两套结构。
- 一致性、VACUUM、更新成本更高。
- 查询编译器需要知道 adjacency 结构。

适合想比 AGE 更强调 traversal 性能的实现。

**路线 D：PG 表做主存储，外部图索引做加速**

`​``text
PostgreSQL: authoritative storage
External graph index: adjacency / CSR / RocksDB / memory graph
`​``

优点：

- PG 保留事务主库地位。
- 外部索引可专门优化 traversal。
- 可服务图算法和低延迟路径查询。

缺点：

- 双写或异步同步复杂。
- 一致性语义难。
- 故障恢复复杂。
- 运维成本高。

适合高性能产品化，但不适合第一阶段。

**5. 属性怎么存？**

常见选择：

**方案 A：属性全放 JSONB/agtype**

`​``text
properties agtype/jsonb
`​``

优点：

- 灵活。
- schema optional。
- 和 Cypher map 天然匹配。
- DDL 少。

缺点：

- 类型约束弱。
- 统计信息弱。
- 属性索引要额外设计。
- 热字段查询性能不如普通列。

AGE 就是类似思路，用 `agtype` 承载属性和返回值。

**方案 B：热属性列化，冷属性放 JSONB**

`​``text
id
name
risk_score
created_at
properties
`​``

优点：

- 高频过滤字段性能好。
- 类型、约束、统计信息好。
- 仍保留灵活扩展。

缺点：

- schema 管理复杂。
- Cypher 属性访问要映射到列或 JSON。
- ALTER TABLE 成本需要处理。

适合企业级生产场景。我的建议是：**不要纯 JSON 化一切**。如果要做商业产品，必须允许用户把热属性提升为 typed column 或 generated column。

**方案 C：EAV 属性表**

`​``text
vertex_properties(vertex_id, key, value)
`​``

优点：

- 属性索引灵活。
- 稀疏属性友好。

缺点：

- 查询 join 爆炸。
- 写入和优化复杂。
- PG 里通常不如 JSONB + 表达式索引实用。

一般不建议作为主方案。

**6. ID 怎么设计？**

AGE 使用 graphid 这类图 ID 抽象。自己设计时要考虑：

- ID 是否全局唯一？
- 是否编码 graph id、label id？
- 是否支持分区？
- 是否支持导入外部 ID？
- 边 ID 是否独立？
- start/end 是否直接引用 vertex id？
- 是否加外键？

建议：

`​``text
internal_id: bigint / uint64 风格，用于内部 join 和索引
external_id: text/uuid，可选，用于业务幂等导入
label_id: smallint/int，用于压缩和优化
`​``

不要一开始就只用业务 ID 做图内部 ID。业务 ID 往往太长、类型不一、join 成本高。

**7. 索引怎么设计？**

PG 内图数据库至少要有：

`​``sql
-- 点
PRIMARY KEY (id)

-- 边
PRIMARY KEY (id)
INDEX ON edge_label(start_id)
INDEX ON edge_label(end_id)
INDEX ON edge_label(start_id, end_id)
INDEX ON edge_label(end_id, start_id)
`​``

如果边有 type 列，而不是按 label 分表，则需要：

`​``sql
INDEX ON edges(start_id, type, end_id);
INDEX ON edges(end_id, type, start_id);
`​``

属性索引分两类：

`​``sql
-- 热属性
INDEX ON person ((properties->>'name'));

-- 或热字段列化
INDEX ON person(name);

-- JSON/agtype 全文式属性检索
GIN(properties)
`​``

PostgreSQL 的 GIN 适合复合值里的元素检索，官方文档说明 GIN 面向 composite value 中 key/element 的检索场景。([postgresql.org](https://www.postgresql.org/docs/current/gin.html?utm_source=openai))

但是要注意：GIN/JSONB 索引不是图遍历索引。它能加速属性过滤，不能直接解决多跳 traversal 的代价问题。

**8. Planner/Executor 怎么做？**

这是 PG 内实现图数据库的技术分水岭。

**初级实现：Cypher 编译成 SQL**

把：

`​``cypher
MATCH (a:Person)-[:KNOWS]->(b:Person)
WHERE a.name = 'Tom'
RETURN b
`​``

转成类似：

`​``sql
SELECT b.*
FROM graph.Person a
JOIN graph.KNOWS e ON e.start_id = a.id
JOIN graph.Person b ON b.id = e.end_id
WHERE a.properties->>'name' = 'Tom';
`​``

优点：

- 实现可控。
- 直接复用 PG optimizer。
- 快速出产品。

缺点：

- 复杂路径、variable length path、shortest path 不好做。
- 代价模型不懂图。
- 生成 SQL 可能很大。
- 递归查询性能有限。

**中级实现：转换成 PG Query tree / Plan node**

AGE 更接近这一类。它不是简单暴露函数，而是有 parser、transform、planner、executor 层，把 Cypher 转成 PG 能执行的结构。官方架构说明就是 parsing -> transform -> planner/optimizer -> executor。([age.apache.org](https://age.apache.org/overview/))

优点：

- 和 PG 集成更深。
- 可以逐步加入图操作节点。
- 可维护性好于字符串拼 SQL。

缺点：

- 要理解 PG 内核结构。
- PG 版本适配成本高。
- hook 和内部 API 稳定性要谨慎。

**高级实现：Custom Scan / 自定义执行节点**

PostgreSQL 支持 custom scan provider。官方文档说明 custom scan provider 可以在 planner 阶段生成自定义 scan path，planner 选中后转成 plan，并在 executor 阶段执行。([postgresql.org](https://www.postgresql.org/docs/17/custom-scan.html?utm_source=openai))

这条路适合实现：

- graph expand operator。
- variable length traversal。
- adjacency index scan。
- path enumeration。
- shortest path operator。
- GPU/向量化/并行图扫描。
- 外部图索引访问。

优点：

- 性能上限更高。
- 可以绕开纯 join 的限制。
- 有机会做真正图感知 cost model。

缺点：

- 工程难度显著提高。
- PG 版本兼容成本更高。
- 调试、崩溃恢复、内存上下文、并发语义都要熟悉。
- 仍受 PG executor 框架约束。

我的判断：如果是商业产品，第一版不应该直接上 full custom executor；但如果长期要和原生图数据库竞争，多跳 traversal 必须走这条路或外部图引擎路线。

**四、技术路径选择建议**

可以按阶段选。

**阶段 1：MVP，做 AGE 式 PG extension**

目标：

- 支持属性图。
- 支持点边 label。
- 支持基础 Cypher 或自定义 SQL API。
- 支持 1 到 2 跳查询。
- 支持 SQL join 图查询结果。
- 支持导入、索引、可视化基础能力。

架构：

`​``text
PostgreSQL extension
  ├── catalog: graph / label metadata
  ├── storage: graph schema + vertex/edge label tables
  ├── type: graph value type, e.g. agtype-like
  ├── API: graph_query() / cypher()
  ├── parser: Cypher subset
  ├── compiler: Cypher -> SQL / Query tree
  └── index: id, start_id, end_id, property expression indexes
`​``

这个阶段不要追求完整 Cypher。先把常见模式做好：

`​``cypher
CREATE
MATCH
WHERE
RETURN
LIMIT
ORDER BY
1-hop / 2-hop
variable length limited support
`​``

产品价值聚焦：

- “在 PG 里直接查关系网络”。
- “不用引入单独 Neo4j/Nebula 集群”。
- “关系数据和图数据事务一致”。
- “SQL + 图混合查询”。

**阶段 2：增强查询优化**

补齐：

- label/table 统计信息。
- 度分布统计。
- 属性选择率统计。
- 起点选择优化。
- pattern reorder。
- predicate pushdown。
- edge direction pruning。
- variable-length path 剪枝。
- prepared statement 缓存。
- 图查询 explain。

这阶段的关键不是多支持几个 Cypher 语法，而是让常见查询稳定快。

例如同一个 pattern：

`​``cypher
MATCH (a:Account)-[:TRANSFER]->(b:Account)
WHERE b.risk_score > 90
RETURN a, b
`​``

优化器应该知道从高选择率的 `b.risk_score` 开始，还是从 `TRANSFER.start_id` 开始，而不是机械按语法顺序执行。

**阶段 3：图执行算子与邻接加速**

当用户开始要求复杂路径查询时，需要考虑：

`​``text
GraphExpand
VariableLengthExpand
PathFilter
ShortestPath
NeighborIndexScan
BidirectionalBFS
`​``

可以通过 PG custom scan 或自定义 executor 节点实现。PG custom scan 的设计本来就是允许扩展在 planner/executor 中加入自定义扫描逻辑。([postgresql.org](https://www.postgresql.org/docs/17/custom-scan.html?utm_source=openai))

此时存储也要增强：

`​``text
edge table: authoritative
adjacency table/index: traversal acceleration
optional memory cache: hot graph
`​``

这一阶段要明确一致性语义：

- adjacency index 是同步更新还是异步？
- 查询读的是事务快照还是近实时索引？
- 出错后如何 rebuild？
- bulk load 时如何处理索引？

**阶段 4：图算法与 AI 场景**

如果面向 AI 时代的 GraphRAG、知识图谱、风控、推荐，可以继续增加：

- 实体解析 entity resolution。
- 文档 chunk 到 entity/relationship 的抽取。
- 图 + 向量混合检索。
- k-hop context expansion。
- path ranking。
- relation evidence。
- temporal graph。
- 权限过滤。
- 图算法物化结果。

但这里建议不要把所有算法都塞进 PG executor。更合理的是：

`​``text
PG graph extension: authoritative graph + online query
Graph analytics worker: async algorithm / materialization
Vector DB or pgvector: semantic retrieval
Application layer: GraphRAG orchestration
`​``

**五、AGE 路线的优缺点**

**优点**

1. **PG 用户迁移成本低**

   业务数据已经在 PG，不需要单独部署图数据库。

2. **事务一致性强**

   点边和关系表可以在同一个事务里操作。

3. **SQL/Cypher 混合能力好**

   AGE 文档展示了 Cypher 可以和 CTE、JOIN、`IN`、`EXISTS`、多 graph 查询结合。([age.apache.org](https://age.apache.org/age-manual/master/advanced/advanced.html))

4. **复用 PG 生态**

   WAL、备份、权限、连接池、监控、索引、复制、客户端协议都能复用。

5. **适合中小规模在线图查询**

   特别是业务关系网络、权限关系、客户关系、组织关系、风控关系、知识图谱基础查询。

**缺点**

1. **深度遍历性能不是原生图引擎级别**

   关系表 join 对 1 到 3 跳可以接受，但复杂可变长路径会很难。

2. **优化器图感知不足**

   PG 原生优化器擅长关系代价估算，但图查询需要度分布、路径选择率、起点选择等特殊统计。

3. **动态属性影响性能和治理**

   `agtype`/JSONB 灵活，但类型约束、统计、索引、冷热字段管理都要额外做。

4. **Cypher 作为函数参数有边界**

   它不是 PG SQL grammar 的原生组成部分，和 SQL 的融合会有表达限制。

5. **PG 版本兼容成本**

   扩展越深入 parser/planner/executor，越容易受 PG 内部 API 变化影响。AGE README 显示它需要按 PG 版本适配，当前 README 提到支持 PostgreSQL 11 到 18，并在 roadmap 中持续支持新版本。([github.com](https://github.com/apache/age))

**六、我会怎么选技术路线**

如果你的目标是做一家新公司或新产品，我建议分两条线判断。

**如果目标是“PG 生态里的图能力”**

选择：

`​``text
AGE-like extension + 更好的垂直场景优化
`​``

定位不要说“我们是通用图数据库替代 Neo4j/Nebula”，而应说：

`​``text
面向 PostgreSQL 用户的事务型图查询扩展，
让已有业务数据在不搬库的情况下获得图建模、路径查询和 GraphRAG 能力。
`​``

技术路线：

1. PG extension。
2. 属性图模型。
3. graph schema + label tables。
4. `agtype`/JSONB-like 图值类型。
5. Cypher subset 或 SQL/PGQ-like DSL。
6. SQL 混合查询作为核心卖点。
7. 热属性列化 + JSONB/agtype 冷属性。
8. `start_id/end_id` 索引。
9. 2-hop/3-hop 查询重点优化。
10. GraphRAG、风控、IAM、主数据管理场景模板。

突破点：

- 比 AGE 更易用。
- 更好的安装和云 PG 兼容。
- 更好的可视化和调试。
- 更好的属性索引建议。
- 更好的 explain/analyze。
- 更好的 GraphRAG 集成。
- 更好的企业权限和审计。
- 针对金融风控、供应链、工业资产、企业知识库做场景包。

**如果目标是“高性能原生图数据库”**

不要完全绑在 PG 内部。选择：

`​``text
PG connector + 外部图存储/图计算内核
`​``

PG 负责：

- 元数据。
- 事务入口。
- 关系数据同步。
- SQL 联邦查询。
- 权限和治理。

图引擎负责：

- adjacency storage。
- 多跳遍历。
- 图算法。
- 分布式扩展。
- 大规模路径查询。

这条路能做大规模图，但工程和运维成本远高于 PG 插件。

**如果目标是“AI 时代 GraphRAG/知识图谱基础设施”**

我会选折中路线：

`​``text
PostgreSQL-native graph + vector + document + evidence path
`​``

核心不是做最强图数据库，而是做：

- 文档、实体、关系、向量、路径的统一存储。
- 从文本抽取实体关系。
- 图谱增量更新。
- 多跳上下文扩展。
- 引用证据链。
- 权限过滤。
- RAG 查询解释。
- 和现有 PG 数据 join。

这类场景对 PG 内实现非常友好，因为企业知识大多已经存在关系库、文档库、权限系统里。图只是把关系显式化，并服务检索和推理。

**七、一个推荐的 PG 内图数据库架构蓝图**

可以设计成这样：

`​``text
Client / Driver
  ├── SQL
  ├── Cypher subset
  └── GraphRAG API

PostgreSQL Extension
  ├── SQL objects
  │     ├── create_graph()
  │     ├── create_vlabel()
  │     ├── create_elabel()
  │     └── graph_query()/cypher()
  │
  ├── Catalog
  │     ├── graph catalog
  │     ├── label catalog
  │     ├── property schema catalog
  │     └── statistics catalog
  │
  ├── Type System
  │     ├── graphid
  │     ├── vertex
  │     ├── edge
  │     ├── path
  │     └── graph value / agtype-like
  │
  ├── Storage Mapping
  │     ├── graph schema
  │     ├── vertex label tables
  │     ├── edge label tables
  │     ├── adjacency helper tables
  │     └── property indexes
  │
  ├── Query Compiler
  │     ├── parser
  │     ├── semantic analyzer
  │     ├── rewrite
  │     ├── SQL/Query tree generation
  │     └── predicate pushdown
  │
  ├── Planner
  │     ├── pattern reorder
  │     ├── degree statistics
  │     ├── label selectivity
  │     ├── property selectivity
  │     └── path cost model
  │
  ├── Executor
  │     ├── PG join-based execution
  │     ├── graph expand operator
  │     ├── variable length expand
  │     ├── shortest path
  │     └── path materialization
  │
  └── Tooling
        ├── import/export
        ├── visualization
        ├── explain graph query
        ├── index advisor
        └── graph statistics refresh
`​``

**八、最终判断**

AGE 代表的是一条非常清晰的路线：**在 PostgreSQL 内做图数据库，优先复用 PG，而不是重造图存储引擎**。

这条路线的正确打开方式是：

`​``text
把 PG 当成事务型多模型数据库底座，
把图能力做成 extension，
用 label table + edge start/end index 解决基础图存储，
用 Cypher/SQL 混合查询解决开发体验，
再逐步补 graph-aware optimizer、adjacency 加速和图算法。
`​``

不建议一开始就做：

- 完整 Cypher。
- 分布式图数据库。
- 自研存储引擎。
- 全量图算法。
- fork PostgreSQL。
- 复杂 custom executor。

建议一开始做：

- AGE 式架构。
- 更好的安装部署。
- 更好的 PG 云环境兼容。
- 热点场景优化。
- 图 + SQL + 向量 + 文档的混合查询。
- 面向 AI/知识图谱/风控/IAM 的产品化封装。

一句话总结：

> 如果你想在 PG 里实现图数据库，AGE 给出的核心启发是：第一阶段不要挑战 PG 内核边界，而是把图模型、Cypher 编译、点边表映射、`agtype` 类型和 SQL 混合查询做好；第二阶段再用图统计、邻接索引和 custom scan/executor 去突破多跳遍历性能上限。

<!-- /message -->
<!-- message:id=msg_edfeda7b9cc5caaf role=user createdAt=2026-06-01T00%3A00%3A26.807Z -->

## User

age删点的时候会删除边吗？快答

<!-- /message -->
<!-- message:id=msg_7adc3641d6bc1d13 role=assistant createdAt=2026-06-01T00%3A00%3A43.487Z runId=run_dc4fe64d8d3ac4a5 status=succeeded -->

## Assistant

不会自动级联，分两种：

- `DELETE v`：只能删除**没有任何关联边**的点；如果这个点还有入边或出边，AGE 会报错，不能删。
- `DETACH DELETE v`：会先删除这个点关联的边，再删除点。

示例：

`​``sql
-- 只删孤立点；有边会失败
SELECT *
FROM cypher('graph_name', $$
  MATCH (v:Person {id: 1})
  DELETE v
$$) AS (v agtype);
`​``

`​``sql
-- 删点，并删除所有关联边
SELECT *
FROM cypher('graph_name', $$
  MATCH (v:Person {id: 1})
  DETACH DELETE v
$$) AS (v agtype);
`​``

所以答案是：**普通删点不会自动删边；要用 `DETACH DELETE` 才会删关联边。**  
官方文档也明确说不能删除仍有关联边的点，除非显式删边或使用 `DETACH DELETE`：https://age.apache.org/age-manual/master/clauses/delete.html

<!-- /message -->
<!-- message:id=msg_d24d8163e82c75e9 role=user createdAt=2026-06-01T00%3A17%3A30.447Z -->

## User

| 知识图谱 | 实体关系、语义、推理、RAG | schema、本体、实体解析、检索 |
| AI/RAG 增强 | GraphRAG、关系约束、上下文扩展 | 图 + 向量 + 文档混合检索 |，面向这两种场景，考虑架构设计

<!-- /message -->
<!-- message:id=msg_960ef13599323ab8 role=assistant createdAt=2026-06-01T00%3A20%3A00.904Z runId=run_ebe0e679dcd4458a status=succeeded -->

## Assistant

面向这两类场景，架构设计不能只说“加一个图数据库”。更准确地说，应设计成一个 **知识层 + 检索层 + 推理层 + 证据层** 的混合系统。

我建议把它理解成：

`​``text
知识图谱场景：
  重点是实体、关系、schema、本体、质量、推理、治理。

AI/RAG 增强场景：
  重点是文档、chunk、向量、图关系、上下文扩展、证据链、答案生成。

共同底座：
  图 + 向量 + 文档 + SQL + 权限 + 证据溯源。
`​``

如果以 PostgreSQL / AGE / pgvector 作为底座，可以设计成一个 **PostgreSQL-native Knowledge Graph + GraphRAG 平台**。

---

**一、两类场景的核心差异**

先区分目标，否则架构会混。

| 场景 | 核心问题 | 数据中心 | 查询中心 | 质量要求 |
|---|---|---|---|---|
| 知识图谱 | 世界中有哪些实体，它们是什么关系 | 实体、关系、本体、属性 | 图查询、规则推理、关系推断 | 高一致性、高治理 |
| AI/RAG 增强 | 如何让大模型回答得更准、更可解释 | 文档、chunk、向量、实体、证据 | 混合检索、路径扩展、上下文组装 | 高召回、可追溯 |

知识图谱更像“结构化知识资产管理系统”。

GraphRAG 更像“面向大模型的知识检索与上下文构造系统”。

两者可以共用图存储，但不应该完全共用同一套数据模型。知识图谱的实体关系是主数据；RAG 的 chunk、embedding、引用证据是检索数据。二者应该关联，而不是混成一张大表。

---

**二、推荐总体架构**

可以按下面的分层设计：

`​``text
                    Application / Agent / BI / Search UI
                                  |
                                  v
                      Query & Reasoning API
        -------------------------------------------------
        | Graph Query | Hybrid Search | GraphRAG | Rules |
        -------------------------------------------------
                                  |
                                  v
                         Retrieval Orchestrator
        -------------------------------------------------
        | entity lookup | vector search | full-text search |
        | path expansion | rerank       | permission filter |
        -------------------------------------------------
                                  |
                                  v
              Knowledge / Evidence / Document Storage Layer
        ---------------------------------------------------------
        | Graph Store | Vector Store | Document Store | SQL Store |
        ---------------------------------------------------------
                                  |
                                  v
                         Ingestion & Knowledge Pipeline
        ---------------------------------------------------------
        | extraction | entity resolution | relation extraction |
        | ontology mapping | validation | evidence linking     |
        ---------------------------------------------------------
                                  |
                                  v
                          Source Systems
        ---------------------------------------------------------
        | DB | files | web pages | tickets | emails | CRM | ERP |
        ---------------------------------------------------------
`​``

如果落在 PostgreSQL 体系里，可以是：

`​``text
PostgreSQL
  ├── AGE / graph extension      -- 实体、关系、路径、图查询
  ├── pgvector                   -- chunk embedding、entity embedding
  ├── JSONB / agtype             -- 半结构化属性
  ├── relational tables          -- 文档、chunk、任务、权限、版本、审计
  ├── full-text index            -- 关键词检索
  ├── btree/gin/hnsw indexes     -- 属性、JSON、向量索引
  └── background workers         -- 抽取、消歧、图谱更新、embedding 更新
`​``

核心原则是：

> 图数据库负责表达“关系”，向量数据库负责表达“语义相似”，文档库负责保存“证据原文”，关系数据库负责治理、权限、事务和业务集成。

---

**三、知识图谱场景的架构设计**

知识图谱的核心不是 RAG，而是 **高质量结构化知识**。

它至少需要这几层。

**1. 本体与 Schema 层**

这一层定义业务世界：

`​``text
EntityType:
  Person
  Company
  Product
  Project
  Contract
  RiskEvent
  Policy
  Concept
  Document

RelationType:
  WORKS_FOR
  OWNS
  SUPPLIES
  BELONGS_TO
  MENTIONS
  CONFLICTS_WITH
  DEPENDS_ON
  SIMILAR_TO
  CAUSED_BY
`​``

需要管理：

`​``text
entity_type
relation_type
property_schema
allowed_relation
cardinality
constraint
synonym
taxonomy
ontology_version
`​``

例如：

`​``text
Person -[WORKS_FOR]-> Company
Company -[SUPPLIES]-> Company
Product -[MENTIONED_IN]-> DocumentChunk
Policy -[APPLIES_TO]-> Product
RiskEvent -[AFFECTS]-> Company
`​``

如果没有这一层，图会很快变成“边和属性的垃圾场”，后面很难做推理、治理和产品化。

在 PG 里可以用普通表维护 schema：

`​``sql
kg_entity_type
kg_relation_type
kg_property_schema
kg_allowed_relation
kg_ontology_version
`​``

图里则存真正的实体和边。

---

**2. 实体层**

实体是知识图谱的主对象。

建议实体至少有：

`​``text
entity_id
entity_type
canonical_name
aliases
external_ids
properties
confidence
status
created_at
updated_at
source
tenant_id
`​``

例如：

`​``text
Entity:
  id: e_123
  type: Company
  name: "华为技术有限公司"
  aliases: ["华为", "Huawei"]
  external_ids:
    credit_code: "..."
    crm_id: "..."
  properties:
    industry: "通信设备"
    country: "CN"
`​``

实体不要只存在于文档 chunk 里。chunk 只是证据，实体应该有自己的生命周期。

关键能力是 **entity resolution**，也就是实体消歧和合并：

`​``text
"阿里"
"阿里巴巴"
"Alibaba Group"
"阿里巴巴集团控股有限公司"
`​``

这些可能指向同一个实体，也可能在不同上下文中不是同一个实体。架构上需要：

`​``text
entity_alias
entity_candidate
entity_merge_log
entity_resolution_job
entity_similarity_score
`​``

这部分对知识图谱非常关键，对 GraphRAG 也非常关键。很多 GraphRAG 失败不是因为向量检索差，而是实体没有对齐。

---

**3. 关系层**

关系边不能只是：

`​``text
A --related_to--> B
`​``

生产级知识图谱里，关系需要元数据：

`​``text
relation_id
relation_type
start_entity_id
end_entity_id
properties
confidence
valid_from
valid_to
source
evidence_id
created_by
status
`​``

尤其要保留：

`​``text
confidence       -- 这条关系有多可信
source           -- 来自哪个系统、文档、模型或人工
evidence         -- 原文证据在哪里
valid_time       -- 关系在哪个时间段有效
transaction_time -- 这条记录何时写入系统
`​``

例如：

`​``text
CompanyA -[SUPPLIES {
  product: "battery",
  confidence: 0.92,
  source: "annual_report_2025",
  evidence_chunk_id: "chunk_778",
  valid_from: "2024-01-01"
}]-> CompanyB
`​``

对于知识图谱，**边的证据和置信度比边本身更重要**。

---

**4. 推理层**

知识图谱的推理可以分三档。

第一档是规则推理：

`​``text
A WORKS_FOR B
B SUBSIDIARY_OF C
=> A ASSOCIATED_WITH C
`​``

第二档是图算法：

`​``text
共同邻居
最短路径
影响传播
PageRank
社区发现
相似实体
风险扩散
`​``

第三档是 LLM 辅助推理：

`​``text
从证据中判断关系是否成立
解释路径为什么重要
生成自然语言摘要
提出可能的新关系
`​``

建议不要一开始就做复杂 OWL/RDF 推理。对于企业场景，最实用的是：

`​``text
规则引擎 + Cypher/SQL 图查询 + 图算法物化结果 + LLM 解释
`​``

架构上可以有：

`​``text
kg_rule
kg_inference_job
kg_inferred_relation
kg_algorithm_result
kg_reasoning_trace
`​``

推理生成的关系必须和人工/来源关系区分开：

`​``text
source = "inferred"
confidence = 0.74
derived_from = [relation_1, relation_2, relation_3]
`​``

否则图会被模型生成内容污染。

---

**四、AI/RAG 增强场景的架构设计**

GraphRAG 的目标不是“存图”，而是让大模型回答时能拿到更好的上下文。

传统 RAG 是：

`​``text
query -> vector search -> chunks -> LLM
`​``

GraphRAG 应该是：

`​``text
query
  -> query understanding
  -> entity linking
  -> vector / full-text / graph mixed retrieval
  -> graph expansion
  -> evidence ranking
  -> context assembly
  -> LLM answer with citations
`​``

---

**1. 文档与 Chunk 层**

文档仍然是 RAG 的证据源。

建议保留独立文档模型：

`​``text
document
  id
  title
  source_uri
  source_type
  author
  created_at
  updated_at
  permissions
  metadata
  checksum
  version

document_chunk
  id
  document_id
  chunk_index
  text
  token_count
  section_path
  page_no
  embedding
  metadata
`​``

不要把 chunk 只做成图节点，也不要只做向量记录。它需要同时服务：

`​``text
向量检索
全文检索
权限过滤
引用定位
版本管理
证据展示
`​``

因此 chunk 最好用普通 PG 表存，embedding 用 pgvector，必要时把 chunk 也映射成图节点。

---

**2. Chunk 与实体关系**

GraphRAG 的关键表/边是：

`​``text
chunk_mentions_entity
chunk_mentions_relation
entity_supported_by_chunk
relation_supported_by_chunk
`​``

也可以在图里表达成：

`​``text
(Document)-[:HAS_CHUNK]->(Chunk)
(Chunk)-[:MENTIONS]->(Entity)
(Chunk)-[:EVIDENCE_FOR]->(Relation)
(Entity)-[:RELATED_TO]->(Entity)
`​``

但在实现上，我更建议混合：

`​``text
document / chunk: relational + vector
entity / relation: graph
evidence mapping: relational table + optional graph edge
`​``

原因是 chunk 数量通常远大于实体数量。所有 chunk 全部进入图，会导致图膨胀，遍历效率和治理复杂度都会上升。

比较稳的设计是：

`​``text
图中主要放：
  Entity
  Relation
  Document
  高价值 Chunk 或 Section

关系表中放：
  chunk_entity_mentions
  chunk_relation_evidence
  chunk_embeddings
`​``

---

**3. 检索层**

GraphRAG 不应该只依赖一种检索。

推荐做四路检索：

`​``text
1. 向量检索
   query embedding -> similar chunks

2. 全文检索
   keyword / BM25 / tsquery -> exact-match chunks

3. 实体检索
   query -> entity linker -> matched entities

4. 图扩展
   matched entities -> k-hop neighbors / paths / communities
`​``

然后做融合排序：

`​``text
final_score =
  vector_score * w1
  + text_score * w2
  + graph_score * w3
  + authority_score * w4
  + freshness_score * w5
  + permission_score/filter
`​``

图扩展不能无限扩。必须有限制：

`​``text
max_hops
allowed_relation_types
blocked_relation_types
max_neighbors_per_node
edge_confidence_threshold
time_window
tenant_id
permission_scope
`​``

否则 GraphRAG 会把大量弱相关上下文塞给 LLM。

---

**4. 上下文组装层**

GraphRAG 真正的产品差异点在这里。

不是简单把图路径和 chunk 拼起来，而是生成结构化上下文：

`​``text
Question:
  用户问题

Matched Entities:
  命中的实体

Relevant Facts:
  从图谱中查到的事实

Evidence Chunks:
  支撑这些事实的原文片段

Paths:
  关键实体之间的关系路径

Conflicts:
  互相冲突的信息

Answer Constraints:
  只能基于证据回答
  引用来源
  不确定时说明不确定
`​``

上下文可以这样组织：

`​``text
Entity Context:
  - entity_id
  - canonical_name
  - type
  - key_properties

Graph Facts:
  - subject
  - predicate
  - object
  - confidence
  - evidence_chunk_id

Evidence:
  - chunk_id
  - document_title
  - section
  - text
  - source_uri

Reasoning Paths:
  - A -> relation -> B -> relation -> C
  - path_score
  - why_selected
`​``

这比直接塞一堆 chunk 更可控，也更容易解释。

---

**五、面向两类场景的统一数据模型**

建议分成五类对象：

`​``text
1. Ontology objects
   EntityType
   RelationType
   PropertySchema
   Constraint

2. Knowledge objects
   Entity
   Relation
   Attribute
   InferredFact

3. Evidence objects
   Document
   Chunk
   Source
   Citation

4. Retrieval objects
   Embedding
   FullTextIndex
   AliasIndex
   EntityLink

5. Governance objects
   Tenant
   Permission
   Version
   AuditLog
   Confidence
   Provenance
`​``

一个简化模型：

`​``text
Entity
  id
  type
  canonical_name
  aliases
  properties
  confidence
  tenant_id

Relation
  id
  type
  start_entity_id
  end_entity_id
  properties
  confidence
  valid_from
  valid_to
  tenant_id

Document
  id
  title
  source_uri
  source_type
  metadata
  permission_policy
  version

Chunk
  id
  document_id
  text
  embedding
  section_path
  page_no

EntityMention
  chunk_id
  entity_id
  mention_text
  offset_start
  offset_end
  confidence

RelationEvidence
  relation_id
  chunk_id
  evidence_text
  confidence
  extractor
`​``

图结构可以是：

`​``text
(Entity)-[RELATION]->(Entity)
(Document)-[:HAS_CHUNK]->(Chunk)
(Chunk)-[:MENTIONS]->(Entity)
(Chunk)-[:EVIDENCE_FOR]->(Relation)
(Entity)-[:ALIAS_OF]->(Entity)
(Entity)-[:SAME_AS]->(Entity)
(Entity)-[:PART_OF]->(Entity)
`​``

但物理实现不一定都存在图表里。高频检索对象适合放关系表和索引里；高价值关系适合放图里。

---

**六、如果基于 PostgreSQL / AGE 设计**

可以这样落地。

**1. PostgreSQL 表**

`​``text
kg_entity
kg_relation
kg_entity_alias
kg_entity_type
kg_relation_type
kg_property_schema
kg_document
kg_document_chunk
kg_chunk_embedding
kg_chunk_entity_mention
kg_relation_evidence
kg_extraction_job
kg_inference_job
kg_audit_log
`​``

**2. AGE 图**

`​``text
Graph: kg_graph

Vertex labels:
  Entity
  Document
  Chunk
  Concept
  Policy
  Event

Edge labels:
  RELATES_TO
  MENTIONS
  EVIDENCE_FOR
  PART_OF
  SAME_AS
  DEPENDS_ON
  CONFLICTS_WITH
  DERIVED_FROM
`​``

**3. pgvector**

`​``text
document_chunk.embedding
entity.embedding
relation.embedding
community_summary.embedding
`​``

向量不只给 chunk 用。实体、关系、社区摘要也可以有 embedding。

例如：

`​``text
entity embedding:
  由实体名称、别名、描述、关键属性生成

relation embedding:
  由关系类型、两端实体、证据摘要生成

community embedding:
  由某个子图摘要生成
`​``

这对 GraphRAG 很有用。

---

**七、查询流程设计**

一个典型 GraphRAG 查询可以这样走：

`​``text
用户问题：
  “A 公司和 B 公司的供应链风险有什么关系？”

Step 1: Query Understanding
  - 识别意图：风险分析
  - 识别实体：A 公司、B 公司
  - 识别关系类型：供应、投资、诉讼、制裁、上下游

Step 2: Entity Linking
  - A 公司 -> entity_id = e_100
  - B 公司 -> entity_id = e_200

Step 3: Hybrid Retrieval
  - 向量搜索相关 chunk
  - 全文搜索 A/B 公司、供应链、风险
  - 图查询 A 和 B 之间的路径
  - 查找共同供应商、客户、风险事件

Step 4: Graph Expansion
  - 限制 2-3 跳
  - 只允许 SUPPLIES / OWNS / RISK_EVENT / SANCTION / LITIGATION 等边
  - 过滤低置信度边
  - 过滤无权限证据

Step 5: Evidence Ranking
  - 关系路径得分
  - 证据 chunk 得分
  - 来源可信度
  - 时间新鲜度

Step 6: Context Assembly
  - 关键实体
  - 关键关系
  - 证据原文
  - 风险路径
  - 冲突信息

Step 7: LLM Answer
  - 基于证据回答
  - 给出引用
  - 给出不确定性
`​``

这就是 GraphRAG 和普通 RAG 的本质区别：**它不是只找相似文本，而是先把问题落到实体和关系上，再用图控制上下文扩展。**

---

**八、图查询与向量查询如何协同**

有三种模式。

**模式 1：Vector First**

`​``text
query -> vector search chunks -> extract entities -> graph expansion
`​``

适合：

`​``text
开放问题
语义模糊问题
不知道具体实体的问题
`​``

例如：

`​``text
“最近有哪些政策可能影响新能源供应链？”
`​``

先用向量找相关文档，再从文档中的实体进入图。

---

**模式 2：Graph First**

`​``text
query -> entity linking -> graph traversal -> evidence chunks
`​``

适合：

`​``text
明确实体问题
关系路径问题
风险传播问题
权限关系问题
`​``

例如：

`​``text
“A 公司和 B 公司之间有什么关系？”
“某员工为什么能访问这个系统？”
“这个供应商的二级风险来自哪里？”
`​``

先图查询，再找证据 chunk。

---

**模式 3：Hybrid Parallel**

`​``text
query
  -> vector search
  -> full-text search
  -> entity lookup
  -> graph traversal
  -> fusion rerank
`​``

适合生产系统默认模式。

它比单一路径更稳，但成本更高，需要缓存和限流。

---

**九、推理与 RAG 的边界**

一定要区分：

`​``text
已知事实
推理事实
模型猜测
`​``

建议分三层输出：

`​``text
Fact:
  图谱中明确存在，且有证据支撑。

Inference:
  由规则或路径推导出来，有 derived_from。

Hypothesis:
  LLM 根据证据提出的可能解释，不能直接写回主图。
`​``

例如：

`​``text
事实：
  A 公司供应电池给 B 公司。

推理：
  如果 B 公司依赖 A 公司，且 A 公司存在制裁风险，
  则 B 公司存在供应链风险暴露。

假设：
  该风险可能影响 B 公司的欧洲业务。
`​``

假设不能直接进入主知识图谱。可以进入候选事实表，等待人工确认或后续证据验证：

`​``text
kg_candidate_fact
kg_human_review_task
`​``

这是企业级 GraphRAG 很重要的安全边界。

---

**十、索引设计**

面向知识图谱，需要：

`​``text
entity(type, canonical_name)
entity_alias(alias)
relation(start_entity_id, relation_type)
relation(end_entity_id, relation_type)
relation(relation_type, confidence)
relation(valid_from, valid_to)
relation_evidence(relation_id)
`​``

面向 RAG，需要：

`​``text
document_chunk embedding HNSW index
document_chunk full-text index
chunk_entity_mention(entity_id)
chunk_entity_mention(chunk_id)
relation_evidence(chunk_id)
document permission index
`​``

面向图遍历，需要：

`​``text
edge(start_id, edge_type, end_id)
edge(end_id, edge_type, start_id)
edge(confidence)
edge(valid_time)
`​``

如果基于 AGE，至少要关注：

`​``text
边的 start_id / end_id 索引
label 表分布
属性表达式索引
JSONB/agtype 属性索引
高频属性列化
`​``

GraphRAG 里最常见的性能问题不是向量检索，而是：

`​``text
命中实体后，k-hop 扩展爆炸。
`​``

所以图扩展必须有：

`​``text
度数限制
边类型白名单
置信度阈值
时间过滤
租户过滤
权限过滤
路径数量上限
`​``

---

**十一、权限与多租户设计**

这两类场景都不能忽视权限。

尤其是企业知识库，不能出现：

`​``text
用户没有权限看原文，
但 RAG 答案泄露了原文内容。
`​``

权限要贯穿全链路：

`​``text
document permission
chunk permission
entity permission
relation permission
graph traversal permission
retrieval result permission
LLM context permission
citation permission
`​``

推荐设计：

`​``text
所有检索和图扩展之前，先确定 user_scope。

向量检索：
  只检索用户有权限的 chunk。

图扩展：
  只遍历用户有权限的实体和关系。

上下文组装：
  只放入用户有权限的证据。

答案生成：
  引用必须来自可访问证据。
`​``

可以把权限做成：

`​``text
tenant_id
acl_policy_id
security_label
visibility_level
source_system_permission
`​``

图里也要有权限过滤，否则图路径可能泄露敏感关系。

---

**十二、数据更新与质量闭环**

知识图谱和 GraphRAG 都不是一次性导入。

需要流水线：

`​``text
Source Change
  -> ingest
  -> parse
  -> chunk
  -> embedding
  -> entity extraction
  -> entity resolution
  -> relation extraction
  -> ontology mapping
  -> validation
  -> graph update
  -> index refresh
  -> evaluation
`​``

关键是增量更新：

`​``text
文档变了，哪些 chunk 失效？
chunk 变了，哪些 entity mention 失效？
entity 合并了，哪些关系要迁移？
关系删除了，哪些推理结果要撤销？
embedding 变了，哪些缓存要刷新？
`​``

所以需要：

`​``text
version
lineage
job_id
source_checksum
valid_from / valid_to
is_deleted
supersedes_id
`​``

不要只做 append-only，也不要直接覆盖不留痕。知识图谱非常需要版本和溯源。

---

**十三、面向产品的模块拆分**

如果要做成产品，我建议拆成这些模块：

`​``text
1. Knowledge Modeler
   管理实体类型、关系类型、本体、约束。

2. Ingestion Pipeline
   接入数据库、文档、API、网页、对象存储。

3. Extraction Engine
   做实体抽取、关系抽取、属性抽取、摘要。

4. Entity Resolution Engine
   做别名、合并、去重、冲突处理。

5. Graph Store
   存实体、关系、路径、图查询。

6. Evidence Store
   存文档、chunk、引用、来源、版本。

7. Vector Retrieval
   存 embedding，做语义检索。

8. GraphRAG Orchestrator
   做查询理解、混合检索、图扩展、上下文组装。

9. Reasoning Engine
   做规则推理、路径解释、风险传播、候选事实生成。

10. Governance Console
   做权限、审计、质量评分、人工审核、血缘追踪。
`​``

早期不需要全做。最小可行版本可以是：

`​``text
文档导入
chunk + embedding
实体抽取
实体消歧
实体关系图
chunk-entity evidence linking
混合检索
GraphRAG answer with citation
`​``

---

**十四、推荐的 MVP 架构**

如果你现在要做第一版，我建议不要一上来做完整知识图谱平台，而是做一个窄但完整的闭环：

`​``text
PostgreSQL + pgvector + AGE-like graph
`​``

第一版支持：

`​``text
1. 导入文档和结构化表
2. 文档切 chunk
3. 生成 chunk embedding
4. 抽取实体和关系
5. 做基础实体合并
6. 把实体和关系写入图
7. 建立 chunk -> entity / relation evidence
8. 支持实体搜索
9. 支持 1-3 跳图扩展
10. 支持 GraphRAG 问答和引用
`​``

暂时不要做：

`​``text
复杂本体编辑器
完整 Cypher 兼容
复杂 OWL 推理
分布式图计算
全自动知识图谱构建
大规模图算法平台
`​``

MVP 的核心演示应该是：

`​``text
用户问一个问题
  -> 系统识别实体
  -> 找到相关文档
  -> 找到实体关系路径
  -> 给出答案
  -> 展示支撑答案的原文证据
  -> 展示实体关系路径
`​``

这个闭环最能体现 GraphRAG 的价值。

---

**十五、以 AGE / PG 为底座的具体技术路线**

可以分阶段。

**阶段 1：PG-native GraphRAG**

`​``text
PostgreSQL:
  document / chunk / evidence / job / permission

pgvector:
  chunk embedding

AGE 或自研图扩展:
  entity / relation graph

应用层:
  extraction pipeline
  entity resolution
  retrieval orchestration
  LLM answer generation
`​``

这一阶段重点是产品闭环，不是图数据库内核性能。

---

**阶段 2：知识图谱治理增强**

增加：

`​``text
ontology schema
entity merge workflow
relation validation
confidence score
human review
source lineage
versioning
conflict detection
`​``

这一阶段从“RAG 工具”变成“知识资产平台”。

---

**阶段 3：图查询和推理增强**

增加：

`​``text
路径查询优化
关系类型白名单
路径打分
规则推理
风险传播
图算法物化
community summary
entity neighborhood summary
`​``

GraphRAG 性能和效果会明显提升。

---

**阶段 4：高性能图加速**

如果数据规模上来，再考虑：

`​``text
邻接索引
图缓存
预计算 k-hop neighborhood
图摘要
异步图算法 worker
外部图计算引擎
`​``

不要第一天就做这个。早期真正的难点是数据质量、实体消歧和证据链，不是图遍历引擎。

---

**十六、关键架构取舍**

**1. 文档 chunk 要不要作为图节点？**

我的建议：

`​``text
小规模可以放图里。
中大规模不要所有 chunk 都进图。
`​``

更稳的方式：

`​``text
Document / Chunk 用关系表 + 向量索引。
Entity / Relation 用图。
Chunk 与 Entity/Relation 的映射用关系表。
高价值 chunk 或 section 可以进入图。
`​``

原因是 chunk 数量太大，全部进图会让图变重。

---

**2. 用 RDF 还是 Property Graph？**

如果主要面向企业应用、RAG、业务实体关系：

`​``text
优先 Property Graph。
`​``

原因：

`​``text
属性图更贴近业务建模
Cypher/图遍历更直观
和 PG/AGE 更匹配
工程实现更简单
`​``

如果强依赖标准本体、语义网、SPARQL、OWL 推理：

`​``text
考虑 RDF 或 RDF + Property Graph 双模型。
`​``

但早期不要双模型。会增加很多复杂度。

---

**3. 图和向量谁是主？**

知识图谱场景：

`​``text
图是主，向量是辅助检索。
`​``

GraphRAG 场景：

`​``text
文档和证据是主，图和向量共同服务检索。
`​``

企业产品里，最稳的是：

`​``text
关系数据库作为治理主库
图作为关系索引
向量作为语义索引
文档作为证据源
`​``

---

**4. LLM 抽取结果能不能直接进图？**

不建议直接进主图。

推荐流程：

`​``text
LLM extraction
  -> candidate entity / candidate relation
  -> confidence scoring
  -> rule validation
  -> dedup / entity resolution
  -> human review if needed
  -> promoted to graph
`​``

高置信自动入图，低置信进入审核队列。

---

**十七、最终推荐架构一句话**

面向“知识图谱 + AI/RAG 增强”，我建议的架构是：

`​``text
以 PostgreSQL 为治理与事务底座，
用 AGE/属性图表达实体关系，
用 pgvector 存 chunk/entity/relation embedding，
用关系表管理文档、chunk、证据、权限、版本和任务，
在应用层构建实体抽取、实体消歧、关系抽取、混合检索、图扩展、上下文组装和推理链路。
`​``

核心不是单独做一个图数据库，而是做一个：

`​``text
Knowledge Graph + Evidence Store + Vector Retrieval + GraphRAG Orchestrator
`​``

的组合系统。

如果要找产品定位，可以这样定：

> 面向企业知识和 AI 应用的 PostgreSQL-native 知识图谱与 GraphRAG 基础设施：把结构化数据、文档、实体关系、向量检索和证据链统一在一个可治理、可追溯、可权限控制的知识底座里。

这比单纯说“我们做图数据库”更清晰，也更符合 AI 时代的增量价值。

<!-- /message -->
<!-- message:id=msg_97d7568be8bf34e4 role=user createdAt=2026-06-01T05%3A30%3A58.143Z -->

## User

如果限制到图数据库来支撑这两个场景，应该如何考虑数据库的系统架构设计？可以先给出设计思路。快答

<!-- /message -->
<!-- message:id=msg_d785b4d43bed9451 role=assistant createdAt=2026-06-01T05%3A32%3A02.138Z runId=run_76f19625e8c3a901 status=succeeded -->

## Assistant

限制在“图数据库本身”来支撑 **知识图谱** 和 **AI/RAG 增强**，设计重点应从“单纯存点边”升级成：

> 一个支持实体、关系、证据、语义检索、路径推理和上下文扩展的图数据库内核。

快答版设计思路如下。

**1. 先明确图数据库的定位**

不要把图数据库只设计成：

`​``text
Vertex + Edge + Property + Cypher
`​``

而应设计成：

`​``text
知识图谱场景：
  图数据库 = 实体关系事实库 + schema/ontology + provenance + inference

AI/RAG 场景：
  图数据库 = 关系检索引擎 + evidence graph + context expansion engine
`​``

也就是说，图数据库不仅要回答：

`​``text
A 和 B 有什么关系？
`​``

还要回答：

`​``text
这条关系来自哪里？
可信度是多少？
什么时候有效？
能否作为 RAG 答案证据？
从一个实体扩展上下文时，应该走哪些边、不走哪些边？
`​``

---

**2. 总体系统架构**

可以按六层设计：

`​``text
Query/API Layer
  Cypher / GQL / GraphRAG API / KG API

Query Planning Layer
  pattern matching / path search / graph + vector hybrid planning / permission filter

Execution Layer
  graph traversal / shortest path / k-hop expansion / rule inference / context subgraph extraction

Storage Layer
  vertex store / edge store / property store / evidence store / embedding store / schema catalog

Index Layer
  adjacency index / property index / text index / vector index / temporal index / provenance index

Governance Layer
  ontology / schema / lineage / confidence / version / ACL / audit
`​``

传统图数据库重点在前四层。面向知识图谱和 GraphRAG，必须把 **Index Layer** 和 **Governance Layer** 做重。

---

**3. 数据模型要从属性图扩展成知识图谱属性图**

基础模型：

`​``text
Vertex:
  id
  labels
  properties

Edge:
  id
  type
  start_id
  end_id
  properties
`​``

需要增强为：

`​``text
Entity Vertex:
  entity_id
  entity_type
  canonical_name
  aliases
  external_ids
  properties
  confidence
  source
  tenant_id
  version

Relation Edge:
  relation_id
  relation_type
  start_entity_id
  end_entity_id
  properties
  confidence
  evidence_ids
  valid_from
  valid_to
  source
  status
`​``

关键是边必须带：

`​``text
confidence      -- 置信度
evidence        -- 证据来源
valid_time      -- 业务有效时间
source          -- 来自文档、系统、人工还是模型
version         -- 版本
tenant/security -- 权限范围
`​``

否则它只能算“图存储”，很难支撑知识图谱和 RAG。

---

**4. 内核存储设计**

图数据库底层建议不要只做一种存储。

可以拆成：

`​``text
1. Entity Store
   存实体点、类型、属性、别名、外部 ID。

2. Relation Store
   存边、方向、类型、属性、置信度、时间。

3. Adjacency Store
   面向遍历优化：
   out_adj(entity_id, relation_type) -> neighbors
   in_adj(entity_id, relation_type)  -> neighbors

4. Property Store
   存动态属性、热属性列化、JSON/property map。

5. Evidence Store
   存证据引用：
   relation_id -> document/chunk/source/span

6. Embedding Store
   存 entity embedding、relation embedding、subgraph embedding、chunk embedding。

7. Schema/Ontology Catalog
   存实体类型、关系类型、约束、允许的边类型、属性定义。
`​``

图数据库要支撑 RAG，最好内建向量索引，不然图和向量割裂太严重。

---

**5. 索引体系要重做**

普通图数据库常见索引：

`​``text
vertex id index
edge start_id index
edge end_id index
property index
`​``

面向这两个场景，需要增加：

`​``text
Adjacency Index:
  start_id + relation_type + end_id
  end_id + relation_type + start_id

Entity Lookup Index:
  entity_type + canonical_name
  alias -> entity_id
  external_id -> entity_id

Evidence Index:
  relation_id -> evidence
  entity_id -> mentioning chunks
  source_id -> extracted facts

Vector Index:
  entity embedding
  relation embedding
  chunk embedding
  subgraph/community embedding

Temporal Index:
  valid_from / valid_to
  event_time

Confidence Index:
  confidence threshold filtering

Permission Index:
  tenant_id
  acl/security label
`​``

GraphRAG 的高频查询一般是：

`​``text
query -> entity linking -> k-hop expansion -> evidence fetch -> context ranking
`​``

所以最核心的是：

`​``text
entity lookup index
adjacency index
evidence index
vector index
permission index
`​``

---

**6. 查询引擎要支持四类查询**

不能只支持 Cypher 的 MATCH。

至少要支持：

`​``text
1. Pattern Query
   查实体关系模式：
   MATCH (a)-[:SUPPLIES]->(b)

2. Path Query
   查路径、最短路径、受约束路径：
   A 到 B 之间经过哪些供应链/投资/风险路径？

3. Evidence Query
   查关系证据：
   这条边由哪些文档、chunk、系统记录支撑？

4. Context Query
   面向 RAG 的上下文子图：
   给定问题和实体，返回一个可喂给 LLM 的证据子图。
`​``

第四类是传统图数据库通常没有的，但对 GraphRAG 很关键。

可以设计一个专门 API：

`​``text
graph_context(
  seeds = [entity_id],
  max_hops = 2,
  relation_types = [...],
  min_confidence = 0.7,
  time_range = ...,
  evidence_required = true,
  max_tokens = 6000
)
`​``

返回：

`​``text
entities
relations
paths
evidence_chunks
scores
citations
`​``

这比让应用层自己拼 Cypher 更产品化。

---

**7. Planner 要图感知 + RAG 感知**

普通图查询优化关注：

`​``text
从哪个点开始？
先走哪条边？
哪个过滤条件选择率最高？
`​``

这里还要关注：

`​``text
哪些边类型对问题有用？
哪些路径可信度更高？
哪些证据可引用？
哪些节点用户有权限看？
哪些结果适合进入 LLM context？
`​``

所以 planner 需要同时考虑：

`​``text
graph cost
semantic score
confidence score
evidence availability
freshness
permission
token budget
`​``

GraphRAG 查询不是返回越多越好，而是要返回“有限、可信、可引用”的子图。

---

**8. 推理能力要内建但分层**

知识图谱需要推理，但不要一开始做复杂 OWL。

建议分三层：

`​``text
Layer 1: Rule Inference
  A 属于 B，B 属于 C，则 A 间接属于 C。

Layer 2: Path Reasoning
  根据路径类型、长度、置信度评估关系强弱。

Layer 3: LLM-assisted Reasoning
  LLM 只做解释、候选关系生成、冲突摘要，不直接污染主图。
`​``

推理结果应单独标记：

`​``text
edge.source = inferred
edge.derived_from = [edge_id_1, edge_id_2]
edge.confidence = 0.76
edge.status = candidate / verified
`​``

不要把推理事实和原始事实混在一起。

---

**9. Evidence Graph 是关键设计**

如果面向 RAG，图数据库里必须有“证据图”概念。

可以建模为：

`​``text
(Entity)-[RELATION]->(Entity)

(Relation)-[SUPPORTED_BY]->(Evidence)
(Evidence)-[FROM_CHUNK]->(Chunk)
(Chunk)-[FROM_DOCUMENT]->(Document)
(Document)-[FROM_SOURCE]->(Source)
`​``

这样 RAG 回答时可以输出：

`​``text
答案依据哪些边？
这些边来自哪些 chunk？
chunk 来自哪个文档？
用户是否有权限看这些证据？
`​``

没有 evidence graph，GraphRAG 很容易变成“看似有图，实际不可追溯”。

---

**10. 对外接口设计**

建议不要只暴露 Cypher。

可以分三类 API：

`​``text
1. Graph Query API
   Cypher / GQL / Gremlin-like traversal

2. Knowledge API
   upsert_entity
   upsert_relation
   merge_entity
   get_evidence
   validate_relation

3. GraphRAG API
   link_entities(query)
   retrieve_context(query, entities)
   expand_subgraph(seeds, constraints)
   explain_path(path_id)
`​``

这样数据库不只是“能查图”，而是直接服务知识图谱和 AI 应用。

---

**11. 写入链路设计**

知识图谱/RAG 的写入不是简单插点边，而是：

`​``text
source ingest
  -> entity extraction
  -> entity resolution
  -> relation extraction
  -> schema validation
  -> evidence linking
  -> confidence scoring
  -> graph upsert
  -> index update
`​``

数据库系统层面至少要支持：

`​``text
幂等 upsert
实体合并
边去重
候选事实
人工审核状态
版本回滚
证据绑定
批量导入
增量更新
`​``

尤其是实体合并：

`​``text
A_alias 和 A_entity 被合并后，
相关边、证据、embedding、缓存都要迁移或重建。
`​``

这是知识图谱数据库区别于普通图数据库的重要点。

---

**12. 推荐的系统形态**

如果从数据库产品角度设计，我会选：

`​``text
Property Graph Core
+ Ontology Catalog
+ Evidence Store
+ Vector Index
+ GraphRAG Context Engine
+ Rule/Path Reasoning
+ Governance Layer
`​``

一句话：

> 不是做一个“带向量字段的图数据库”，而是做一个“面向知识和 RAG 的证据型图数据库”。

---

**13. MVP 可以这样收敛**

第一阶段不要做太大。先做这些：

`​``text
1. 属性图存储
   Entity / Relation / properties

2. Schema catalog
   entity_type / relation_type / allowed_relation

3. Evidence-aware edge
   每条边可以挂 source、confidence、evidence_id

4. Adjacency index
   支持 1-3 跳高性能扩展

5. Entity linking support
   name / alias / external_id 检索

6. Vector index
   支持 entity / relation / chunk embedding

7. Context retrieval API
   输入 query + seeds，输出 ranked subgraph + evidence

8. Permission filter
   图遍历和证据返回都要过权限

9. Basic reasoning
   支持规则推导和路径打分

10. Explainability
   每个返回关系都能解释：为什么返回、证据在哪、置信度多少
`​``

暂时不要做：

`​``text
完整 RDF/OWL
完整 Cypher/GQL 兼容
分布式图计算
复杂图算法平台
全自动知识构建
`​``

---

**14. 最终设计判断**

如果限制在“图数据库”内支撑知识图谱和 GraphRAG，我的核心建议是：

`​``text
图数据库内核 = 图存储 + 图查询 + 图索引
             + 本体/schema
             + 证据/provenance
             + 向量/语义索引
             + 权限/版本/置信度
             + 面向 RAG 的上下文子图检索
`​``

传统图数据库解决的是：

`​``text
关系怎么存，路径怎么查。
`​``

面向 AI 的图数据库要解决的是：

`​``text
哪些关系可信？
关系证据在哪？
如何把相关子图变成 LLM 可用上下文？
如何避免无权限和低质量信息进入答案？
`​``

所以架构重心应该从“图查询数据库”转向：

`​``text
Evidence-aware Knowledge Graph Database for AI
`​``

这会比单纯做 AGE/Neo4j/Nebula 的同类替代更有差异化。

<!-- /message -->
<!-- message:id=msg_901f5c74168e35cc role=user createdAt=2026-06-01T05%3A56%3A52.227Z -->

## User

对于存储层而言，这几种store的核心差异是什么，是逻辑上分开还是物理上实现为不同类型的存储？内建向量索引的对立面是什么？内建对图的能力有何要求？举例说明四类查询中的3，4类。知识图谱一般用什么数据库，属性图在这个场景的优劣势。

<!-- /message -->
<!-- message:id=msg_fdab98e2a26deb55 role=assistant createdAt=2026-06-01T05%3A59%3A03.293Z runId=run_337bde4357fb4b20 status=succeeded -->

## Assistant

可以把问题拆成 6 个点。

**1. 几种 Store 的核心差异是什么**

这里说的 `Entity Store`、`Relation Store`、`Adjacency Store`、`Property Store`、`Evidence Store`、`Embedding Store`、`Schema/Ontology Store`，核心差异不是名字，而是 **访问模式不同**。

| Store | 存什么 | 主要访问模式 | 核心优化目标 |
|---|---|---|---|
| Entity Store | 实体点：公司、人、产品、文档、概念 | 按 ID、名称、别名、外部 ID 查实体 | 实体定位、实体消歧、主数据治理 |
| Relation Store | 关系边：供应、投资、任职、引用、依赖 | 按边 ID、两端点、关系类型查边 | 关系事实管理、边属性、置信度、时间 |
| Adjacency Store | 邻接关系：某点有哪些入边/出边 | 给定点快速扩展邻居 | 多跳遍历、路径查询、k-hop expansion |
| Property Store | 点边属性 | 按属性过滤、排序、聚合 | 属性查询、schema 灵活性、冷热字段优化 |
| Evidence Store | 边/实体的证据来源 | 从关系追溯文档、chunk、span、来源系统 | 可解释、可引用、可信度、审计 |
| Embedding Store | entity/relation/chunk/subgraph embedding | 向量近邻检索 | 语义召回、实体链接、GraphRAG |
| Schema/Ontology Store | 类型、本体、约束、规则 | 校验、推理、查询规划 | 图谱治理、语义约束、规则推导 |

举个例子：

`​``text
Entity Store:
  A公司
  B公司

Relation Store:
  A公司 -[SUPPLIES {product: 电池, confidence: 0.91}]-> B公司

Adjacency Store:
  out_adj(A公司, SUPPLIES) -> [B公司, C公司, D公司]

Evidence Store:
  relation_id = r123 -> 年报第 32 页、新闻稿 chunk_889、供应商公告

Embedding Store:
  A公司 embedding
  “电池供应链风险” chunk embedding
  SUPPLIES relation embedding

Schema/Ontology Store:
  Company 可以通过 SUPPLIES 指向 Company
  SUPPLIES 必须有 product、valid_from、confidence
`​``

所以它们的区别主要是：

`​``text
Entity/Relation Store 解决“事实怎么存”
Adjacency Store 解决“关系怎么快速走”
Property Store 解决“属性怎么查”
Evidence Store 解决“事实凭什么可信”
Embedding Store 解决“语义上怎么召回”
Schema/Ontology Store 解决“什么关系合法、如何治理和推理”
`​``

---

**2. 逻辑上分开，还是物理上实现为不同类型存储？**

两层都要考虑。

**逻辑上一定要分开。**

因为它们的职责不同。即使最后都落在同一个数据库、同一个存储引擎里，系统设计上也应该分成这些逻辑模块。

例如在 PostgreSQL 里可以这样实现：

`​``text
kg_entity                 -- Entity Store
kg_relation               -- Relation Store
kg_adj_out / kg_adj_in     -- Adjacency Store
kg_entity_property         -- Property Store，或者 JSONB/agtype
kg_evidence               -- Evidence Store
kg_embedding              -- Embedding Store，pgvector
kg_ontology_*             -- Schema/Ontology Store
`​``

物理上是否分开，取决于规模和性能目标。

**早期/MVP：可以物理上不分开。**

比如：

`​``text
一个 PostgreSQL 集群
  - 普通表存实体、关系、证据、本体
  - AGE/自研图扩展存点边
  - pgvector 存 embedding
  - GIN/B-tree/HNSW 做索引
`​``

这种方式工程复杂度低，事务一致性好，适合知识图谱和 GraphRAG 的第一阶段。

**中后期：部分 Store 需要物理分开。**

因为访问模式差异太大。

例如：

`​``text
Entity/Relation Store:
  行存、LSM、关系表都可以，重视事务和更新。

Adjacency Store:
  更适合邻接表、KV、CSR、压缩数组、边分区。
  核心是给定 vertex_id 快速拿到邻居。

Embedding Store:
  需要 HNSW、IVF、PQ 等 ANN 向量索引。
  和普通 B-tree/邻接索引完全不同。

Evidence Store:
  可能是关系表 + 文档存储 + 对象存储。
  需要保留 chunk、span、原文、来源、版本。

Property Store:
  热属性可以列式化，冷属性可以 JSON/agtype。
  分析型属性还可能需要列存。
`​``

可以这样理解：

`​``text
逻辑分离是架构要求。
物理分离是性能和规模要求。
`​``

一个成熟图数据库可能内部是混合物理布局：

`​``text
Graph Core:
  vertex table
  edge table
  adjacency index

Property Layer:
  columnar hot properties
  dynamic property map

Vector Layer:
  ANN index

Evidence Layer:
  document/chunk/span store

Catalog Layer:
  ontology/schema/rule metadata
`​``

但对外暴露成一个统一的图数据库。

---

**3. 内建向量索引的对立面是什么？**

“内建向量索引”的对立面不是“不做向量”，而是：

`​``text
图数据库不负责向量检索，
向量检索交给外部系统。
`​``

常见对立方案有三种。

**方案 A：外部向量数据库**

例如：

`​``text
Graph DB:
  存实体、关系、路径、证据 ID

Vector DB:
  Milvus / Qdrant / Weaviate / Pinecone / Elasticsearch / OpenSearch
  存 chunk embedding、entity embedding、relation embedding
`​``

查询流程：

`​``text
query -> vector DB 找 chunk/entity
      -> 返回 entity_id / chunk_id
      -> graph DB 做图扩展
      -> evidence store 拉证据
`​``

优点：

`​``text
向量能力强
规模扩展独立
ANN 算法、压缩、GPU、分片能力更成熟
`​``

缺点：

`​``text
图和向量之间需要跨系统 join
权限过滤复杂
事务一致性弱
更新链路复杂
GraphRAG latency 更难控
`​``

**方案 B：应用层自己编排**

`​``text
应用层先查向量库
再查图数据库
再查文档库
最后自己融合排序
`​``

优点是灵活，缺点是数据库产品本身没有形成闭环，很多能力都变成应用代码。

**方案 C：只存 embedding，不建向量索引**

图数据库里有：

`​``text
entity.embedding
chunk.embedding
relation.embedding
`​``

但不提供 ANN 索引，只支持外部扫描或离线计算。

这只能用于小规模，生产上不够。

所以：

`​``text
内建向量索引 =
  图数据库内部直接支持 embedding 存储、ANN 检索、图遍历和权限过滤的联合执行。

对立面 =
  向量检索外置，图数据库只接受外部返回的 entity_id/chunk_id，再做图查询。
`​``

内建的关键价值是：

`​``text
语义召回 + 图扩展 + 权限过滤 + 证据追溯 可以在一个查询计划里完成。
`​``

---

**4. “内建”对图能力有什么要求？**

如果图数据库要内建支撑知识图谱和 GraphRAG，不能只是“有点边表”。至少要有这些能力。

**第一，原生图模型**

需要支持：

`​``text
vertex
edge
label/type
properties
path
subgraph
`​``

而且边应该是一等对象，有自己的 ID 和属性。因为知识图谱里的边需要挂：

`​``text
confidence
evidence
source
valid_time
status
version
permissions
`​``

**第二，邻接访问能力**

必须能高效回答：

`​``text
给定实体 A，查它的所有出边
给定实体 A，查它的所有入边
给定实体 A，只查 SUPPLIES 类型出边
给定实体 A，查 2 跳以内的风险路径
`​``

也就是需要类似：

`​``text
out_adj(vertex_id, edge_type)
in_adj(vertex_id, edge_type)
`​``

如果每次遍历都退化成大表 join，GraphRAG 的 k-hop expansion 会很容易炸。

**第三，路径查询能力**

知识图谱和 GraphRAG 很多问题不是查一条边，而是查路径：

`​``text
A 公司和 B 公司之间有什么关系？
某风险事件会通过哪些供应链路径影响目标公司？
某员工为什么有权限访问某系统？
某概念和某产品之间通过哪些政策、文档、实体相连？
`​``

所以要支持：

`​``text
variable-length path
shortest path
bounded path
typed path
filtered path
top-k path
path scoring
cycle control
`​``

**第四，图查询优化器**

Planner 不能只知道普通索引，还要知道图统计：

`​``text
每类边有多少
某类实体平均度数
某个实体是不是超级节点
路径扩展会不会爆炸
从哪个种子点开始查更便宜
哪些边类型应该先过滤
`​``

比如：

`​``cypher
MATCH (a:Company)-[:SUPPLIES*1..3]->(b:Company)
WHERE b.risk_score > 90
`​``

优化器应该判断：

`​``text
是先找 risk_score > 90 的 b，
再反向遍历 SUPPLIES，
还是从 a 正向扩展更便宜。
`​``

**第五，证据感知能力**

知识图谱/RAG 场景下，边不只是边。数据库要能快速回答：

`​``text
这条边有哪些证据？
证据来自哪个文档？
文档是否可访问？
证据是否过期？
证据是否和其他证据冲突？
`​``

这要求图查询能和 evidence store 联动。

**第六，权限过滤能力**

GraphRAG 不能先查出全图再在应用层过滤。否则可能泄露敏感关系。

图遍历阶段就要支持：

`​``text
tenant_id filter
ACL filter
security label filter
document permission filter
edge permission filter
`​``

**第七，上下文子图抽取能力**

这是面向 RAG 的特殊要求。

传统图数据库返回：

`​``text
点
边
路径
`​``

GraphRAG 更需要返回：

`​``text
有限 token budget 内最有用的子图
相关实体
关键关系
证据 chunk
路径解释
引用信息
置信度
`​``

所以最好内建类似：

`​``text
retrieve_context()
expand_context_subgraph()
rank_paths()
`​``

而不是只让用户自己写复杂 Cypher。

---

**5. 举例：四类查询中的第 3 类，Evidence Query**

第 3 类是：

`​``text
Evidence Query = 查某个实体、关系、路径背后的证据。
`​``

它回答的不是“有没有关系”，而是：

`​``text
这条关系为什么成立？
来自哪里？
置信度多少？
哪段原文支持？
有没有冲突证据？
`​``

例子：

用户问：

`​``text
A 公司是否供应电池给 B 公司？证据是什么？
`​``

图里有：

`​``text
(A公司)-[r:SUPPLIES {product: "电池", confidence: 0.91}]->(B公司)

(r)-[:SUPPORTED_BY]->(ev1:Evidence)
(ev1)-[:FROM_CHUNK]->(c1:Chunk)
(c1)-[:FROM_DOCUMENT]->(d1:Document)
`​``

查询可以是 Cypher 风格：

`​``cypher
MATCH (a:Entity {name: "A公司"})-[r:SUPPLIES]->(b:Entity {name: "B公司"})
MATCH (r)-[:SUPPORTED_BY]->(ev:Evidence)-[:FROM_CHUNK]->(c:Chunk)-[:FROM_DOCUMENT]->(d:Document)
WHERE r.product = "电池"
  AND r.confidence >= 0.8
RETURN
  a.name AS supplier,
  b.name AS customer,
  r.product AS product,
  r.confidence AS confidence,
  d.title AS document_title,
  c.section AS section,
  c.text AS evidence_text,
  ev.extractor AS extractor,
  ev.created_at AS extracted_at
ORDER BY r.confidence DESC;
`​``

返回结果不是普通图节点，而是类似：

`​``text
supplier: A公司
customer: B公司
relation: SUPPLIES
product: 电池
confidence: 0.91
evidence:
  - document: 2025 年供应链报告
    section: 第 3 章，核心供应商
    text: “A 公司为 B 公司提供动力电池模组……”
    source: annual_report
    extractor: llm_v2
`​``

更进一步，Evidence Query 还可以查冲突：

`​``cypher
MATCH (a:Entity {name: "A公司"})-[r]->(b:Entity {name: "B公司"})
MATCH (r)-[:SUPPORTED_BY]->(pos:Evidence)
OPTIONAL MATCH (r)-[:CONTRADICTED_BY]->(neg:Evidence)
RETURN r, collect(pos), collect(neg);
`​``

这类查询对知识图谱和 RAG 都很关键，因为它让系统能说：

`​``text
我为什么相信这条边？
我能不能把它作为答案引用？
`​``

---

**6. 举例：四类查询中的第 4 类，Context Query**

第 4 类是：

`​``text
Context Query = 面向 LLM/RAG 的上下文子图查询。
`​``

它不是简单查边，也不是简单查证据，而是：

`​``text
给定问题和种子实体，
返回一个有限、可信、可引用、可放进 prompt 的子图上下文。
`​``

例子：

用户问：

`​``text
B 公司是否存在电池供应链风险？
`​``

系统先做 entity linking：

`​``text
B 公司 -> entity_id = company_b
`​``

然后执行 context query：

`​``text
seeds:
  [B公司]

constraints:
  max_hops = 2
  relation_types = [SUPPLIES, OWNS, SANCTIONED_BY, HAS_RISK_EVENT, LOCATED_IN]
  min_confidence = 0.75
  evidence_required = true
  time_range = 最近 24 个月
  max_tokens = 6000
`​``

可以设计成专门 API：

`​``json
{
  "query": "B 公司是否存在电池供应链风险？",
  "seeds": ["company_b"],
  "max_hops": 2,
  "relation_types": [
    "SUPPLIES",
    "OWNS",
    "HAS_RISK_EVENT",
    "SANCTIONED_BY",
    "LOCATED_IN"
  ],
  "min_confidence": 0.75,
  "evidence_required": true,
  "time_range": {
    "from": "2024-06-01",
    "to": "2026-06-01"
  },
  "max_tokens": 6000
}
`​``

数据库内部做：

`​``text
1. 从 B 公司开始反向查供应商
2. 找供应商是否有风险事件
3. 找供应商所在地、制裁、诉讼、产能异常
4. 找每条关系的证据 chunk
5. 对路径打分
6. 过滤无权限证据
7. 返回 top-k 风险路径和证据
`​``

返回可能是：

`​``json
{
  "entities": [
    {"id": "company_b", "name": "B公司", "type": "Company"},
    {"id": "company_a", "name": "A公司", "type": "Company"},
    {"id": "risk_77", "name": "产能受限事件", "type": "RiskEvent"}
  ],
  "relations": [
    {
      "id": "r1",
      "type": "SUPPLIES",
      "from": "company_a",
      "to": "company_b",
      "properties": {"product": "电池"},
      "confidence": 0.91,
      "evidence_ids": ["ev1"]
    },
    {
      "id": "r2",
      "type": "HAS_RISK_EVENT",
      "from": "company_a",
      "to": "risk_77",
      "properties": {"risk_type": "capacity_constraint"},
      "confidence": 0.86,
      "evidence_ids": ["ev2"]
    }
  ],
  "paths": [
    {
      "path": ["company_a", "SUPPLIES", "company_b"],
      "score": 0.88,
      "reason": "A公司是B公司的电池供应商，且关系有近期证据支撑"
    },
    {
      "path": ["company_a", "HAS_RISK_EVENT", "risk_77"],
      "score": 0.81,
      "reason": "供应商A公司存在产能受限风险事件"
    }
  ],
  "evidence": [
    {
      "id": "ev1",
      "document": "B公司2025供应链报告",
      "chunk": "A公司为B公司的电池模组供应商……",
      "citation": "第12页"
    },
    {
      "id": "ev2",
      "document": "A公司产能公告",
      "chunk": "受工厂检修影响，动力电池产线预计减少……",
      "citation": "公告第2段"
    }
  ]
}
`​``

这类查询的重点不是“返回所有邻居”，而是：

`​``text
返回适合 LLM 使用的、有证据支撑的、经过排序的上下文子图。
`​``

传统 Cypher 可以拼出来一部分，但作为数据库产品，最好把它抽象成专门能力。

---

**7. 知识图谱一般用什么数据库？**

知识图谱常见有几类数据库路线。

**第一类：RDF Triple Store / Semantic Graph Database**

典型代表：

`​``text
GraphDB
Stardog
Virtuoso
Amazon Neptune RDF mode
Apache Jena/Fuseki
Blazegraph
`​``

数据模型：

`​``text
subject - predicate - object
`​``

查询语言：

`​``text
SPARQL
`​``

优势：

`​``text
语义标准成熟
RDF/OWL/RDFS 生态完整
适合本体、推理、语义互操作
适合开放知识图谱、标准知识表达
`​``

劣势：

`​``text
业务工程开发门槛高
属性、边属性、复杂业务对象表达不如属性图直观
SPARQL 对普通工程团队不友好
和业务系统的数据模型有距离
`​``

适合：

`​``text
本体强约束
语义网
标准知识库
学术/公共知识图谱
需要 OWL/RDFS 推理的场景
`​``

---

**第二类：Property Graph Database**

典型代表：

`​``text
Neo4j
NebulaGraph
TigerGraph
JanusGraph
Amazon Neptune property graph mode
ArangoDB graph
AGE / PostgreSQL graph extension
`​``

数据模型：

`​``text
(node)-[edge]->(node)
node 和 edge 都有 label/type/properties
`​``

查询语言：

`​``text
Cypher
GQL
Gremlin
nGQL
`​``

优势：

`​``text
贴近业务实体关系建模
边可以直接带属性
路径查询直观
工程团队更容易理解
适合风控、推荐、供应链、IAM、企业知识图谱、GraphRAG
`​``

劣势：

`​``text
语义标准化弱于 RDF
本体和推理能力通常不如 RDF 原生
跨系统语义互操作弱
复杂语义约束要自己实现
`​``

适合：

`​``text
企业业务图谱
AI/GraphRAG
风控图谱
客户关系图谱
供应链图谱
权限图谱
运维拓扑图
知识检索图谱
`​``

---

**第三类：关系数据库 / 多模型数据库**

典型方式：

`​``text
PostgreSQL + AGE + pgvector
PostgreSQL + recursive SQL + JSONB
Oracle Graph
SQL Server Graph
ArangoDB 多模型
MongoDB + graph lookup
`​``

优势：

`​``text
容易和现有业务系统集成
事务、权限、备份、审计成熟
适合把图作为业务数据的一种视图或索引
`​``

劣势：

`​``text
深度遍历和图算法性能通常不如原生图数据库
图查询能力可能不完整
优化器不一定图感知
`​``

适合：

`​``text
企业已有数据在关系库
图规模中等
重视治理和事务
需要 SQL + 图 + 向量混合
`​``

---

**第四类：搜索/向量/文档系统辅助知识图谱**

例如：

`​``text
Elasticsearch / OpenSearch
Milvus / Qdrant / Weaviate
Document DB
Object Store
`​``

这些一般不是主图数据库，但常用于知识图谱和 GraphRAG 的辅助层：

`​``text
全文检索
向量检索
文档证据存储
chunk 检索
`​``

---

**8. 属性图在知识图谱场景的优势**

属性图最大的优势是 **工程实用性**。

比如表达这个事实：

`​``text
A公司向B公司供应电池，置信度0.91，证据来自某年报，关系有效期从2024年开始。
`​``

属性图可以很自然：

`​``text
(A公司)-[:SUPPLIES {
  product: "电池",
  confidence: 0.91,
  valid_from: "2024-01-01",
  evidence_id: "ev123"
}]->(B公司)
`​``

这对业务系统很友好。

优势包括：

`​``text
1. 实体-关系表达直观
   点就是实体，边就是关系。

2. 边属性天然支持
   置信度、时间、来源、状态、证据 ID 可以直接挂在边上。

3. 路径查询自然
   供应链、投资链、权限链、风险传播都很适合。

4. 和 GraphRAG 匹配
   LLM 需要的是实体、关系、路径、证据，属性图表达很顺。

5. 开发者友好
   Cypher/GQL 风格比 SPARQL 更容易被普通工程团队接受。

6. 适合企业业务图谱
   企业里的“客户、合同、产品、供应商、员工、系统、文档”都可以自然建模。
`​``

例如 GraphRAG 中的典型图：

`​``text
(Document)-[:HAS_CHUNK]->(Chunk)
(Chunk)-[:MENTIONS]->(Company)
(Company)-[:SUPPLIES {confidence, evidence_id}]->(Company)
(Company)-[:HAS_RISK_EVENT]->(RiskEvent)
`​``

属性图非常顺手。

---

**9. 属性图在知识图谱场景的劣势**

属性图的劣势主要在 **语义标准、推理和互操作**。

**第一，本体能力弱于 RDF/OWL**

RDF/OWL 可以表达：

`​``text
父类/子类
等价类
属性继承
domain/range
传递关系
对称关系
函数属性
逻辑约束
`​``

属性图通常没有这些标准语义，需要自己实现。

例如：

`​``text
所有 SubsidiaryOf 都是 OwnedBy 的一种
如果 A SubsidiaryOf B，B LocatedIn C，则 A 也受 C 地区政策影响
`​``

在 RDF/OWL 里有标准推理体系；属性图里通常要靠规则引擎或应用逻辑。

**第二，标准化和互操作弱**

RDF 的三元组和 SPARQL 是知识图谱领域的标准。很多开放数据、本体、语义资源都基于 RDF。

属性图生态更偏工程和产品，不同数据库之间模型、查询语言、导入导出兼容性不如 RDF 统一。

**第三，语义精确性不够**

属性图里一个边类型：

`​``text
RELATED_TO
`​``

很容易被滥用。没有强本体治理时，图会变成“什么都能连”的关系网。

**第四，复杂 n-ary relation 需要额外建模**

有些关系不是简单二元边。

例如：

`​``text
A公司在2025年Q1通过合同C向B公司供应产品P，金额M，交付地L。
`​``

如果只用一条边：

`​``text
(A)-[:SUPPLIES]->(B)
`​``

会丢很多语义。

需要把关系事件实体化：

`​``text
(A公司)-[:PARTICIPATES_IN]->(SupplyContract)
(B公司)-[:PARTICIPATES_IN]->(SupplyContract)
(SupplyContract)-[:HAS_PRODUCT]->(Product)
(SupplyContract)-[:HAS_AMOUNT]->(Amount)
(SupplyContract)-[:HAS_EVIDENCE]->(Evidence)
`​``

属性图可以做，但需要建模规范。

**第五，推理结果和事实容易混淆**

属性图很容易把所有边都混在一起：

`​``text
原始事实边
模型抽取边
规则推理边
人工确认边
候选边
`​``

如果没有 `source/status/confidence/derived_from` 设计，会污染图谱。

---

**10. 怎么选：RDF 还是属性图？**

可以简单判断。

适合 RDF / Triple Store：

`​``text
强本体
强语义标准
需要 OWL/RDFS 推理
需要 SPARQL
需要和开放知识库互操作
知识表达比业务查询更重要
`​``

适合 Property Graph：

`​``text
企业业务知识图谱
GraphRAG
风控、供应链、权限、客户关系
路径查询多
边属性多
工程落地优先
需要和业务系统、文档、向量检索结合
`​``

如果你的目标是前面说的这两个场景：

`​``text
知识图谱 + AI/RAG 增强
`​``

我会优先选：

`​``text
属性图为核心
Schema/Ontology Catalog 做治理
Evidence Store 做证据链
Vector Index 做语义召回
Rule Engine 做轻量推理
`​``

也就是：

`​``text
Property Graph + Ontology-lite + Evidence-aware + Vector-native
`​``

而不是一开始就上完整 RDF/OWL。

原因是这两个场景更关心：

`​``text
实体怎么对齐
关系怎么查
路径怎么解释
证据怎么引用
上下文怎么喂给 LLM
权限怎么过滤
`​``

这些问题属性图更直接。

最终判断：

`​``text
RDF 更像“语义标准知识库”。
属性图更像“企业关系事实库 + 路径查询引擎”。
GraphRAG 更需要后者，但要补上本体、证据、置信度、版本和权限。
`​``

<!-- /message -->
<!-- message:id=msg_fac0b1f8cf4ce9c3 role=user createdAt=2026-06-01T06%3A27%3A28.928Z -->

## User

Amazon Neptune 支持多种模式，这几种模式底层是一样的吗？数据如何互通？快答

<!-- /message -->
<!-- message:id=msg_0dd9eb54a257386b role=assistant createdAt=2026-06-01T06%3A28%3A08.266Z runId=run_27f52866979eee77 status=succeeded -->

## Assistant

快答：

**Neptune 不是“三种语言查同一份统一图”。它是两种数据模型、三种查询入口。**

`​``text
Property Graph 模型：
  Gremlin
  openCypher

RDF 模型：
  SPARQL
`​``

**底层是否一样？**

从产品形态看，它们在同一个 Neptune Database cluster 里，由 Neptune 的专用图数据库引擎管理；AWS 明确说 Neptune 不是关系数据库，而是 purpose-built graph database engine。  
但从数据模型和查询可见性看，**Property Graph 和 RDF 是两套逻辑数据空间/模型**，不能互相直接查询。

AWS 官方 FAQ 说得很清楚：

- 一个 Neptune Database cluster 可以同时存 property graph 数据和 RDF 数据。
- Gremlin 和 openCypher 可以查询同一份 property graph 数据。
- 不能用 Gremlin/openCypher 查 RDF 数据，也不能用 SPARQL 查 property graph 数据。  
来源：AWS Neptune FAQ 和文档说明。  
https://aws.amazon.com/neptune/faqs/  
https://docs.aws.amazon.com/neptune/latest/userguide/get-started-access-graph.html

**数据如何互通？**

分两层：

**1. Gremlin 与 openCypher 之间：基本互通**

它们都跑在 Neptune 的 **property graph** 模型上。

也就是说：

`​``text
用 Gremlin 写入的点边
可以用 openCypher 查询。

用 openCypher 写入的点边
也可以用 Gremlin 查询。
`​``

AWS 文档明确说：Gremlin 和 openCypher 可以查询 Neptune 中任何 property-graph 数据，不管数据最初是怎么加载的。  
https://docs.aws.amazon.com/neptune/latest/userguide/access-graph-queries.html

例如：

`​``text
Gremlin:
  g.addV('Company').property('name', 'A')

openCypher:
  MATCH (c:Company {name: 'A'}) RETURN c
`​``

大体可以互通。

但要注意细节差异：

`​``text
Gremlin 支持一些属性图能力，openCypher 未必完全等价。
例如 Neptune 文档提到：Gremlin 创建的 multi-valued properties，在 openCypher 里有特殊语义处理。
`​``

所以是 **同一 property graph 数据模型上的语言互通**，不是 100% 语义完全一致。

**2. Property Graph 与 RDF/SPARQL 之间：不直接互通**

`​``text
Gremlin/openCypher 不能直接查 RDF。
SPARQL 不能直接查 property graph。
`​``

如果要互通，需要做 **模型转换 / ETL / 双写 / 映射层**。

例如把属性图：

`​``text
(A:Company)-[:SUPPLIES {product: "battery", confidence: 0.9}]->(B:Company)
`​``

转换成 RDF，可能要变成：

`​``text
:A rdf:type :Company .
:B rdf:type :Company .
:A :supplies :B .
`​``

如果要保留边属性 `product`、`confidence`，RDF 里通常还要把关系实体化：

`​``text
:supply_123 rdf:type :SupplyRelation .
:supply_123 :supplier :A .
:supply_123 :buyer :B .
:supply_123 :product "battery" .
:supply_123 :confidence "0.9" .
`​``

反过来，把 RDF 转成属性图，也要决定：

`​``text
RDF subject/object -> 节点？
RDF predicate -> 边类型？
literal -> 属性？
IRI/blank node 怎么映射？
RDF reification / RDF-star 怎么映射成边属性？
`​``

所以：

`​``text
Gremlin <-> openCypher：
  同一 property graph，天然互通。

Property Graph <-> RDF/SPARQL：
  同 cluster 可共存，但不是同一可查询数据模型；需要转换。
`​``

**加载数据也是分开的**

Neptune bulk load 支持两类格式：

`​``text
Property Graph:
  Gremlin CSV
  openCypher CSV

RDF:
  N-Triples
  N-Quads
  RDF/XML
  Turtle
`​``

AWS 文档也说明：property-graph 格式加载的数据可用 Gremlin 和 openCypher 查询；RDF 格式加载的数据用于 SPARQL 查询。  
https://docs.amazonaws.cn/en_us/neptune/latest/userguide/bulk-load-tutorial-format.html

**一句话总结**

`​``text
Amazon Neptune 是一个多模型图数据库服务，
但不是把 RDF、Gremlin、openCypher 都统一到一张完全互通的底层逻辑图里。

Gremlin 和 openCypher 共享 property graph 数据；
SPARQL 面向 RDF 数据；
Property Graph 和 RDF 可以在同一个 Neptune cluster 中共存，
但查询语言不能跨模型直接互查，互通要靠转换/ETL/应用层映射。
`​``

对架构启发是：如果你自己做图数据库，多模型支持要分清楚两件事：

`​``text
多语言访问同一模型：
  比如 Gremlin + Cypher 查询同一 property graph。
  这是相对可控的。

多模型互通：
  比如 RDF 和 Property Graph 互查互转。
  这是语义映射问题，不只是加一个查询 parser。
`​``

<!-- /message -->
<!-- message:id=msg_e7db3b7e96e6ebdf role=user createdAt=2026-06-02T00%3A22%3A33.718Z -->

## User

postgresql pgq实现定位，关键设计考虑和争议，用开发过程中的讨论来组织回答

<!-- /message -->
<!-- message:id=msg_a5ad2d0fc9fbfb37 role=assistant createdAt=2026-06-02T00%3A25%3A10.977Z runId=run_9985704f2fc4fc6e status=succeeded -->

## Assistant

下面用“开发过程讨论纪要”的方式组织。这里的 **PGQ** 指 **SQL/PGQ = SQL Property Graph Queries**，不是 SkyTools 那个队列系统 PGQ。

截至 **2026-06-02**，PostgreSQL 的 SQL/PGQ patch 已在 **2026-03-16** 提交到 `master`，CommitFest 记录为 **PG19-Final: Committed**，因此它是面向 PostgreSQL 19 开发线的能力，不是 PostgreSQL 18 稳定版里的成熟功能。提交说明明确说它实现 SQL/PGQ 标准，增加 `GRAPH_TABLE`、`CREATE/ALTER/DROP PROPERTY GRAPH`、新系统 catalog、`psql \dG`、`pg_get_propgraphdef()` 等。参见 CommitFest 和 commit 邮件。  
来源：PostgreSQL CommitFest、commit 邮件。  
https://commitfest.postgresql.org/patch/4904/  
https://www.postgresql.org/message-id/E1w247I-0000Tk-2Y%40gemulon.postgresql.org

**讨论 1：PGQ 的产品定位是什么？**

核心定位不是：

`​``text
PostgreSQL 变成 Neo4j / Nebula / TigerGraph 这样的原生图数据库
`​``

而是：

`​``text
PostgreSQL 在 SQL 标准内支持“关系数据的属性图视图”和图模式查询。
`​``

开发版文档里的表述很关键：property graph 是一种把数据库内容表示为图的方式；SQL/PGQ 中，property graph 被定义为 **relational tables 之上的 read-only view**，实际数据仍在表或类表对象里，图只是暴露给图查询的逻辑视图。图查询和关系查询可以在同一个 SQL 查询里混合。  
来源：pgEdge 镜像的 PostgreSQL development 文档。  
https://docs.pgedge.com/postgresql/development/the-sql-language/data-definition/property-graphs/

所以 PGQ 的第一性定位是：

`​``text
标准 SQL 图查询层
+ 关系表到属性图的映射
+ SQL 查询中的 GRAPH_TABLE 入口
+ 复用 PostgreSQL 现有 planner/executor/storage/security
`​``

而不是：

`​``text
新图存储
新邻接索引
新 Cypher 引擎
新图执行器
`​``

这点和 AGE 不同。AGE 更像 PostgreSQL extension 里的 Cypher/属性图执行层；PostgreSQL PGQ 是 SQL 标准的一部分，优先保证和 SQL 体系、catalog、权限、依赖、dump、psql、信息模式的融合。

**讨论 2：图数据到底怎么存？**

开发讨论里的结论基本是：

`​``text
数据仍然存在普通关系表中。
PROPERTY GRAPH 是一个新 relkind 对象，行为类似 view。
`​``

commit 邮件明确说：property graph 是一个带新 `RELKIND_PROPGRAPH` 的 relation，它在很多方面像 view，会在 rewriter 阶段改写成标准关系查询；权限类似 security invoker view。  
来源：commit 邮件。  
https://www.postgresql.org/message-id/E1w247I-0000Tk-2Y%40gemulon.postgresql.org

典型模型是：

`​``sql
CREATE TABLE customers (
  customer_id integer PRIMARY KEY,
  name text
);

CREATE TABLE orders (
  order_id integer PRIMARY KEY
);

CREATE TABLE customer_orders (
  customer_orders_id integer PRIMARY KEY,
  customer_id integer REFERENCES customers(customer_id),
  order_id integer REFERENCES orders(order_id)
);

CREATE PROPERTY GRAPH myshop
  VERTEX TABLES (
    customers,
    orders
  )
  EDGE TABLES (
    customer_orders
      SOURCE customers
      DESTINATION orders
  );
`​``

然后：

`​``sql
SELECT customer_name
FROM GRAPH_TABLE (
  myshop
  MATCH (c IS customers)-[IS customer_orders]->(o IS orders)
  COLUMNS (c.name AS customer_name)
);
`​``

这大致等价于关系 join：

`​``sql
SELECT customers.name
FROM customers
JOIN customer_orders USING (customer_id)
JOIN orders USING (order_id);
`​``

所以设计含义是：

`​``text
PGQ 不负责替你创建点表/边表。
PGQ 不把数据迁移到图存储。
PGQ 负责把已有表定义成图，并提供图模式匹配语法。
`​``

这对 PostgreSQL 很自然：关系库本来就擅长存表、约束、权限、事务、WAL、MVCC、备份。PGQ 把“图”作为关系数据的另一种查询视角。

**讨论 3：为什么走 GRAPH_TABLE，而不是 Cypher 函数？**

PGQ 的查询入口是 SQL 标准里的 `GRAPH_TABLE`：

`​``sql
SELECT ...
FROM GRAPH_TABLE (
  graph_name
  MATCH graph_pattern
  COLUMNS (...)
) AS g;
`​``

它像一个 `FROM` 子句里的 table function，输出一张关系表，可以继续 join、filter、order、group。开发版文档也说明，`GRAPH_TABLE` 对外表现为一个 computed table，可和其他表 join。  
来源：pgEdge 镜像文档。  
https://docs.pgedge.com/postgresql/development/the-sql-language/queries/graph-queries/

这体现了 PostgreSQL PGQ 的定位：

`​``text
图查询不是独立语言入口；
图查询是 SQL 里的子语言。
`​``

这和 AGE 的：

`​``sql
SELECT *
FROM cypher('graph', $$ MATCH ... $$) AS (...);
`​``

不同。AGE 是把 Cypher 字符串嵌进 SQL 函数；PGQ 是 SQL parser 直接理解 `GRAPH_TABLE` 和 graph pattern 语法。

优势：

`​``text
1. 更符合 SQL:2023 标准。
2. 更容易和 SQL 查询组合。
3. 更容易进入 PostgreSQL core 的权限、依赖、dump、psql、information_schema 体系。
4. 对企业 SQL 用户更自然。
`​``

劣势：

`​``text
1. 不是 Neo4j/Cypher 生态。
2. 图语法能力目前是 SQL/PGQ 子集。
3. 复杂图查询表达力和成熟图数据库还有差距。
`​``

**讨论 4：最大的架构争议，为什么用 rewriter？**

这是开发讨论里最关键的争议。

Andres Freund 早期指出：把 property graph query 在 **rewrite system** 中转换成 PostgreSQL 能理解的关系查询，可能不是好主意，因为这会让 planner 丢失图查询语义，也会增加对老旧 rewrite system 的依赖。  
来源：pgsql-hackers 讨论。  
https://www.postgresql.org/message-id/flat/a855795d-e697-4fa5-8698-d20122126567%40eisentraut.org

Peter Eisentraut 的回应是：PGQ 本来就倾向于像 view 展开成 joins/unions 那样实现；这是标准制定、其他实现和研究中的主流理解。也就是说，SQL/PGQ 的目标并不是 native graph storage + special graph executor。它是把关系数据暴露成图，再映射回关系执行。

这个争议可以概括成：

`​``text
方案 A：早期 rewrite 成关系查询
  优点：
    和 view 机制一致
    权限/安全/依赖处理路径清晰
    复用现有 planner/executor
    对 PostgreSQL core 更可接受
  缺点：
    planner 很早失去“这是图查询”的语义
    后续做图感知优化更难
    专用邻接索引、路径执行节点、图 cost model 不容易接入

方案 B：保留 graph plan node 到 planner/executor
  优点：
    可以做 graph-aware optimization
    可以引入路径、遍历、邻接索引、特殊 executor node
  缺点：
    工程复杂度大
    与 SQL/PGQ“关系视图”定位不完全一致
    会大幅影响 PostgreSQL core
`​``

最终 PostgreSQL 选择的是 **方案 A**。这说明它现在更像：

`​``text
SQL 标准图视图 + 关系查询改写
`​``

而不是：

`​``text
原生高性能图执行引擎
`​``

**讨论 5：是否要做“图数据库能力”？**

开发讨论中有人提出：如果 rewrite 太早，会不会影响未来做图索引、辅助结构、特殊 executor node？Tomas Vondra 就提到，保留图语义可能有助于新 index、辅助结构或特殊 executor node。Ashutosh Bapat 也提到，显式 graph annotation 未来可能帮助自动建索引、扩展统计、不同 costing 等。  
来源：pgsql-hackers 讨论。  
https://www.postgresql.org/message-id/flat/a855795d-e697-4fa5-8698-d20122126567%40eisentraut.org

这说明 PostgreSQL 社区其实意识到：

`​``text
当前 PGQ 是功能入口，不是性能终局。
`​``

现在的实现重点是：

`​``text
先把 SQL/PGQ 的 DDL、catalog、parser、rewriter、安全、dump、psql、基本查询跑通。
`​``

未来如果要更像图数据库，需要补：

`​``text
1. 图统计信息：
   label cardinality、edge type cardinality、degree distribution。

2. 图感知 planner：
   pattern reorder、起点选择、方向选择、路径代价估算。

3. 图索引：
   start_id/end_id/type 组合索引建议，或者专用 adjacency index。

4. 图执行节点：
   variable-length path、shortest path、top-k path、cycle control。

5. 图查询 explain：
   用户能看出图 pattern 如何被展开和执行。
`​``

但这些不是第一版 PGQ 的核心目标。

**讨论 6：功能范围为什么收得比较窄？**

PostgreSQL 社区对大 patch 非常谨慎。2026 年 1 月有一条专门讨论“如何 gate experimental features”的邮件。Peter 提出这个 patch 很大，不会第一版就完美，是否可以按 experimental 理解合入。Andres Freund 则质疑：一个处于早期的大功能是否值得合入，是否应该缩小范围、拆分，因为当时规模已经超过一百个文件、约一万五千行新增。  
来源：pgsql-hackers 讨论。  
https://www.postgresql.org/message-id/cwthrfgp2foh64aylx6wkcl2v2543qlqxztgkg6pwscqum7jw7%40oyf57d3x46fu

这带来一个重要实现策略：

`​``text
先合入核心框架；
高级图语义后续增量补。
`​``

从后续 Oracle 行为对比讨论看，当前 PostgreSQL 版本在查询覆盖上还不是完整 SQL/PGQ。邮件里提到尚未实现的范围包括：

`​``text
quantified path patterns
shortest path
graph types
ACYCLIC / SIMPLE path constraints
`​``

来源：pgsql-hackers Oracle 对比测试讨论。  
https://www.postgresql.org/message-id/flat/a855795d-e697-4fa5-8698-d20122126567%40eisentraut.org

所以第一版的定位应理解为：

`​``text
PostgreSQL 进入 SQL/PGQ 标准图查询的第一步。
不是完整图数据库，也不是完整 GQL/Cypher 替代。
`​``

**讨论 7：安全模型怎么定？**

安全是 PGQ 能否进入 PostgreSQL core 的硬门槛。

讨论里形成的逻辑是：

`​``text
1. 用户需要有 property graph 的 SELECT 权限。
2. property graph 类似 security invoker view。
3. 查询实际涉及到底层表和列时，当前用户也必须有相应权限。
4. RLS 需要在 GRAPH_TABLE 访问底层表时继续生效。
5. 暂不实现 security definer property graph，因为 SQL/PGQ 标准没有要求。
`​``

邮件中专门提到，`GRAPH_TABLE` rewrite 时要给底层表创建合适的权限信息，并把使用到的列加入 selected columns，以尊重列级权限；RLS 测试也单独补充。  
来源：pgsql-hackers 权限/RLS 讨论。  
https://www.postgresql.org/message-id/flat/a855795d-e697-4fa5-8698-d20122126567%40eisentraut.org

这个设计很 PostgreSQL：

`​``text
不能因为用户能访问一个 graph，就绕过底层表/列/RLS 权限。
`​``

这也是为什么它选择像 view 一样处理，而不是做一个完全独立图执行入口。

**讨论 8：依赖和 DDL 怎么处理？**

PGQ 引入 `CREATE PROPERTY GRAPH`、`ALTER PROPERTY GRAPH`、`DROP PROPERTY GRAPH`。这带来依赖问题：

`​``text
如果图里的 vertex table 被 drop，edge 怎么办？
如果 edge 依赖 vertex，能否自动删？
如果 view 依赖 GRAPH_TABLE，drop graph 怎么办？
`​``

PostgreSQL 的路线是严格依赖管理：

`​``text
默认 RESTRICT，必须显式 CASCADE。
`​``

在 Oracle 对比邮件中也提到：Oracle 倾向 lazy invalidation，允许底层表被 drop 后对象失效；PostgreSQL 采用严格依赖，防止留下无效对象。  
来源：Oracle 对比讨论。  
https://www.postgresql.org/message-id/flat/a855795d-e697-4fa5-8698-d20122126567%40eisentraut.org

这说明 PostgreSQL PGQ 的设计不是“图优先”，而是：

`​``text
PostgreSQL 对象依赖规则优先。
`​``

这对企业场景是好事：迁移、dump/restore、权限、审计、DDL 可预期性更强。

**讨论 9：catalog 和 relkind 怎么接入？**

commit 邮件显示新增了多个系统 catalog：

`​``text
pg_propgraph_element
pg_propgraph_element_label
pg_propgraph_label
pg_propgraph_label_property
pg_propgraph_property
`​``

同时 property graph 是新的 `RELKIND_PROPGRAPH`。  
来源：commit 邮件。  
https://www.postgresql.org/message-id/E1w247I-0000Tk-2Y%40gemulon.postgresql.org

这里也有实现争议：property graph 虽然是 relation-like object，但它不是普通 table。后续讨论里 Peter 认为不应让 `table_open()` 接受 `RELKIND_PROPGRAPH`，而是写专门的 parser open 逻辑，这样语义更干净、错误信息也更好。  
来源：pgsql-hackers 讨论。  
https://www.postgresql.org/message-id/flat/a855795d-e697-4fa5-8698-d20122126567%40eisentraut.org

设计含义：

`​``text
Property graph 是 PostgreSQL catalog 对象，
但不是可直接 scan/update 的物理表。
`​``

这和 view/materialized view/sequence/index 一样，都是 PostgreSQL 对象系统里的专门对象类型。

**讨论 10：标签和属性怎么设计？**

SQL/PGQ 的 property graph 由：

`​``text
vertex table
edge table
label
property
source/destination vertex
key
`​``

组成。

默认情况下：

`​``text
table name 暴露为 label
column 暴露为 property
primary key / foreign key 推导 vertex key 和 edge source/destination
`​``

也可以显式指定：

`​``sql
CREATE PROPERTY GRAPH myshop
  VERTEX TABLES (
    customers LABEL customer PROPERTIES (name),
    employees LABEL person PROPERTIES (employee_name AS name)
  )
  EDGE TABLES (...);
`​``

一个重要设计点是：同一个 label 可以应用到多个 element table，但 property 的数量、名称、类型要一致。开发版文档明确说明了这种 label sharing 的要求。  
来源：pgEdge 镜像文档。  
https://docs.pgedge.com/postgresql/development/the-sql-language/data-definition/property-graphs/

这对知识图谱很有价值。例如：

`​``text
customers 和 employees 都可以暴露为 person
但它们都必须有兼容的 name 属性
`​``

优势：

`​``text
可以用图 label 抽象不同关系表。
`​``

代价：

`​``text
label/property 一致性检查复杂；
查询展开时需要 union/append 多个底层表。
`​``

**讨论 11：和 Oracle / 其他实现的差异是什么？**

社区有人把 PostgreSQL PGQ regression tests 搬到 Oracle 上做行为对比。结论大致是：

`​``text
PostgreSQL 支持：
  ALTER PROPERTY GRAPH
  NODE/RELATIONSHIP 作为 VERTEX/EDGE 同义词
  TEMPORARY PROPERTY GRAPH
  LATERAL + GRAPH_TABLE
  GRAPH_TABLE 在 UDF 中的静态/动态使用

Oracle 支持但 PG 当前不支持：
  COLUMNS (a.*) 这种 star reference

Oracle 查询覆盖整体更广：
  quantified paths、shortest path 等高级能力
`​``

来源：pgsql-hackers Oracle 对比讨论。  
https://www.postgresql.org/message-id/flat/a855795d-e697-4fa5-8698-d20122126567%40eisentraut.org

这说明 PostgreSQL 的策略不是照抄 Oracle，而是：

`​``text
尽量贴 SQL 标准
遵循 PostgreSQL 自己的对象/依赖/权限哲学
先做 core shape，再补高级查询能力
`​``

**最终判断**

如果用一句话概括 PostgreSQL PGQ 的实现定位：

`​``text
PostgreSQL PGQ 是 SQL 标准下的“关系数据属性图视图 + 图模式查询”能力，
不是原生图数据库内核。
`​``

它的关键设计选择是：

`​``text
1. 数据仍在关系表。
2. PROPERTY GRAPH 是类似 view 的 catalog 对象。
3. GRAPH_TABLE 是 SQL FROM 子句里的图查询入口。
4. 查询在 rewriter 阶段展开成关系查询。
5. 复用 PostgreSQL planner/executor/storage/security。
6. 权限、RLS、依赖、pg_dump、psql、information_schema 要进入 core 体系。
7. 第一版先做标准骨架和基本 pattern matching，高级路径能力后续迭代。
`​``

核心争议是：

`​``text
早 rewrite 是否会牺牲未来 graph-aware optimization？
`​``

PostgreSQL 当前答案是：

`​``text
会牺牲一部分未来图优化空间，
但这是进入 core 最稳妥、最符合 SQL/PGQ 定位、最容易和现有 PostgreSQL 体系融合的路线。
`​``

对你前面讨论的“图数据库支撑知识图谱/GraphRAG”的启发是：

`​``text
PostgreSQL PGQ 适合作为“已有关系数据的图查询标准层”；
不适合作为“深度遍历、高性能路径搜索、图算法、GraphRAG 上下文扩展”的完整内核。
`​``

如果要做产品，可以把 PGQ 看成底座之一：

`​``text
PGQ:
  负责 SQL 标准图视图、图 pattern query、关系数据图化。

图数据库/GraphRAG 增强层:
  负责邻接索引、路径 ranking、证据边、向量索引、上下文子图、权限过滤、图推理。
`​``

也就是说，PGQ 的战略价值不是“PostgreSQL 已经有完整图数据库了”，而是：

`​``text
PostgreSQL core 开始承认 property graph 是 SQL 数据模型的一等视图。
这会给后续图索引、图统计、图执行器、GraphRAG 查询编排留下标准入口。
`​``

<!-- /message -->
