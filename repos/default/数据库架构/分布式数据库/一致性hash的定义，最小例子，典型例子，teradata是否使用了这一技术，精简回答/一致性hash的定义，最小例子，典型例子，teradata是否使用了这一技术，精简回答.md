---
id: "ses_dcd2c9ef662bc0ba"
title: "一致性hash的定义，最小例子，典型例子，teradata是否使用了这一技术，精简回答"
createdAt: "2026-05-21T10:16:47.674Z"
updatedAt: "2026-05-21T10:22:47.537Z"
---

# 一致性hash的定义，最小例子，典型例子，teradata是否使用了这一技术，精简回答

<!-- message:id=msg_f629154e5a5f864d role=user createdAt=2026-05-21T10%3A16%3A47.674Z -->

## User

一致性hash的定义，最小例子，典型例子，teradata是否使用了这一技术，精简回答

<!-- /message -->
<!-- message:id=msg_6dfdfef10ef4c924 role=assistant createdAt=2026-05-21T10%3A17%3A13.861Z runId=run_dff92dc38418ae46 status=succeeded -->

## Assistant

**一致性 Hash 定义**：把数据 key 和节点都映射到同一个 hash 环上；某个 key 顺时针找到的第一个节点负责它。节点增删时，只影响相邻一小段 key，避免普通 `hash(key) % N` 在 `N` 变化时大规模重分布。

**最小例子**：  
环上有节点 `A=20, B=60, C=90`。  
数据 `k1` hash 到 `55`，顺时针下一个节点是 `B`，所以 `k1 -> B`。  
如果新增节点 `D=50`，只有 `(20,50]` 这一段数据改到 `D`，其他大部分不变。

**典型例子**：  
分布式缓存，如 Memcached / Redis Cluster 类场景：用户 `userId` 通过一致性 hash 定位到某台缓存节点。加机器或下线机器时，只迁移少量缓存 key，缓存雪崩风险较低。常配合“虚拟节点”改善负载均衡。

**Teradata 是否使用了这一技术**：  
严格说，Teradata 使用的是**基于 Primary Index 的 hash 分布机制**：对主索引值计算 row hash，再通过 hash map 把 hash bucket 映射到 AMP。官方文档描述的是“row hash + hash map + AMP”的数据分布方式。  
它和一致性 hash 思想相似，都是用 hash 做分布；但通常不称为经典“一致性 hash 环”算法。更准确地说：**Teradata 使用 hash-based distribution，不是典型的一致性 hash ring 实现**。

参考：Teradata 官方文档提到 Vantage 会基于 Primary Index 计算 hash，并用 hash map 决定 row 属于哪个 AMP：<https://docs.teradata.com/r/Enterprise_IntelliFlex_VMware/SQL-Fundamentals/Database-Objects/Indexes/Index-Hash-Mapping>

<!-- /message -->
<!-- message:id=msg_227f9b0a3af11cb0 role=user createdAt=2026-05-21T10%3A22%3A14.502Z -->

## User

具体说下teradata的hash实现，结合扩缩容。精简回答

<!-- /message -->
<!-- message:id=msg_366ff8d95a4235fe role=assistant createdAt=2026-05-21T10%3A22%3A47.537Z runId=run_3fc7eb733ffd0ee9 status=succeeded -->

## Assistant

Teradata 的 hash 分布可以理解为：**不是 `hash % AMP数`，而是 `Primary Index -> Row Hash -> Hash Bucket -> Hash Map -> AMP`**。

**1. 写入时怎么定位 AMP**

假设表有 Primary Index：

`​``sql
CREATE TABLE orders (
  order_id BIGINT,
  customer_id BIGINT,
  amount DECIMAL(10,2)
)
PRIMARY INDEX (customer_id);
`​``

插入一行时：

`​``text
customer_id
   -> HASHROW(customer_id) 得到 32-bit row hash
   -> HASHBUCKET(row hash) 取高位得到 bucket
   -> hash map 查询 bucket 属于哪个 AMP
   -> 该 AMP 存这行
`​``

Teradata 官方文档说明：row hash 是 **32-bit**；根据系统设置，通常用高 **20 bits** 得到 hash bucket，新系统常见是 **1,048,576 个 bucket**，老系统可能是 **65,536 个 bucket**。这些 bucket 再尽量均匀映射到 AMP。来源：Teradata Index Hash Mapping 文档：<https://docs.teradata.com/r/Enterprise_IntelliFlex_VMware/SQL-Fundamentals/Database-Objects/Indexes/Index-Hash-Mapping>

**2. 为什么不是普通取模**

普通做法：

`​``text
AMP = hash(customer_id) % AMP_count
`​``

如果 AMP 数从 100 变成 120，大量数据的 `%` 结果都会变，迁移量很大。

Teradata 多了一层 **hash map**：

`​``text
bucket -> AMP
`​``

扩容时主要改的是 bucket 到 AMP 的映射，不是改 hash 函数本身。

**3. 扩容时发生什么**

例如原来有 4 个 AMP：

`​``text
bucket 0-249999   -> AMP1
bucket 250000-499999 -> AMP2
bucket 500000-749999 -> AMP3
bucket 750000-999999 -> AMP4
`​``

扩到 5 个 AMP 后，Teradata 会生成/使用新的 map，让 bucket 更均匀：

`​``text
部分 bucket 从 AMP1/2/3/4 迁到 AMP5
`​``

然后通过 reconfiguration / MAPS 相关机制，把受影响 bucket 对应的数据重新分布到新 AMP。结果是：

`​``text
hash 值不变
bucket 不变
bucket -> AMP 的映射变了
数据按新 map 迁移
`​``

所以扩容不是全表按新 AMP 数重新 hash，而是按 hash bucket / map 做再平衡。

**4. 缩容时发生什么**

缩容类似反向操作：

`​``text
要下线的 AMP 上的 bucket
   -> 重新映射到保留 AMP
   -> 对应数据迁走
`​``

缩容后，剩余 AMP 接管原 AMP 的 bucket，数据重新分布到存活 AMP 上。

**5. 和一致性 hash 的关系**

相似点：

`​``text
都避免 hash % N 带来的大规模重分布
都通过中间映射层减少扩缩容影响
`​``

不同点：

`​``text
一致性 hash：通常是 hash ring，key 顺时针找节点
Teradata：是固定 hash bucket + hash map + AMP
`​``

所以更准确地说：**Teradata 使用 hash bucket / hash map 的分布式数据定位机制；它有一致性 hash 的扩缩容思想，但不是典型的一致性 hash 环。**

<!-- /message -->
