---
layout: post
title: "人物 · David Wagner"
date: 2026-09-06
description: "两次给新出现的 AI 安全问题定下「什么才算证明有效」，并做出了 Meta 采用的开源抗注入模型"
categories: card
tags: [llm-security, card, person, academic]
giscus_comments: false
---
<img src="/assets/img/radar/david-wagner.gif" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**两次给新出现的 AI 安全问题定下「什么才算证明有效」，并做出了 Meta 采用的开源抗注入模型**

- **身份**：UC Berkeley EECS 计算机科学教授
- **主页**：[https://citris-uc.org/people/person/professor-david-wagner/](https://citris-uc.org/people/person/professor-david-wagner/)
- **从哪读起**：先读 [SecAlign](https://arxiv.org/abs/2410.05451)（CCS 2025），再读 [Meta SecAlign](https://arxiv.org/abs/2507.02735)：前者是训练层面的抗注入做法，后者是它被做成开源权重后在没训练过的 agent 任务上还剩多少鲁棒性。
- **成名作**：与 Nicholas Carlini 合作的 [Towards Evaluating the Robustness of Neural Networks](https://arxiv.org/abs/1608.04644)（C&W 攻击）——它打掉了当时被广泛引用的 defensive distillation，也把「防御必须扛得住针对它本身设计的攻击」变成了这个领域的评测底线。

| 时期 | |
|---|---|
| 2020–2022 | UC Berkeley 计算机科学系主任（据维基百科） |
| 2010–今 | UC Berkeley 正教授 |
| 2000–今 | UC Berkeley EECS 任教 |
| 1995–2000 | UC Berkeley 计算机科学硕士、博士（导师 Eric Brewer）；此前 Princeton 数学本科 |

## 他先证明了「防御报的数字大多是假的」

2016 年 defensive distillation 号称把对抗样本的攻击成功率从 95% 压到 0.5%。他和 Carlini 写了一组新攻击，在同一批蒸馏网络上成功率 100%——原来那 0.5% 是蒸馏时的温度常数让梯度趋近于零、攻击者算不出方向造成的假象，模型本身并没有变结实。次年他们又用一篇论文一次性打掉十个「检测对抗样本」的方法：这些检测器都只在论文自己选的固定攻击集上测过，攻击者只要知道检测器存在并把它的判据一起写进优化目标，检测率就归零。2018 年 Obfuscated Gradients（获 ICML 最佳论文）把这件事收成一条规矩：**报 ASR 时必须报自适应攻击下的 ASR**，也就是让攻击者知道你的防御细节、可以拿到梯度、可以为你这个防御专门重写攻击。

这条规矩今天还在管着 prompt injection。他组 2025 年的 DefensiveToken 论文自己就把最坏数字写进摘要：面对基于优化搜索的注入，ASR 从 95.2% 只降到 48.8%——同一篇里既报最好数字也报最坏数字。读者该带走的是这个口径，不是任何一个攻击算法。

## StruQ 和 SecAlign：不指望模型「看懂」边界，而是训练它服从边界

StruQ（USENIX Security 2025）做两件事：前端把系统指令和外部数据格式化成两个通道，中间用特殊分隔符隔开，并把用户输入里所有伪造的分隔符过滤掉（不然攻击者只要在邮件正文里打一个假的 `[MARK] instruction:` 就能自己开一个指令通道）；然后拿普通 instruction-tuning 数据集，在 data 段里塞进注入指令、把「正确答案」标成执行原指令，微调出一个只听 prompt 通道的模型。

SecAlign（CCS 2025）把这一步换成偏好优化：同一段被注入的数据，构造一对回答——一个执行原指令（secure），一个执行注入指令（insecure），做 DPO。论文报告注入成功率降到 10% 以下，且对比训练时没见过的、更复杂的攻击也成立。终点是与 Meta FAIR 合作的 Meta-SecAlign 8B/70B：只用通用数据训练，却在没训过的 tool-calling 和网页导航任务上保住了鲁棒性，70B 在多项安全 benchmark 上强过若干带注入防御的闭源模型，权重公开。这一点重要在于——此前效果最好的模型级防御全在闭源模型里，外面的人只能攻不能改。

## 他同时做系统级隔离，也同时写下它治不了什么

Type-Directed Privilege Separation（[2509.25926](https://arxiv.org/abs/2509.25926)）不把不可信输入当裸字符串传，而是强制转成受限类型；Web Agents Should Adopt the Plan-Then-Execute Paradigm（[2605.14290](https://arxiv.org/abs/2605.14290)）指出 ReAct 把「读到什么」和「下一步调哪个工具」压进同一次推理，网页里的一句话可以直接改变工具选择，先把计划定死再执行就切断这条路。

但他组自己反复记录了两条限制，这比结论本身有用：严格的控制流/数据流分离挡不住**不改变控制流的注入**——攻击者不让 agent 多调一个工具，只是把它写出来的摘要内容篡改掉，隔离机制全程看不出异常；而在数据本就该影响决策的任务上（比如「读完这封邮件再决定要不要回复」），这种分离会大幅掉效用。DataFilter（[2510.19207](https://arxiv.org/abs/2510.19207)）那种在输入前面挂一个过滤器的做法，是留给拿不到权重、改不了训练的人的折中。

## 近两年的火力转向持久状态和微调通道

Trojan Hippo（[2605.01970](https://arxiv.org/abs/2605.01970)，2026 年预印本）把 agent memory 当攻击面：这次会话里被写进长期记忆的一段内容，会在几天后一次完全无关的会话里被重新读出来执行，用来做数据外泄。GradShield（[2605.14194](https://arxiv.org/abs/2605.14194)，预印本）针对另一条路径——在**完全良性、一条有害样本都没有**的普通数据上微调，安全对齐就会被侵蚀、有害顺从率回升；而且自动内容审核和 LLM 审计器只抓得住那些内容明显有害或明显做作的数据，抓不住这种看起来正常的投毒。

## 一把量别人论文的尺子

他组多篇论文（Twin Agent、Trojan Hippo、DataFilter）独立得到同一条：降 ASR 一定伴随良性任务效用或性能的对应损失，不存在免费的安全改进。另一条经验规律来自 Twin Agent：目标模型越强越稳，成功的注入需要越长的恶意文本。所以看到一篇防御论文只报 ASR 不报 utility，默认它把账藏起来了。

**已核实来源**

- <https://arxiv.org/abs/1608.04644>
- <https://arxiv.org/abs/1705.07263>
- <https://arxiv.org/abs/2402.06363>
- <https://arxiv.org/abs/2410.05451>
- <https://arxiv.org/abs/2507.02735>
- <https://arxiv.org/abs/2507.07974>
- <https://arxiv.org/abs/2510.19207>
- <https://arxiv.org/abs/2509.25926>
- <https://github.com/facebookresearch/Meta_SecAlign>
- <https://citris-uc.org/people/person/professor-david-wagner/>
- <https://en.wikipedia.org/wiki/David_A._Wagner>
- <https://dl.acm.org/doi/10.1145/3719027.3744836>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
