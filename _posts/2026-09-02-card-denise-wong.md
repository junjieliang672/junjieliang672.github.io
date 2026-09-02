---
layout: post
title: "人物 · Denise Wong"
date: 2026-09-02
description: "新加坡数据与 AI 治理的分管官员，让 GenAI 应用的安全测试有了可交差的规程"
categories: card
tags: [llm-security, card, person, policy]
giscus_comments: false
---
<img src="/assets/img/radar/denise-wong.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**新加坡数据与 AI 治理的分管官员，让 GenAI 应用的安全测试有了可交差的规程**

- **身份**：新加坡 IMDA 助理首席执行官（Data Innovation & Protection Group）；2026 年 4 月起任 PDPC 专员
- **主页**：[https://oecd.ai/en/community/denise-wong](https://oecd.ai/en/community/denise-wong)
- **从哪读起**：直接读 [Global AI Assurance Pilot 报告的 Executive Summary](https://assurance.aiverifyfoundation.sg/report/executive-summary/)，十分钟能看清一个 GenAI 应用在合规语境下会被怎么测、测出来什么。
- **成名作**：她分管的 IMDA/AI Verify Foundation 这条线做出了 [Global AI Assurance Pilot](https://assurance.aiverifyfoundation.sg/report/executive-summary/)——2025 年 3–5 月让 17 家机构、10 个行业的真实 GenAI 应用交给第三方测试公司实测，5 月底公开复盘，是第一份成规模的「部署方的应用被外部测出了什么」的行业报告。

| 时期 | |
|---|---|
| 2026-04 起 | 新加坡个人数据保护委员会（PDPC）专员，此前为副专员 |
| 近年 | IMDA 助理首席执行官，负责 Data Innovation & Protection Group（数据与 AI 治理） |
| 早年 | 新加坡总检察署、最高法院 law clerk |

## 先把权力边界说清楚：她管的是数据 + AI 治理

她是 IMDA 里 Data Innovation & Protection Group 的助理首席执行官，2026 年 4 月 1 日起兼任 PDPC 专员（此前是副专员）。

这个组合决定了她这条线上产出的形态。新加坡没有 AI 专法，她手上真正有强制力的是 PDPA 下的个人数据执法；AI 那一半没有罚则，只有 advisory guidelines、开源工具、和一个行业联盟（AI Verify Foundation）。也就是说，IMDA 出的任何一套 LLM 测试规程，都不能靠「不做就罚你」推下去，只能靠部署方觉得好用、觉得能拿去给客户和监管看。这一点直接解释了后面几件事为什么长成那个样子。

## 把测试对象从模型换成应用

AI Verify Foundation 的 Project Moonshot 是个开源的 LLM 评测工具包，把 benchmark 跑分和红队攻击放进同一个工具里，有 Python 库也有网页界面，还能接进 CI/CD 流水线——意思是你改一次 system prompt，流水线自动把整套攻击集和题库重跑一遍，出一份可分享的报告，而不是靠人临时想几个问法试试。

配套的是 IMDA 2025 年 5 月发的 Starter Kit，把 LLM 应用的风险切成四类，每一类都给了怎么测：hallucination（生成事实错误或无依据的内容）、undesirable content（对个人或公众造成伤害的内容）、data disclosure（把不该说的个人或企业敏感信息漏出来）、vulnerability to adversarial prompts（被刻意构造的输入撬开）。

关键的口径选择在这里：测的对象是「模型 + 检索 + system prompt + 护栏」组成的那个完整应用，不是裸模型。同一个基座模型，A 公司接了内部 HR 数据库、B 公司接了病历，两边的 data disclosure 风险完全不是一回事，测裸模型看不出来。代价是它的题目看上去比学术 jailbreak 基准粗糙——它服务的不是想发论文的人，是要向客户或监管方交一份测试材料的部署方。

## 多语言红队挑战：测的是文化偏见，不是越狱能力

2024 年 11–12 月，IMDA 与 Humane Intelligence 合办了一次亚太范围的红队活动，据 IMDA 的公开材料，来自 9 个国家的 350 多名参与者测了 4 个模型（Aya、Claude、Llama、SEA-LION），产出 1000 多条成功 prompt，评测报告 2025 年 2 月发布。

判定口径要说清楚，不然容易读错：它找的不是「怎么让模型教我造炸弹」，而是模型在非英语、本地文化语境下的刻板印象和护栏失效——比如同一个带有族群刻板印象的请求，用英文提出会被拒绝，换成某种东南亚语言就顺利答了；又比如模型对本地宗教、种族关系的默认叙述踩了当地的雷。参与者是各国的领域专家和母语者，因为这类失效只有本地人认得出来。

它真正想输出的是那套能被别的司法辖区照搬的组织方法——怎么招募母语专家、怎么给「这条算不算有害」定标注规则、怎么把 1000 条散乱的成功 prompt 归并成可比的类别。把它当成一张 jailbreak leaderboard 去看，会得出「攻击手法很平常」的结论，那是看错了东西。

## Assurance Pilot：她这条线上最可能改变别人做法的一步

2025 年 2 月在巴黎 AI Action Summit 上发起，3 到 5 月间，17 家机构、覆盖 HR、医疗、金融等 10 个行业的 GenAI 应用，交给专门做 GenAI 测试的第三方公司实测，5 月 29 日发报告。

对做评测的人有直接影响的结论有两条。一是 LLM-as-judge：报告说它是主流做法之一，但必须要有大量人工校准和持续监控，否则会 silent failure——判分模型自己出错，而你从聚合分数上完全看不出来；而且不少场景下更便宜更简单的判法（正则、结构化校验、检索证据比对）就够用，不必上 LLM 判官。二是人类专家在每一个环节都去不掉：定义这个应用的风险、设计测什么、解读结果，都不是能自动化的部分。这对当时「一套自动化评测跑完就算测过了」的预期是个泼冷水。

后续动作是往两个方向推：一套「测什么、怎么测」的行业标准，和一套第三方测试机构的认证框架。如果认证这条落地，第三方 AI 审计就第一次有了准入门槛——谁有资格签这个字。这是她这条线上影响最大的一步，也是最没定局的一步：认证框架能不能被新加坡以外的市场接受，目前没有答案。

## 为什么是新加坡在做

定位很清楚：不做前沿模型，只做中立的测试基础设施。中美双方都不完全采信对方实验室的评测结论，而一个不训练前沿模型的第三方出的测试规程，双方接受起来成本都低一些。本地语料显示她参与了《The 2026 Singapore Consensus on Global AI Safety Research Priorities》（arXiv [2608.14611](https://arxiv.org/abs/2608.14611)）——此处仅据语料，未独立核实她是署名作者还是参会方。

另一条线是人。据媒体报道，2026 年 7 月 22 日她以 PDPC 专员兼 IMDA 助理首席执行官身份与 IAPP 签了一份三年备忘录，做 AI 治理人才培训。把「谁有资格做这个测试」推向职业认证，和 Assurance Pilot 里那条「人类专家去不掉」的结论是同一件事的两头。

**已核实来源**

- <https://www.imda.gov.sg/resources/press-releases-factsheets-and-speeches/press-releases/2026/ms-denise-wong-appointed-as-commissioner-for-pdpc>
- <https://www.asiaone.com/singapore/personal-data-protection-commission-denise-wong>
- <https://oecd.ai/en/community/denise-wong>
- <https://assurance.aiverifyfoundation.sg/report/executive-summary/>
- <https://assurance.aiverifyfoundation.sg/report/whats-next/>
- <https://www.imda.gov.sg/resources/press-releases-factsheets-and-speeches/press-releases/2025/sg-unveils-insights-from-worlds-first-technical-testing-of-real-world-applications-of-genai>
- <https://www.imda.gov.sg/-/media/imda/files/about/emerging-tech-and-research/artificial-intelligence/large-language-model-starter-kit.pdf>
- <https://aiverifyfoundation.sg/project-moonshot/>
- <https://github.com/aiverify-foundation/moonshot>
- <https://www.imda.gov.sg/resources/blog/blog-articles/2025/01/red-teaming-challenge>
- <https://www.imda.gov.sg/-/media/imda/files/about/emerging-tech-and-research/artificial-intelligence/singapore-ai-safety-red-teaming-challenge-evaluation-report.pdf>
- <https://www.imda.gov.sg/resources/press-releases-factsheets-and-speeches/press-releases/2025/singapore-ai-safety-initiatives-global-ai-summit-france>
- <https://opengovasia.com/singapore-new-imda-iapp-partnership-advances-ai-governance/>
- <https://www.humane-intelligence.org/red-teaming-events/singapore-imda-red-teaming-on-multilingual-and-multicultural-biases-2024>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
