---
layout: post
title: "人物 · Florian Tramèr"
date: 2026-08-30
description: "他专门证明别人的安全评测在骗自己：攻击算多少钱、防御让模型变多笨，都得写清楚"
categories: card
tags: [llm-security, card, person, academic]
giscus_comments: false
---
<img src="/assets/img/radar/florian-tramèr.png" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**他专门证明别人的安全评测在骗自己：攻击算多少钱、防御让模型变多笨，都得写清楚**

- **身份**：ETH Zürich 计算机系助理教授（tenure track），SPY Lab 负责人
- **主页**：[https://floriantramer.com/](https://floriantramer.com/)
- **从哪读起**：先读 On Adaptive Attacks to Adversarial Example Defenses（arXiv [2002.08347](https://arxiv.org/abs/2002.08347)）第 3 节之后的逐个防御拆解，那里能看清他判断「一个安全声明成不成立」的全部标准；再读 The Jailbreak Tax（arXiv [2504.10694](https://arxiv.org/abs/2504.10694)）看同一把尺子怎么量到 LLM 上。
- **成名作**：2020 年与 Carlini 等人合作的 [On Adaptive Attacks to Adversarial Example Defenses](https://arxiv.org/abs/2002.08347)：逐个针对 ICLR/ICML/NeurIPS 上刚发表的 13 个鲁棒性防御手工设计攻击，几乎全部打回接近零鲁棒精度，从此「防御论文必须自己写针对性攻击」成了投稿默认要求。

| 时期 | |
|---|---|
| 2022-08–今 | ETH Zürich 计算机系 tenure-track 助理教授，领导 SPY Lab（Secure and Private AI） |
| 2021–2022 | Google Brain 研究员（一年） |
| –2021 | Stanford 大学计算机科学博士，导师 Dan Boneh |

## 2020 年拆掉 13 个「已发表的鲁棒防御」，从此论文得自己写攻击

那篇论文没有提出新攻击算法。做法是把 ICLR/ICML/NeurIPS 上刚发表的 13 个防御逐个读懂机制，再为每一个手工凑损失函数和优化过程：防御里塞了个不可导的量化层，就用 BPDA 在反向传播时把它当恒等函数直通过去；防御靠随机化（每次前向随机裁剪/加噪），就用 EOT 对随机性多次采样求梯度期望；防御靠一个额外的检测器判断输入是不是对抗样本，就把检测器的分数一起写进攻击目标里。结果几乎所有防御的鲁棒精度掉到接近零。

真正改掉的是投稿规范。在此之前，一篇防御论文写「我们用标准 PGD / AutoAttack 攻了，鲁棒精度 60%」就算证据；此后审稿人会问：如果攻击者完整知道你这套防御的实现细节，他会怎么改攻击？作者拿不出这个论证，数字就不算数。后面几节都是这条要求换到新场景重来一遍。

## 把「黑盒所以安全」一次次证伪，并且标上价钱

他 2016 年在 USENIX Security 的 Stealing Machine Learning Models via Prediction APIs 就是这条线的起点——只靠预测 API 的返回值就能复制出一个功能等价的模型（这篇 2026 年拿了 USENIX Security Test of Time Award）。十年后同一件事搬到 LLM：Stealing Part of a Production Language Model 只用 API 返回的 logprobs，就把模型最后那层嵌入投影矩阵恢复出来，顺带确定隐层维度——对 OpenAI 的 ada 和 babbage，完整投影矩阵不到 20 美元；对 gpt-3.5-turbo，作者估算完整提取在 2000 美元以内。Scalable Extraction of Training Data from (Production) LMs 则让 ChatGPT 一直重复某个词，直到它从对话分布里掉出来、开始成段吐训练语料原文，吐出真实训练数据的频率比正常提问高约 150 倍。Poisoning Web-Scale Training Datasets is Practical 走另一个方向：LAION 这类数据集存的是 URL 不是图片，标注时那些域名有效，等你下载时其中一部分已经过期可以买回来——买下过期域名换掉图片内容，污染 0.01% 的样本大约 60 美元。

他每篇都把攻击折算成美元和查询次数。这不是修辞，是给「攻击者预算」这个量纲定值：一个防御如果只在攻击者花不起 2000 美元的假设下成立，就得把这句话写在论文里。

## 也拆自己这边：越狱成功率根本不衡量攻击有没有用

The Jailbreak Tax（[2504.10694](https://arxiv.org/abs/2504.10694)）针对的是越狱文献的通用报数惯例：只报 attack success rate——模型没拒绝就记一次成功。他们改成让越狱后的模型去做有标准答案的任务（比如一道数学题、一段可验证的化学流程），发现很多越狱确实绕过了拒绝，但输出的内容能力大幅退化：答案错、步骤对不上。更麻烦的是 ASR 高低和退化程度不相关——一个 ASR 更高的越狱方法，未必给出更可用的输出。所以拿 ASR 排名越狱方法这件事本身是失效的。他同样批评过隐私防御的评测：成员推断攻击的基线设置不当会系统性高估保护效果。他要求的规矩是同一条：任何安全声明必须同时报 utility。

## 从拆到造，代价是承认防御会让 agent 变笨

2024 年他组里的 AgentDojo（一作 Edoardo Debenedetti）把 prompt injection 评测做成可扩展的动态环境而不是固定题库——新攻击可以随时加进去，防御没法靠过拟合一批测试样本刷分。2025 年的 CaMeL（Google DeepMind 主导，他组参与）走架构路线：一个可信 LLM 只看用户原话生成一段程序，决定接下来调哪些工具；邮件、网页这些不可信内容只能作为数据在程序里流动，永远不能改变下一步执行哪个动作。

Design Patterns for Securing LLM Agents against Prompt Injections（[2506.08837](https://arxiv.org/abs/2506.08837)）把这类能给出保证的模式列了六种，同时把代价写明：在开放式真实任务上，越强的动作限制越会大幅牺牲可用性。他也把残余漏洞写出来了——控制流隔离挡住了「换一个动作」，挡不住「改已计划动作的参数」：agent 本来就要发一封邮件，注入内容改不了这个决定，但能把收件人字段改成攻击者地址。据其组 2026 年把 CaMeL 用到 computer-use agent 的近期工作，能操纵感知模块输出的攻击者还可以在合法计划内部把执行推向自己选的分支，而且能同时骗过视觉和 DOM 两路校验——「多个独立验证器投票会更鲁棒」这个假设不成立。

## 他现在的判断：LLM 判 LLM 不算防御

据其组 2026 年的近期工作，方向集中在三处。一是执行动作前做规则化、结构化的确定性检查，比让一个 LLM safety judge 去判断可靠，并且如实报告这套检查多花的 token、延迟和钱。二是用 RL 同时优化「注入成功」和「任务照常完成」——攻击后的 agent 表面上任务完成得甚至比没被攻击时更好，所以「任务正常完成」不能拿来当没被攻击的信号，很多线上监控正是这么判的。三是把 agent 的长期记忆当作投毒载体：写进记忆的一句话在之后任意一次会话里被读回来，用于数据外泄。

**已核实来源**

- <https://floriantramer.com/>
- <https://inf.ethz.ch/news-and-events/spotlights/infk-news-channel/2022/08/welcome-florian-tramer.html>
- <https://ethz.ch/staffnet/en/news-and-events/internal-news/archive/2026/08/florian-tramer-test-of-time-award.html>
- <https://spylab.ai/contact/>
- <https://arxiv.org/abs/2002.08347>
- <https://arxiv.org/abs/2403.06634>
- <https://arxiv.org/abs/2311.17035>
- <https://arxiv.org/abs/2302.10149>
- <https://arxiv.org/abs/2504.10694>
- <https://arxiv.org/abs/2506.08837>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
