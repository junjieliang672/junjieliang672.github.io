---
layout: post
title: "人物 · Risto Uuk"
date: 2026-09-07
description: "研究欧盟 AI 法案里「系统性风险」到底指哪些风险，并把结论同时写进论文和立法意见"
categories: card
tags: [llm-security, card, person, policy]
giscus_comments: false
---
<img src="/assets/img/radar/risto-uuk.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**研究欧盟 AI 法案里「系统性风险」到底指哪些风险，并把结论同时写进论文和立法意见**

- **身份**：Future of Life Institute，欧洲政策与研究负责人
- **主页**：[https://ristouuk.com/](https://ristouuk.com/)
- **从哪读起**：先读 arXiv:[2412.02145](https://arxiv.org/abs/2412.02145) 的结果表（27 项缓解措施 × 76 位专家打分），它直接告诉你哪三项措施最可能变成前沿模型开发者的硬性义务；再回头看 arXiv:[2412.07780](https://arxiv.org/abs/2412.07780) 的 13 类风险，看你的评测覆盖了几格。
- **成名作**：把欧盟 AI 法案里那个只有循环定义的法律词「系统性风险」做成了一份可枚举的清单——[A Taxonomy of Systemic Risks from General-Purpose AI](https://arxiv.org/abs/2412.07780)（2024），从 1781 篇文献筛到 86 篇，归出 13 类风险、50 个风险来源，成为讨论第 51/55 条义务时最常被引的那份口径。

| 时期 | |
|---|---|
| 至今 | Future of Life Institute，Head of European Policy and Research；主编 EU AI Act Newsletter，官方口径 5 万+ 订阅 |
| 至今 | 爱沙尼亚政府 AI 顾问委员会成员（据 FLI 官方人员页） |
| — | KU Leuven 博士（据 FLI 官方人员页） |
| 更早 | 曾任职 World Economic Forum 与 European Commission；LSE 哲学与公共政策硕士，塔林大学本科 |

## 法条写了「系统性风险」，但没说是哪些风险

AI 法案第 51/55 条给「具系统性风险的通用 AI 模型」加了一堆义务：做风险评估、做对抗性测试、报告严重事件。但第 3(65) 条对「系统性风险」的定义几乎是循环的——大意是「因能力规模大而可能对整个联盟市场或社会产生重大负面影响的风险」。你拿着这句话去写一份合规报告，不知道该测什么。

[A Taxonomy of Systemic Risks from General-Purpose AI](https://arxiv.org/abs/2412.07780)（Uuk、Gutierrez、Guppy、Lauwaert、Kasirzadeh、Velasco、Slattery、Prunkl，2024）做的就是把这句话展开。方法是系统性文献综述：1781 篇初筛，86 篇入选，归纳出 13 类系统性风险和 50 个风险来源，并把根源归到三处——知识缺口、危害难以被识别、发展轨迹不可预测。

对做评测的人，这份清单里最该注意的是它有多宽。环境影响、结构性歧视、劳动力市场冲击、治理失灵，跟失控、CBRN 是并列条目。换句话说，一份只报告了 CBRN uplift 分数和越狱成功率的模型卡——比如「我们跑了 500 条有害请求，拒答率 97%，另请生物学专家做了一轮 uplift 评估」——在这套口径下不是「安全」，是覆盖了 13 格里的两格。合著者里有 MIT AI Risk Repository 的 Peter Slattery，这套分类跟那个数据库是同一条路数：先把所有人说过的风险摊平，再看法律的口子对上哪几条。

## 76 位专家给 27 项缓解措施排了名

[Effective Mitigations for Systemic Risks from General-Purpose AI](https://arxiv.org/abs/2412.02145)（Uuk、Brouwer、Schreier、Dreksler、Pulignano、Bommasani，2024）是配套的另一半。先从文献里抽出 27 项候选缓解措施，然后请 76 位专家打分，专家跨五个领域：AI safety、关键基础设施、民主进程、CBRN、歧视与偏见——也就是说，评「模型审计有没有用」这件事，问的不只是 AI 圈的人。打分维度是有效性和技术可行性，另外还让专家自己组合出一套他认为该采用的措施包。

结果：安全事件报告与信息共享、独立的部署前模型审计、部署前风险评估这三项，在全部四个风险领域都拿到 >60% 的专家认同，且在专家自选的组合里出现率 >40%。作者从中提炼出三条原则：外部审查、事前评估、透明度。

这三条的共同点值得技术读者留意：它们都要求「不是你自己说了算」。一家实验室发布前自己组织内部红队、自己写报告、自己判定通过，这套流程在上面三项里一项都不占。真按这个方向立规，独立评测机构就有了付费需求方——谁能做第三方部署前审计，谁就在这条线上。

## 写论文的人和写立法意见的人是同一个

Uuk 是 Future of Life Institute 的 Head of European Policy and Research（FLI 官方人员页写明），同时主编 EU AI Act Newsletter（双周刊，FLI 自述 5 万+ 订阅），配套站点是 artificialintelligenceact.eu——很多人查法条原文和实施时间线用的就是这个站。

这件事对读者的意义是双向的。好处：同一套分类既发成论文也进立法评议，所以你在 arXiv 上读到的 13 类风险，和欧盟层面讨论 GPAI 义务时被端上桌的清单，是同一份东西——想知道你的技术结论会被塞进哪个格子，读他比读法条快。需要留意的是：FLI 是倡导机构，它的官方页面自述参与塑造了法案的通用 AI 与系统性风险条款，这是机构自我描述，不是第三方认定。分类学和专家调查的方法是公开可查的，但「哪些风险应该被管」这件事本身带立场，看的时候要分开：文献计数和专家打分是证据，「因此应该立法要求第三方审计」是主张。

## 「通用 AI」当年得先被定义出来才能被监管

更早的一篇是 [A Proposal for a Definition of General Purpose AI Systems](https://link.springer.com/article/10.1007/s44206-023-00068-w)（Gutierrez、Aguirre、Uuk、Boine、Franklin，2023），在 GPAI 条款成形前后梳理「通用系统」和「通用模型」在文献里被用得有多乱，并给了一个可操作的定义。

看这篇是为了理解法案最后为什么把义务挂在「模型」而不是「系统」上，以及为什么会冒出训练算力这种代理指标——一个模型要不要被划进系统性风险类，看的是训练算力估计值，不是它实测出来的危险能力。这意味着两件事同时成立：一个 10^25 FLOP 以上、实测什么危险能力都没有的模型照样背上全套义务；而一个算力不到线、但被微调出很强攻击辅助能力的模型不自动进这一类。（Uuk 本人对具体门槛数值持什么主张，我没查到直接来源。）

## 在他的框架里，prompt injection 只占一格

他不做攻击复现，不做模型级评测方法学。间接注入、工具调用滥用、模型权重窃取这些具体形态，在 13 类分类里被压成少数几个条目——你花半年做出来的一整条攻击链，在那份表里可能就是一行。所以把他当技术来源会失望，当「技术结论如何被转译成法律义务」的通道才对。

实用性上，2026 年的数字综合法案（Digital Omnibus）推迟了部分高风险义务的适用时间，GPAI 这一侧的时间线到底怎么走，他的 newsletter 是逐期跟的。

**已核实来源**

- <https://futureoflife.org/person/risto-uuk/>
- <https://arxiv.org/abs/2412.07780>
- <https://arxiv.org/abs/2412.02145>
- <https://link.springer.com/article/10.1007/s44206-023-00068-w>
- <https://ristouuk.com/>
- <https://artificialintelligenceact.substack.com/about>
- <https://futureoflife.org/about-us/our-people/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
