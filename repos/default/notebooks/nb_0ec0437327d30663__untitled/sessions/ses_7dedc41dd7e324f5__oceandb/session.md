---
id: "ses_7dedc41dd7e324f5"
title: "oceandb架构的主要演进，从系统架构师的视角看"
createdAt: "2026-05-17T23:02:51.450Z"
updatedAt: "2026-05-18T00:18:25.891Z"
settings: {}
runs: [{"id":"run_bc8b2221a8d8958c","status":"succeeded","startedAt":"2026-05-17T23:03:34.700Z","finishedAt":"2026-05-17T23:05:27.173Z","userMessageId":"msg_27ef96af5f5f24e0"},{"id":"run_8e20a549d2982853","status":"succeeded","startedAt":"2026-05-18T00:16:08.018Z","finishedAt":"2026-05-18T00:18:25.891Z","userMessageId":"msg_719c44c7f50af115"}]
---

# oceandb架构的主要演进，从系统架构师的视角看

<!-- message:id=msg_27ef96af5f5f24e0 role=user createdAt=2026-05-17T23%3A03%3A34.697Z -->

## User

oceandb架构的主要演进，从系统架构师的视角看

<!-- /message -->
<!-- message:id=msg_a3d6f9e44d832541 role=assistant createdAt=2026-05-17T23%3A05%3A27.173Z runId=run_bc8b2221a8d8958c status=succeeded -->

## Assistant

下面按 **OceanBase/OceanDB** 理解来讲。如果你说的是另一个名为 OceanDB 的项目，名称可能需要再对齐；但从“架构演进”这个语境看，通常讨论的是 OceanBase。

**一句话主线**

OceanBase 架构演进的核心，不是简单地“从单机走向分布式”，而是经历了三次重心切换：

1. 从“分层式分布式数据库”走向“对等节点一体化数据库”。
2. 从“为超大规模互联网金融场景优化”走向“兼顾中小规模、单机体验和云上弹性”。
3. 从“强一致分布式 OLTP”扩展到“HTAP、云原生、AI/向量/混合检索等统一数据底座”。

从系统架构师视角看，它的演进本质是在几个目标之间反复找平衡：**强一致、高可用、水平扩展、低延迟、成本、运维复杂度、租户隔离、云上弹性**。

**1. 早期 0.5：分层式架构，先解决“能分布式跑起来”**

早期 OceanBase 0.5 更像典型的分层式分布式数据库：

- SQL/查询层、存储层、更新层分离。
- 有类似 MergeServer、ChunkServer、UpdateServer 的角色划分。
- 读写路径经过不同组件。
- 数据分片后分布到多台机器。
- 通过多副本和日志保证可靠性。

这个阶段的架构目标是明确的：先把数据库从单机约束中释放出来，支撑大规模数据和高并发业务。

但从架构师视角看，问题也很典型：

- **写入扩展受限**：UpdateServer 这类集中写入角色容易变成瓶颈。
- **链路变长**：SQL 层和存储层分离，查询会引入更多网络跳转。
- **尾延迟难控**：金融交易类场景对 P99/P999 延迟敏感，网络抖动会放大体验问题。
- **角色复杂**：不同节点承担不同职责，扩容、故障切换和容量规划更复杂。

