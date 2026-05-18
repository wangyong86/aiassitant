---
id: "ses_f4f95088795bfb37"
title: "产品定位"
createdAt: "2026-05-16T02:14:19.547Z"
updatedAt: "2026-05-17T00:53:07.897Z"
settings: {}
runs: [{"id":"run_03218565d053b7e3","status":"succeeded","startedAt":"2026-05-16T02:16:33.374Z","finishedAt":"2026-05-16T02:18:05.152Z","userMessageId":"msg_cf0aaaddd674b791"}]
---

# 产品定位

<!-- message:id=msg_cf0aaaddd674b791 role=user createdAt=2026-05-16T02%3A16%3A33.371Z -->

## User

vast在一个更快的存储之上增加了aios能力这部分市场buyin吗？请结合客户和营收来分析这部分的真正前景

<!-- /message -->
<!-- message:id=msg_1a8f2e2daaa38ffd role=assistant createdAt=2026-05-16T02%3A18%3A05.152Z runId=run_03218565d053b7e3 status=succeeded -->

## Assistant

有买单，但市场买单的核心不是“AI OS”这四个字本身，而是它把“更快的存储”继续往上做成了“更靠近 GPU 的数据与编排层”，也就是把存储、数据库、事件、治理、推理工作流尽量收敛到一套平台里。

我的判断是：

- **客户侧，是真买。**
  - VAST 官网当前直接展示的客户/标识包括 xAI、Adobe、CVS、CrowdStrike、Lowe’s、ServiceNow、CoreWeave 等。这个信号说明它已经不只是“卖闪存”，而是在 AI 基础设施和企业数据平台里拿到了一批可见度很高的客户。
  - 2025 年 11 月 6 日，VAST 公开宣布与 CoreWeave 签下 **11.7 亿美元** 商业合作，把 VAST AI OS 作为 CoreWeave 的主要数据底座。这不是概念验证级别，而是长周期大合同。
  - 2026 年 4 月 22 日的融资公告里，VAST 还点名了 Mistral AI、JPMorganChase、Crusoe 等客户/合作方对其平台价值的背书。

- **营收侧，增长也是真增长，但要分清是“收入”还是“承诺”。**
  - VAST 2023 年官方披露过 **ARR 超过 1 亿美元**。
  - Reuters 被 TechCrunch 转引的报道说，VAST 到 **2025 年 1 月** 已到 **2 亿美元 ARR**，并预计之后继续快速增长。
  - VAST 在 **2026 年 4 月 22 日** 的官方融资稿里说，它已经超过 **40 亿美元累计 bookings**，并在上一财年结束时拥有 **超过 5 亿美元 CARR**，同时保持正经营利润和正自由现金流。
  - 这里要注意，**CARR/ARR 不是 GAAP revenue**，尤其对私营公司来说，它更像“已签或可预期的经常性收入跑道”，不是审计口径的实际收入。因此它是强信号，但不能当成完全等同于营收。

- **所以，市场 buy-in 的本质是什么？**
  - 不是大家真的在买“一个新 OS 的类比”，而是买它解决的三个痛点：
    - 数据离 GPU 太远，导致训练和推理慢
    - AI 管道太碎，存储、向量库、编排、治理分散
    - 企业数据和受监管数据进入 agentic AI 后，安全和权限复杂度暴增
  - 也就是说，买的是 **AI data infrastructure / control plane**，不是单纯的 storage appliance。
  - 这个角度上，VAST 的 AI OS 叙事是成立的，但它更像是把既有强项往上叠层，扩大可卖的软件范围，而不是从零发明一个全新的大众软件类别。

- **真正前景，我会分成两层看。**
  - **短中期前景不错**：只要 AI 训练、推理、RAG、KV cache、agent 工作流还在受“数据移动和治理”约束，VAST 就有持续放大合同规模的空间。CoreWeave 这种 AI 云客户，尤其适合把它从“存储供应商”变成“平台型基础设施供应商”。
  - **长期上限取决于能否上探到更高层**：如果 VAST 能把数据层、治理层、agent runtime 变成企业 AI 的默认底座，它就不只是 storage vendor，而会更接近 Databricks / Snowflake 这种平台逻辑。
  - **但它仍然有边界**：如果客户只把它当成“更快、更贵、但仍然是存储”，那 AI OS 的命名溢价就会受限。

- **风险也很明确。**
  - 第一，很多收入和增长目前仍然高度绑定 AI 基建周期，尤其是 neocloud、模型训练和大规模推理。
  - 第二，TechTarget 的独立报道提到，市场方向确实在往更紧耦合的 AI stack 走，但一些 DPU/集成式架构到底能带来多大真实成本节省和性能提升，**仍需要更多真实工作负载验证**。
  - 第三，VAST 现在的叙事很强，但独立、可审计的收入拆分并没有公开，外界只能通过 ARR、CARR、bookings 和大合同来判断质量。

**一句话结论：**
VAST 的 AI OS 不是纯包装，它背后有真实客户、真实大单和真实收入增长支撑；但市场买入的主要是“AI 数据底座 + 邻近算力 + 治理/编排”这套基础设施价值，不是“操作系统”这个名词本身。真正的天花板，取决于它能否持续把 storage 预算变成 platform 预算。

参考链接：
- [VAST 2026 融资与 30B 估值公告](https://www.vastdata.com/press-releases/vast-series-f-financing-at-30-billion-valuation)
- [VAST AI OS 发布公告](https://www.vastdata.com/press-releases/vast-data-unveils-ai-os-for-thinking-machines)
- [VAST 官网客户展示页](https://www.vastdata.com/)
- [CoreWeave 11.7 亿美元合作公告](https://www.vastdata.com/press-releases/vast-data-secures-commercial-partnership-deal-with-coreweave)
- [TechCrunch 对 2025 年 ARR / 客户的报道](https://techcrunch.com/2025/06/10/ai-storage-platform-vast-data-aimed-for-25b-valuation-in-new-round-sources-say/)
- [TechTarget 对 2026 AI OS 更新的独立分析](https://www.techtarget.com/searchstorage/news/366639519/Vast-Data-services-speed-up-AI-workloads-add-intelligence)
- [Futuriom 对 VAST AI OS 的独立解读](https://www.futuriom.com/articles/news/vast-data-declares-itself-an-ai-operating-system/2025/05)

<!-- /message -->
