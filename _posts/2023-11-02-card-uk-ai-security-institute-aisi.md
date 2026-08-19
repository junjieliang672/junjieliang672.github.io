---
layout: post
title: "机构 · UK AI Security Institute (AISI)"
date: 2023-11-02
description: "英国政府出钱养的模型测试队：能在模型发布前上手测，但拦不住任何一次发布"
categories: card
tags: [llm-security, card, org]
giscus_comments: false
---
**英国政府出钱养的模型测试队：能在模型发布前上手测，但拦不住任何一次发布**

- **身份**：英国科学、创新与技术部（DSIT）下属研究机构
- **主页**：[https://www.aisi.gov.uk/](https://www.aisi.gov.uk/)
- **从哪读起**：先读官网的 Frontier AI Trends Report（aisi.gov.uk/frontier-ai-trends-report），它把两年来的测试结果汇总成了几条能直接引用的结论，比单篇博客更能看出这家机构的判断口径。

| 时期 | |
|---|---|
| 2025年2月14日 | 更名为 AI Security Institute（原 AI Safety Institute），公开说明不再把偏见、言论自由类问题作为工作重点 |
| 2023年11月至今 | 设于英国 DSIT 内部的研究机构，官网自述每财年 £66m 经费、100+ 技术人员 |

## 它能做什么，不能做什么：一个没有否决权的检测方

最容易误解的一点：AISI 不是监管机构。它是英国科学、创新与技术部（DSIT）里的一个研究机构，官网自己的定义就是「a research organisation within the UK government's Department for Science, Innovation and Technology」。它能在 OpenAI、Anthropic、Google DeepMind 的新模型对外发布之前拿到手测试，靠的是和这些公司签的自愿协议——不是法律义务。测出问题之后，它能做的是写报告、告诉政府、告诉公司，然后眼看着模型照常上线。没有哪条英国法律给了它「你这个模型不许发」的权力。

资源上它并不寒酸。据官网 About 页：每财年 £66m 经费、100+ 技术人员、优先使用英国国家算力资源（自述价值超过 £15 亿）。另有一个对外资助计划 The Alignment Project，官网称首轮投出超过 £27m，单个项目最高 £1m，出资方包括加拿大 AI 安全研究所、OpenAI、Anthropic、微软、AWS 等。

所以它的全部影响力只有一个来源：把测试做得足够扎实、工具做得足够好用，让别人不得不引用它。下面几节都是这句话的具体后果。

## 它管的是「模型能不能帮人造出真东西」，不是「模型会不会说脏话」

研究议程只覆盖六个方向：网络攻击滥用、两用科学（比如生物化学知识被用来做危险的事）、犯罪滥用、自主系统（模型在没人盯着的情况下自己行动）、社会韧性、人类影响（说服和操纵）。

2025 年 2 月 14 日，它从 AI Safety Institute 改名为 AI Security Institute。官方公告里写得很直白：新的优先级意味着它「will not focus on bias or freedom of speech」。也就是说，如果你关心的是聊天机器人回答里有没有性别刻板印象、生成图片有没有版权问题、平台该怎么做内容审核——这家机构不管这些，你在它的产出里找不到答案。它盯的是「一个人拿着这个模型，能不能真的做出一批危险的东西」「模型会不会在没人监督时干出格的事」。

## 它真正的影响力可能是一个 Python 库

2024 年 5 月 10 日，AISI 开源了评测框架 Inspect。它做的事情说白了很简单：把「怎么调模型」和「你想测什么」拆开。你写一份测试题目和打分逻辑，换个配置就能在 OpenAI、Anthropic、Google 的模型上跑同一套题，也能跑本地部署的模型（官网现在称支持 20+ 家模型提供方，以及 HuggingFace、vLLM、SGLang 本地推理）。框架里自带工具调用（让模型能敲命令行、跑 Python、搜网页）、多轮对话、沙箱（Docker/Kubernetes 等，把模型的操作关在隔离环境里），以及用另一个模型来给回答打分的功能。

为什么这比报告重要：一个没有执法权的机构，可以通过让所有人用同一把尺子量，来定义什么叫「测过了」。这是在定标准，不是在发论文。（外界常见的「某某公司也在用 Inspect」的名单多来自二手报道，这里不复述。）

## 红队结论：他们测过的每一个系统都被攻破了

AISI 的 Frontier AI Trends Report 里有一句原话：「we've found universal jailbreaks for every system we've tested」。universal jailbreak 指的是一条通用的越狱提示词——不是针对某个问题临时凑出来的话术，而是套在很多种请求前面都能让模型交出本该拒绝的答案。

有两个具体数字：一次测试里，专家红队用 10 分钟、套用一个已经公开的手法，就拿到了生物滥用相关的回答；另一次，同类目标花了 7 小时以上，而且需要现研究出一个新的通用越狱提示。两个模型相隔约半年，也就是难度大概涨了 40 倍——防护确实在变好，但没有一个撑住过。报告还给了一个反直觉的观察：模型能力更强，防护不一定更好，两者相关性极弱（报告给出 R² = 0.097），防护质量主要取决于公司投了多少资源。

这类结论的用处和边界都要说清楚。它能证明「不要把防护当成安全保证」，但它不能告诉你「有多不安全」——花 7 小时的红队专家有测试协议给的特权访问，普通攻击者未必这么快；反过来，攻击手法一旦公开，成本又会塌到几分钟。而且不管测出什么，模型照发。

## 产出节奏：报告先于论文

它的主要出货形式是面向政府的汇总报告（如 Frontier AI Trends Report）和按团队分栏的研究博客，而不是会议论文。偶尔也发顶刊：2025 年 12 月 4 日 Science 上那篇关于对话式 AI 政治说服力的研究，AISI 与牛津互联网研究所、LSE、斯坦福、MIT 合作，近 7.7 万名英国参与者、约 9.1 万次对话、19 个模型、707 个政治议题。最反直觉的发现是：让 AI 更能说服人的不是「针对你个人量身定制话术」，也不是把模型做得更大，而是回答里塞进多少具体事实。

据官网 About 页，领导层包括 Interim Director Adam Beaumont（前 GCHQ 首席 AI 官）、CTO Jade Leung、Chief Scientist Geoffrey Irving、Research Director Chris Summerfield、Chair Ian Hogarth——多数来自 OpenAI、DeepMind、GCHQ 和牛津。这批人的来路解释了它的偏好：做能跑出数字的实证测试，不做理论安全论证。

**已核实来源**

- <https://www.aisi.gov.uk/about>
- <https://www.aisi.gov.uk/research-agenda>
- <https://www.aisi.gov.uk/frontier-ai-trends-report>
- <https://www.gov.uk/government/news/tackling-ai-security-risks-to-unleash-growth-and-deliver-plan-for-change>
- <https://www.gov.uk/government/news/ai-safety-institute-releases-new-ai-safety-evaluations-platform>
- <https://inspect.aisi.org.uk/>
- <https://github.com/UKGovernmentBEIS/inspect_ai>
- <https://alignmentproject.aisi.gov.uk/>
- <https://www.science.org/doi/10.1126/science.aea3884>
- <https://www.oii.ox.ac.uk/news-events/oxford-researchers-reveal-how-conversational-ai-can-change-political-opinions/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
