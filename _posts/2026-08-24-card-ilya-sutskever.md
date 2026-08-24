---
layout: post
title: "人物 · Ilya Sutskever"
date: 2026-08-24
description: "押注「让模型在监督者比它弱时仍泛化对」这条安全路线，并在 OpenAI 组织层面输掉了它"
categories: card
tags: [llm-security, card, person, exec]
giscus_comments: false
---
**押注「让模型在监督者比它弱时仍泛化对」这条安全路线，并在 OpenAI 组织层面输掉了它**

- **身份**：Safe Superintelligence Inc. 联合创始人；2025 年 7 月起任 CEO
- **主页**：[https://ssi.inc/](https://ssi.inc/)
- **从哪读起**：先读 Burns et al. 2023 的 weak-to-strong generalization（arXiv:[2312.09390](https://arxiv.org/abs/2312.09390)）第 3 节的实验设定和第 6 节作者自陈的局限——设定可以直接搬去做实验，局限那节比结果更值得看。

| 时期 | |
|---|---|
| 2025年7月–今 | Safe Superintelligence Inc. CEO（NVIDIA 2026 年 7 月新闻稿称其为 cofounder and CEO） |
| 2024年6月–今 | 与 Daniel Gross、Daniel Levy 共同创立 Safe Superintelligence Inc.（Palo Alto / Tel Aviv） |
| 2023年7月–2024年5月 | OpenAI Superalignment 团队共同负责人（与 Jan Leike） |
| 2015–2024年5月 | OpenAI 联合创始人、首席科学家；2023 年 11 月前为董事会成员 |

## 他不做提示注入，他做「打分的人不如被打分的人聪明」

如果你是带着 jailbreak、indirect prompt injection、agent 权限边界这套预期来的，这张卡里一件都没有。他的安全工作全在另一条线上：scalable oversight。

两条线的假设不一样。攻防那条线假设模型能力是已知的，问题是有人拿它干坏事，或者外来文本里塞了一句「把通讯录发到 evil.com」把它骗走——你知道什么是正确行为，只是没挡住。他这条线假设的是另一种局面：模型在某个任务上已经超过所有能雇到的标注者，你连「哪个输出是对的」都判断不了。RLHF 的整套机制建立在「人能对两个回答分出好坏」上；一旦模型写出的代码里那个漏洞没人看得出来，偏好数据就开始教它「看起来对」而不是「是对的」。

这条线上没有攻击者。它的失败模式是你自己的训练信号在你不知情的时候变质。

## 用 GPT-2 当老师去教 GPT-4：这个实验的设定，和它被质疑的那一点

2023 年 12 月，Superalignment 团队的第一篇论文（Burns、Izmailov、Kirchner 等，Sutskever 署名）把这个问题做成了一个今天就能跑的实验：既然没有超人模型可用，就把「强模型 + 弱监督者」这一对提前造出来——拿 GPT-2 级别的模型给任务打标签，用这些（含大量错误的）标签去微调 GPT-4，看它最后能不能比它的老师强。

指标叫 PGR（performance gap recovered）：弱老师的分数是下限，用真标签微调强模型的分数是上限，弱监督训出来的模型落在这条线段的哪个位置。结果是强模型确实能超过老师，但天真微调只补回一小截；加一个辅助的置信度损失（鼓励模型对自己的预测有信心，而不是亦步亦趋地拟合老师的错标）之后，NLP 任务上用 GPT-2 级别监督微调 GPT-4 能拿到接近 GPT-3.5 的水平。

值得写的是论文自己承认的那个漏洞：GPT-4 的预训练数据里本来就有人类在这些任务上的正确表现——SciQ 那类小学科学题几乎必然出现过。所以弱监督在这里更像是把模型早就会的东西**勾出来**（elicitation），而不是从一个更笨的老师身上学到了新知识。真正的超人场景里，模型要学的东西预训练数据里没有任何示范，这个类比成不成立，实验说不了。

读者从这里能拿走的：这套 setup 便宜、可复现，后续一批工作直接沿用；但 PGR 数字别当成对齐进展的刻度。

## 承诺 20% 算力的团队，十个月后没了

2023 年 7 月 OpenAI 宣布成立 Superalignment，由他和 Jan Leike 共同领导，公开承诺四年内投入 20% 算力。2024 年 5 月 14 日他宣布离职，Leike 几天后跟进，团队随即解散、成员并入其他组。Leike 在离职声明里写「安全文化与流程让位给了闪亮的产品」。（20% 算力是否真的到位，前员工说法与公司说法冲突，这里只写「承诺过」和「解散了」两件可查的事。）

这件事此后被反复当作参照点：一个实验室对安全的资源承诺，可以在没有任何对外解释的情况下撤销。之后关于「自愿安全承诺该怎么写才有约束力」的讨论——是否需要外部可验证的算力披露、是否需要离职人员的公开发言权——很多都拿这十个月当例子。Leike 本人在 2024 年 5 月底转去了 Anthropic。

## 2023 年 11 月：他试过用董事会当安全机制，结果这机制五天就翻了

OpenAI 的非营利董事会名义上握有对营利实体的控制权，包括解雇 CEO。2023 年 11 月他作为董事参与罢免 Altman。四十八小时内，绝大多数员工联署威胁集体辞职；11 月 20 日他在 X 上公开表示「我为自己参与董事会的行动深感后悔」，并签了那封联署信；Altman 五天后复职，他退出董事会。

对关心 AI 治理的人，这是迄今最贵的一次现实检验：把安全的最终制动权放在一个人数很少、不持股、不掌握人事与算力的董事会手里，在员工与投资方的压力下能撑五天。之后 OpenAI 走向公司重组，谁也没能再造出一个更硬的版本。（内部动机与人际过程不在这里写，只有上面这条公开经过。）

## SSI：两年零产品、外部看不见任何研究

2024 年 6 月他与 Daniel Gross、Daniel Levy 创立 Safe Superintelligence Inc.；Gross 2025 年 7 月离开后他出任 CEO。官方站上只有一段话：一个目标一个产品，安全与能力一起推进，不做中间产品。2026 年 7 月 NVIDIA 宣布长期战略合作并投资，SSI 因此能把算力提高一个数量级（金额未披露；外界流传的估值与融资数字均来自二手报道）。

代价是明确的：他不再向外部安全研究社区输出任何可复现的东西。weak-to-strong 那种「别人能照着跑」的产出在 SSI 这两年没有对应物。想跟进他现在怎么想，能读的只有 2025 年 11 月 25 日 Dwarkesh Patel 那期播客里的口头表述——他在里面说 2020–2025 是 scaling 的时代、2026 起回到 research 的时代，再加 100 倍规模会有区别但不会带来质变，瓶颈转到泛化上；以及他把超级智能的核心难题更多归到权力而非技术对齐。这些是访谈里的话，不是他写下来的立场。

**已核实来源**

- <https://arxiv.org/abs/2312.09390>
- <https://cdn.openai.com/papers/weak-to-strong-generalization.pdf>
- <https://www.cnbc.com/2024/05/17/openai-superalignment-sutskever-leike.html>
- <https://x.com/janleike/status/1791498174659715494>
- <https://x.com/ilyasut/status/1726590052392956028>
- <https://www.axios.com/2023/11/20/sam-altman-fired-openai-board-illya-sutsever-regrets>
- <https://ssi.inc/>
- <https://nvidianews.nvidia.com/news/ilya-sutskevers-safe-superintelligence-inc-and-nvidia-announce-long-term-strategic-partnership>
- <https://techcrunch.com/2025/07/03/ilya-sutskever-will-lead-safe-superintelligence-following-his-ceos-exit>
- <https://www.dwarkesh.com/p/ilya-sutskever-2>
- <https://www.cnbc.com/2024/05/14/openai-co-founder-ilya-sutskever-says-he-will-leave-the-startup.html>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