VLDB 论文对这段演进有比较清楚的描述：OceanBase 0.5 使用分层结构，后来因为单点写、SQL 与存储分离导致延迟等问题，转向了 1.0 到 3.0 的 P2P 架构。来源：[OceanBase Paetica, VLDB 2023](https://www.vldb.org/pvldb/vol16/p3728-xu.pdf)。

**架构判断**：  
0.5 阶段解决了“分布式化”的第一性问题，但还没有解决“分布式数据库能不能像单机数据库一样低延迟、简单、稳定”的问题。

**2. 1.0 到 3.0：对等 OBServer，走向 Shared-Nothing + 一体化内核**

1.0 到 3.0 是 OceanBase 架构的关键跃迁：从分层架构转为 **P2P 对等架构**。每个 OBServer 都包含 SQL、事务、存储能力。

这意味着：

- 节点角色趋于统一。
- SQL、事务、存储引擎在同一节点内协同。
- 数据按分区分布。
- 每个分区有多个副本。
- 副本之间通过 Paxos/Multi-Paxos 保证强一致和高可用。
- OBProxy/Driver 负责路由请求到合适节点。
- 多 Zone 部署支撑容灾。

官方文档也明确说 OceanBase 采用 shared-nothing 分布式集群架构，节点对等，依赖本地存储；集群跨 Zone 部署，每个 OBServer 进程承载数据库实例运行，且支持多租户资源隔离。来源：[OceanBase System Architecture V4.0 Docs](https://en.oceanbase.com/docs/common-oceanbase-database-10000000000829641)。

这一阶段的核心架构收益是：

- **横向扩展能力增强**：增加 OBServer 就能扩展计算和存储。
- **高可用内生化**：不依赖传统主备复制，而是用 Paxos 作为一致性和故障切换基础。
- **延迟路径缩短**：SQL、事务、存储在一个 OBServer 内聚合，减少跨层 RPC。
- **架构同构化**：节点职责统一，扩容和调度更容易自动化。
- **业务连续性更好**：多副本、多 Zone、自动选主支撑金融级容灾。

从系统架构角度，这一步很关键：OceanBase 没有走“计算层 + 存储层完全分离”的纯粹分层路线，而是把分布式数据库做成 **单节点内一体化、集群间 shared-nothing** 的形态。

可以把它理解成：

`​``text
单机内：SQL + 事务 + 存储紧耦合，追求低延迟
集群间：数据分区 + Paxos 副本，追求扩展和高可用
`​``

**架构判断**：  
1.0 到 3.0 是 OceanBase 成为真正分布式关系数据库的阶段。它用 shared-nothing 对等架构换来了强扩展、高可用和低延迟之间较好的平衡，尤其适合支付宝、淘宝这类超大规模交易场景。

**3. 3.0 阶段：大规模 OLTP 能力成熟，但资源下限偏高**

OceanBase 3.0 的标志是：在大规模场景下证明了分布式 OLTP 的能力。

公开论文提到，OceanBase 3.0 在 TPC-C 测试中体现出很强的水平扩展能力，从少量节点扩展到大规模集群，吞吐随节点增加呈现良好扩展。来源仍可参考：[OceanBase Paetica, VLDB 2023](https://www.vldb.org/pvldb/vol16/p3728-xu.pdf)。

但架构师不能只看峰值能力，还要看“适用边界”。

3.0 的问题是：

- 分区、日志流、事务和高可用粒度绑定较紧。
- 在超大规模场景没问题，但在中小规模部署中，固定架构开销偏高。
- 对很多企业来说，数据库集群未必一上来就是几十台、几百台机器。
- 如果小规模场景也要承担完整分布式架构成本，性价比会下降。
- 单机场景、三节点小集群、混合负载场景的体验还需要优化。

这就是典型的架构张力：  
**为极限规模设计的架构，未必天然适合中小规模普惠化部署。**

**架构判断**：  
3.0 证明了“强一致分布式数据库可以支撑超大规模核心交易”，但下一步问题变成：“能否让同一套架构也适合普通企业、云环境、小规格租户和单机场景？”

**4. 4.0：Paetica，一体化单机与分布式架构**

4.0 是 OceanBase 架构演进中的另一个重要转折。论文称其为 **stand-alone and distributed integrated architecture**，也就是“单机与分布式一体化架构”。来源：[OceanBase Paetica, VLDB 2023](https://www.vldb.org/pvldb/vol16/p3728-xu.pdf)。

4.0 的关键变化之一是引入 **动态日志流**，把原来绑定较紧的几个概念拆开：

- 存储分片粒度
- 事务处理粒度
- 高可用复制粒度
- Paxos 日志流粒度

以前一个分区往往对应一套较重的事务和高可用机制；4.0 之后，多个存储分片可以共享一个事务日志流和高可用服务。

这带来的架构价值很大：

- **降低小规模部署成本**：小集群不需要维护过多日志流。
- **提升单机/小规格性能**：减少分布式元数据和复制开销。
- **更适合中小企业**：不再只有超大规模业务才能享受 OceanBase。
- **兼顾纵向扩展与横向扩展**：单机内充分利用 CPU、内存、磁盘；集群间继续分区扩展。
- **租户粒度更灵活**：多租户资源隔离能力进一步成为架构核心。

可以把 4.0 的架构哲学概括为：

`​``text
不是让所有场景都付出分布式代价，
而是在需要分布式时自然扩展，
在单机/小集群时尽量接近单机数据库效率。
`​``

这对架构师非常重要。因为数据库产品一旦进入企业通用市场，不能只回答“最大能跑多大”，还必须回答：

- 最小能多轻？
- 运维能多简单？
- 单机性能是否有竞争力？
- 三节点是否值得部署？
- 多租户是否隔离可靠？
- 云上资源是否能弹性使用？

**架构判断**：  
4.0 的核心不是继续堆分布式能力，而是把分布式能力“内化”为可按需使用的能力，让同一套内核覆盖单机、小集群、大集群、多租户和云环境。

**5. 存储引擎演进：从分布式 LSM 到云原生共享存储探索**

OceanBase 的存储层长期采用 LSM-Tree 思路，典型结构包括 MemTable、SSTable、Compaction、Redo/Commit Log 等。官方 V2 文档也提到其存储引擎采用基于 LSM-Tree 的架构，基线数据和增量数据分别存储在 SSTable 和 MemTable 中。来源：[OceanBase System Architecture V2.2 Docs](https://en.oceanbase.com/docs/enterprise-oceanbase-database-en-10000000000383206)。

传统 shared-nothing 下，每个 OBServer 依赖本地盘，数据和日志分布在各节点。这样有几个好处：

- 本地读写延迟低。
- 节点自治性强。
- 故障域清晰。
- Paxos 多副本保证可靠性。

但在云上，shared-nothing 也有成本：

- 计算和存储耦合，扩缩容不够细。
- 数据迁移可能较重。
- 云对象存储便宜但延迟和语义不同。
- 多云环境下底层存储差异明显。

因此后续研究开始探索云原生共享存储架构。例如 2026 年公开的 OceanBase Bacchus 论文提出面向对象存储的 LSM 架构，让计算节点更接近无状态化，并通过 PALF 日志、异步后台服务和共享块缓存来支撑性能。来源：[OceanBase Bacchus, arXiv 2026](https://arxiv.org/abs/2602.23571)。

这代表了一个新方向：

`​``text
传统阶段：shared-nothing，本地存储，强一致多副本
云原生阶段：计算存储解耦，共享对象存储，缓存服务化，日志服务化
`​``

但这不是简单取代关系。系统架构师要看到两类架构的取舍：

| 架构方向 | 优点 | 代价 |
|---|---|---|
| Shared-Nothing + 本地盘 | 低延迟、自治性强、性能稳定 | 扩缩容和数据迁移成本较高 |
| Shared-Storage / 对象存储 | 弹性好、存储成本低、云上友好 | 缓存、日志、读放大、尾延迟控制更复杂 |

**架构判断**：  
OceanBase 的后续演进方向，不只是“数据库内核优化”，而是逐渐变成“数据库内核 + 云基础设施 + 缓存/日志/对象存储协同设计”。

**6. 从 OLTP 到 HTAP，再到 AI 数据底座**

早期 OceanBase 的核心标签是分布式 OLTP：强一致、高并发、金融级高可用。

之后架构目标逐步扩展：

- OLTP：核心交易、支付、订单。
- HTAP：同一系统内兼顾事务和分析。
- 多租户：一套集群服务多个业务/客户。
- 云数据库：弹性、自动化、资源池化。
- AI/混合检索：关系、全文、向量、JSON、GIS 等统一查询能力。

OceanBase 官网当前产品定位也已经从单纯分布式事务数据库扩展为面向 transactional、analytical 和 AI workloads 的统一分布式数据库。来源：[OceanBase Product Overview](https://en.oceanbase.com/product/overview)。

从系统架构师视角，这意味着它的架构边界在变大：

`​``text
过去：如何把关系数据库做成强一致分布式 OLTP
现在：如何把 OLTP、OLAP、搜索、向量、AI 工作负载放进一个统一数据平台
`​``

这会带来新的架构压力：

- 行存与列存/分析执行如何协同。
- 事务写入和分析扫描如何隔离资源。
- 向量索引、全文索引和关系索引如何统一优化。
- SQL 优化器如何处理混合查询。
- 多租户下如何做 CPU、内存、IO、缓存隔离。
- 云上如何在成本和性能之间动态调度。

**架构判断**：  
OceanBase 的演进已经从“数据库高可用架构”进入“统一数据平台架构”阶段。难点不再只是 Paxos、分片和事务，而是不同类型工作负载在一个内核里的资源治理和执行优化。

**7. 总结成架构演进路线**

可以用这条线概括：

`​``text
0.5：分层式分布式数据库
  目标：先解决分布式扩展
  问题：写入瓶颈、链路长、尾延迟难控

1.0-3.0：P2P 对等 OBServer + Shared-Nothing
  目标：强一致、高可用、水平扩展、低延迟
  手段：SQL/事务/存储一体化，分区，多副本，Paxos，多 Zone

3.0：超大规模 OLTP 成熟
  目标：支撑金融级核心交易和极大规模并发
  问题：中小规模部署成本和资源下限偏高

4.0：单机与分布式一体化 Paetica
  目标：同时服务单机、小集群、大集群、多租户
  手段：动态日志流，分区与日志流解耦，降低小规模开销

后续：云原生与统一工作负载
  目标：HTAP、云上弹性、AI/向量/混合检索
  手段：计算存储解耦探索、共享缓存/日志服务、对象存储适配
`​``

**从架构师角度的核心洞察**

OceanBase 的演进不是单点技术升级，而是系统边界不断变化后的架构重构。

第一阶段，它要回答：  
**数据库如何突破单机容量和可用性限制？**

第二阶段，它要回答：  
**分布式数据库如何做到低延迟、强一致、高可用？**

第三阶段，它要回答：  
**极大规模架构如何下沉到普通企业、小集群、云上租户？**

第四阶段，它要回答：  
**一个数据库如何同时承载事务、分析、搜索、向量和 AI 工作负载？**

所以，如果从系统架构师视角评价 OceanBase/OceanDB 的主线，我会概括为：

> 它从“分布式能力优先”的数据库，逐步演进为“以统一内核承载多规模、多租户、多负载”的数据库平台；架构重点从节点分工、分片复制，转向资源隔离、负载融合、弹性成本和云原生协同。

<!-- /message -->
<!-- message:id=msg_719c44c7f50af115 role=user createdAt=2026-05-18T00%3A16%3A08.015Z -->

## User

0.5阶段uodate和查询分离，如何保证事务性？3.0如何在sn架构下做出高事务吞吐

<!-- /message -->
<!-- message:id=msg_f6acb5914b7e2894 role=assistant createdAt=2026-05-18T00%3A18%3A25.891Z runId=run_8e20a549d2982853 status=succeeded -->

## Assistant

可以把这两个问题放在一条主线上看：

> 0.5 是用“集中写入”换事务简单性；  
> 3.0 是把“集中写入”拆成大量分区 Leader 上的并行小事务，同时用 Paxos + 优化 2PC 保证强一致。

下面分开说。

**1. 0.5 阶段 update 和查询分离，事务性靠什么保证？**

先纠正一个容易误解的点：0.5 不是“查询层自己查、更新层自己改，然后彼此异步拼一下”。它更像：

`​``text
基准数据：ChunkServer，分布式，只读，多副本
增量数据：UpdateServer，集中写入，主备热备
SQL/合并：MergeServer，解析、路由、结果合并
`​``

也就是说，数据被拆成两部分：

`​``text
完整逻辑数据 = ChunkServer 上的基准数据 + UpdateServer 上的增量数据
`​``

公开的 0.4.2/0.5 早期文档里说得很直接：OceanBase 把数据按时间线分成“基准数据”和“增量数据”，所有修改先进入增量数据；基准数据只读，系统定期把增量合并进基准数据。读的时候，ChunkServer 读取基准数据后，还会从 UpdateServer 获取对应增量并融合成最终结果。来源：[OceanBase 0.4.2 描述文档](https://docs.huihoo.com/oceanbase/OceanBase-0.4.2-%E6%8F%8F%E8%BF%B0.pdf)。

事务性的关键在于：**所有写事务最终都由单个主 UpdateServer 执行**。

早期架构里，MergeServer 做 SQL 解析、Schema 检查、计划生成、必要的基准数据读取；但真正决定写事务是否成功、如何修改、何时提交的是 UpdateServer。对于读写事务，流程大致是：

`​``text
客户端
  -> MergeServer 解析 SQL，生成执行计划
  -> MergeServer 从 ChunkServer 读取需要的基准数据
  -> MergeServer 把执行计划 + 基准数据发给 UpdateServer
  -> UpdateServer 融合“基准数据 + 增量数据”
  -> UpdateServer 判断条件、加锁、写 redo、更新 MemTable
  -> 返回成功/失败
`​``

这点很重要。MergeServer 从 ChunkServer 拿到的基准数据可能不是“最终最新值”，因为最新修改在 UpdateServer 里。但 UpdateServer 才做最终判断，它会把 MergeServer 传来的基准数据和自己手里的增量数据融合后再执行事务逻辑。

举个例子：

`​``sql
update t1 set c1 = c1 + 1 where rowkey = 1;
`​``

如果 ChunkServer 上基准值是 `2012`，MergeServer 会把这个值和物理计划传给 UpdateServer。UpdateServer 再看自己 MemTable 里有没有对 `rowkey=1` 的更新、删除、插入记录，融合出当前逻辑值，然后执行加一，并把新修改写入 MemTable。早期文档里也用了类似例子说明这个流程。

所以 0.5 的事务性不是靠分布式 2PC，而是靠把写事务收敛到一个主 UpdateServer：

`​``text
跨行、跨表事务
  -> 从分布式问题
  -> 退化成单机事务管理问题
`​``

这就是它的核心取舍。

**原子性怎么保证？**

靠 UpdateServer 的 redo log + MemTable 提交流程。

UpdateServer 在更新 MemTable 之前先记录操作日志，并同步给备 UpdateServer。早期文档描述的更新流程是：

`​``text
1. 更新操作发送到备机
2. redo 日志写入主机硬盘
3. redo 应用到主机 MemTable
4. 返回客户端写入成功
`​``

也就是说，客户端看到提交成功前，主备已经具备恢复这笔事务所需的信息。主 UpdateServer 故障时，RootServer 只会选择日志足够新的备 UpdateServer 接管，避免把落后的备机提升为主机。这保证了“不丢已提交事务”。

**隔离性怎么保证？**

靠 MemTable 内的 MVCC + 行锁。

早期 UpdateServer 的 MemTable 是内存 B+Tree，并维护多版本操作链。写事务分两个阶段：

`​``text
预提交：
  锁住待更新行
  把操作追加到未提交操作链
  加入提交任务队列

提交：
  提交线程把日志追加到日志缓冲区
  同步备机并写主机日志
  日志成功后，把未提交操作链转入已提交操作链
  释放锁，返回客户端
`​``

读事务只读某个版本以前已经提交的修改，不读未提交修改。文档中也说明，每个写事务提交时生成事务版本；读事务只读取它之前提交的写事务修改。默认隔离级别偏向 `read committed`，并支持 `select ... for update` 显式锁行来满足更强一致要求。

可以总结为：

`​``text
0.5 的 ACID 来源：
Atomicity：UpdateServer 单点提交 + redo log
Consistency：主备同步后才返回成功，故障切换不丢已提交日志
Isolation：MVCC + 行锁 + 提交版本
Durability：主机日志落盘 + 备机同步
`​``

**那查询为什么还能事务一致？**

只读查询走 MergeServer/ChunkServer，但结果不是只看 ChunkServer 的旧基准数据。ChunkServer 会把基准数据和 UpdateServer 上的增量数据融合后返回。

概念上是：

`​``text
read(row, version)
  = baseline(row, version) from ChunkServer
  + delta(row, version) from UpdateServer
  - uncommitted changes
  - changes newer than read version
`​``

所以查询和更新虽然在物理角色上分离，但逻辑数据视图由“基准 + 增量 + 版本过滤”组成。只要 UpdateServer 的增量有序、可恢复、可按版本读取，就能给读请求提供一致快照。

**这个设计的优点和代价**

优点是事务实现简单，尤其是跨行跨表事务：

`​``text
不需要每个分片都参与 2PC
不需要每个 ChunkServer 都维护复杂写事务状态
写冲突集中在 UpdateServer 内处理
`​``

但代价也很明显：

`​``text
所有写入集中到主 UpdateServer
写吞吐被单点 CPU、内存、网络、日志能力限制
读路径也经常要访问 UpdateServer 获取最新增量
UpdateServer 增量过大时，读延迟和合并成本都会上升
`​``

这就是为什么 0.5 能比较好地支撑早期淘宝收藏夹这类“读多写少、基准数据巨大、近期增量可控”的场景，但不适合更高并发的核心交易写入场景。

官方架构演进文章也指出，0.5 的主要弱点就是写入由 UpdateServer 处理，属于单点写、多点读，无法继续支撑更高写并发；同时 SQL 层和存储层分离导致延迟难控。来源：[The architectural evolution of OceanBase Database](https://oceanbase.github.io/docs/blogs/showcases/architectural-evolution)。

**2. 3.0 如何在 shared-nothing 架构下做出高事务吞吐？**

3.0 的本质变化是：把 0.5 的“全局单写点”改成“每个分区一个写主，很多分区并行写”。

也就是：

`​``text
0.5：
  全集群一个主 UpdateServer 处理写入

1.0-3.0：
  数据拆成很多 partition
  每个 partition 有自己的 Paxos group
  每个 partition 有自己的 leader
  不同 partition leader 分布在不同 OBServer 上
  所有 OBServer 都有 SQL + 事务 + 存储能力
`​``

这正是 shared-nothing 架构的价值：每台机器有自己的 CPU、内存、磁盘、日志和事务执行能力，没有共享存储或全局写节点。公开文档描述 3.x/4.x 的 SN 架构时也强调：每个 OBServer 都有 SQL 引擎、事务引擎和存储引擎，节点对等，数据和事务 redo log 存在本地文件中；多副本之间用 Paxos 保证一致性。来源：[OceanBase Database architecture](https://en.oceanbase.com/docs/common-oceanbase-database-10000000001971022)。

**高吞吐的第一层：事务尽量本地化**

SN 架构下，事务吞吐高不高，关键不是“有没有分布式事务”，而是：

`​``text
大多数事务是不是能落在一个分区 leader 或同一台 OBServer 上？
`​``

如果一个事务只访问一个分区 leader：

`​``text
客户端 -> OBProxy -> 分区 leader 所在 OBServer
OBServer 本地执行 SQL/事务/存储
redo 通过该分区 Paxos group 同步多数派
提交
`​``

这个路径里没有跨分区 2PC，只有这个分区自己的复制协议。

如果事务访问多个分区，但这些分区 leader 在同一台 OBServer 上，也可以减少网络跳转和协调成本。

所以 3.0 的高吞吐非常依赖：

`​``text
合理分区
合理主键/分区键设计
表组/亲和放置
OBProxy 路由到 leader
leader balance 把不同分区 leader 分散到多台机器
`​``

OceanBase 文档也提到，RootService 会做副本和 leader 均衡；不同分区 leader 可以聚集到同一 OBServer 以减少分布式事务，也可以分散到多个 OBServer 以充分利用资源、提高吞吐。来源：[Overview of data distribution](https://en.oceanbase.com/docs/enterprise-oceanbase-database-en-10000000000847988)。

**高吞吐的第二层：每个分区独立 Paxos，而不是全局共识**

这是理解 SN 高吞吐的关键。

OceanBase 不是让全库所有事务进入一个全局 Paxos 日志。它是：

`​``text
partition A -> Paxos group A
partition B -> Paxos group B
partition C -> Paxos group C
...
`​``

每个分区独立复制 redo log。一个事务只需要让它涉及的分区日志持久化到对应 Paxos group 的多数派。

这意味着：

`​``text
单分区事务：
  只打一个 Paxos group

多分区事务：
  打多个 Paxos group + 2PC 协调

全局吞吐：
  来自大量 Paxos group 在大量 OBServer 上并行提交
`​``

所以 shared-nothing 并不天然导致低吞吐。低吞吐通常来自“跨分区事务比例过高”或“热点 leader 集中”。如果分区和业务模型匹配，SN 架构反而能把事务压力切成大量独立并行流。

VLDB 2022 论文《OceanBase: A 707 Million tpmC Distributed Relational Database System》也说明：表分区是数据分布、负载均衡和 Paxos 同步的基本单元；每个分区通常对应一个 Paxos group。来源：[VLDB 2022 paper](https://vldb.org/pvldb/vol15/p3385-xu.pdf)。

**高吞吐的第三层：Paxos + 2PC 被专门优化**

跨分区事务还是需要分布式提交。但 OceanBase 3.0 的做法不是传统 2PC 原样搬过来，而是做了 Paxos-based 2PC 优化。

传统分布式数据库里，2PC 的问题是：

`​``text
协调者要持久化状态
参与者也要持久化 prepare/commit 状态
协调者故障恢复复杂
日志同步轮次多
延迟高
`​``

OceanBase 的论文里提出 OceanBase 2PC，核心是把 Paxos 引入 2PC，并减少协调者持久状态。论文说，相比传统 2PC，OceanBase 2PC 把 Paxos 同步次数从三次减少到两次，并把 2PC 延迟进一步压到一个 Paxos 同步的量级。来源同上：[VLDB 2022 paper](https://vldb.org/pvldb/vol15/p3385-xu.pdf)。

概念上可以理解为：

`​``text
传统 2PC：
  coordinator log
  participant prepare log
  participant commit log
  多轮持久化与网络交互

OceanBase 2PC：
  让参与者 Paxos 日志承担事务状态持久化
  coordinator 状态可由参与者状态恢复
  减少协调者持久化路径
  降低跨分区事务提交延迟
`​``

这不等于跨分区事务没有成本。它只是把不可避免的成本压低，并让故障恢复更自动化。

**高吞吐的第四层：OBProxy 透明路由，减少跨机事务**

OceanBase 3.0 的高吞吐不是只靠内核，还靠访问层配合。

OBProxy 知道分区位置，尽量把请求转发到正确的 OBServer，尤其是 leader 所在节点。这样可以减少：

`​``text
请求打到错误节点再转发
SQL 执行节点和数据 leader 分离
不必要的跨机 RPC
不必要的分布式事务
`​``

VLDB 2022 论文在解释 TPC-C New-Order 可扩展性时明确列出原因，其中第一条就是通过无状态 OBProxy 的透明转发大幅减少分布式事务，并优化本地事务；还包括 OceanBase 2PC、存储过程、优化的 LSM-tree、快速 SQL 参数化等。来源：[VLDB 2022 paper](https://vldb.org/pvldb/vol15/p3385-xu.pdf)。

**高吞吐的第五层：LSM/MemTable 写路径减少随机写**

事务吞吐最终会落到写路径上。OceanBase 的存储引擎是 LSM-tree 风格：

`​``text
DML 先写 MemTable
redo log 保证可恢复
MemTable 到一定大小后转储成 SSTable
后台 compaction 合并
`​``

这种写路径对 OLTP 很友好：

`​``text
前台写入主要是 redo append + 内存结构更新
避免频繁原地随机更新磁盘页
后台再做合并和压缩
`​``

VLDB 2022 论文也提到，OceanBase 的数据分为 SSTable 中的静态基准数据和 MemTable 中的动态增量数据；DML 先写 MemTable，查询时合并 SSTable 和 MemTable 结果；同时用 Block Cache、Row Cache、Bloom Filter 等优化小查询，使 OLTP 小查询性能接近内存数据库。来源：[VLDB 2022 paper](https://vldb.org/pvldb/vol15/p3385-xu.pdf)。

**高吞吐的第六层：工程上规避 TPC-C 的分布式热点**

TPC-C 是很适合暴露分布式数据库弱点的测试，因为 New-Order、Payment 等事务容易产生跨仓库、热点表、外键访问等问题。

OceanBase 做了几类典型优化：

`​``text
1. 大表按业务键分区，分散到所有数据节点
2. 高频小表 ITEM 做同步复制表，避免每次远程访问
3. OBProxy 路由减少分布式事务
4. 存储过程减少客户端和数据库之间的往返
5. 快速 SQL 参数化和 plan cache 适配 OLTP 短 SQL
6. 后台 compaction 资源隔离，降低吞吐抖动
7. 多类型副本降低资源成本
`​``

VLDB 论文披露，2020 TPC-C 测试中 OceanBase 集群有 1,557 台服务器，其中 1,554 台数据服务器，分布在 3 个 zone；除 ITEM 表外，其他 TPC-C 表都分区并分布到所有数据服务器，ITEM 表被配置为同步复制表以避免远程访问。测试结果显示，tpmC 随数据节点数从 3、9、27、81、243、510 到 1554 基本线性上升，最终达到 707 million tpmC。来源：[VLDB 2022 paper](https://vldb.org/pvldb/vol15/p3385-xu.pdf)。

**从架构师视角看，两代设计的本质差异**

可以这样对比：

| 维度 | 0.5 | 3.0 |
|---|---|---|
| 写入入口 | 单主 UpdateServer | 每个分区 leader 都可写 |
| 事务模型 | 集中式事务管理 | 分区内本地事务 + 跨分区 2PC |
| 一致性 | 主备同步 redo，单点提交 | Paxos group 多数派复制 |
| 查询 | ChunkServer 基准 + UpdateServer 增量融合 | OBServer 本地 SQL/事务/存储一体化 |
| 扩展方式 | 读扩展较好，写扩展受限 | 读写都可随分区和 OBServer 扩展 |
| 主要瓶颈 | UpdateServer 单点 | 跨分区事务比例、热点分区、leader 分布、Paxos 延迟 |
| 适合场景 | 读多写少、增量可控 | 高并发 OLTP、金融交易、大规模在线业务 |

一句话总结：

`​``text
0.5 保证事务性的方式是“把所有写事务集中到一个强一致的 UpdateServer”，
所以事务简单，但写扩展差。

3.0 做高事务吞吐的方式是“把数据切成大量独立 Paxos 分区，
让事务尽量本地化，让分区 leader 在大量 OBServer 上并行提交”，
再通过 OBProxy 路由、OceanBase 2PC、LSM 写路径、存储过程和资源隔离把延迟压低。
`​``

所以 3.0 的高吞吐不是违反 shared-nothing 的结果，恰恰是把 shared-nothing 用到了极致：**没有全局写点，没有共享存储仲裁点，只有大量可并行推进的分区事务流**。

<!-- /message -->
