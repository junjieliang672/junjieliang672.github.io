---
layout: post
title: "人物 · Dawn Song"
date: 2026-07-24
description: "AI 安全领域先把「怎么测」定下来的人：从贴纸骗过路牌识别，到今天给 AI 智能体做红队评测"
categories: card
tags: [llm-security, card, person, industry]
giscus_comments: false
---
<img src="/assets/img/radar/dawn-song.png" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**AI 安全领域先把「怎么测」定下来的人：从贴纸骗过路牌识别，到今天给 AI 智能体做红队评测**

- **身份**：Meta Superintelligence Labs AI Research 副总裁（2026 年 6 月宣布）；UC Berkeley 计算机系教授
- **主页**：[https://dawnsong.io](https://dawnsong.io)
- **从哪读起**：先看 2018 年那篇 Robust Physical-World Attacks（openaccess.thecvf.com 上有全文，看图就够），它用一张贴了几片黑白贴纸的停车牌，一句话说清了她做事的方式：不争论风险存不存在，直接做一个能拍照的证据出来。
- **成名作**：[Robust Physical-World Attacks on Deep Learning Visual Classification](https://arxiv.org/abs/1707.08945)：在停车标志上贴几块黑白胶带，就让识别模型稳定读成「限速 45」，第一次证明对抗样本能在真实光照和视角下生效，是物理世界攻击的奠基工作。

| 时期 | |
|---|---|
| 2026.06–今 | 宣布加入 Meta Superintelligence Labs，任 AI Research 副总裁，负责 AI 安全与安全性方向 |
| 2025 | 加州州长办公室《California Report on Frontier AI Policy》高级顾问（2025 年 6 月 17 日定稿） |
| 2024–今 | AI 安全公司 Virtue AI 联合创始人 |
| 至今 | UC Berkeley 计算机系教授；据其个人主页，兼任 Berkeley RDI 中心联合主任、世界经济论坛 AGI Global Council 成员（2025– ） |

## 先造尺子，再谈防守：她每次进新领域的第一动作

2017 年之前，「机器学习模型能被骗」这件事主要活在数学里：给图片加一层人眼看不见的噪声，分类器就认错。很多人的反应是「实验室玩具，现实里不会发生」。

2018 年那篇 Robust Physical-World Attacks（Eykholt 等，Dawn Song 是共同作者）把这个争论终结了：他们在一块真实的停车牌上贴了几片黑白贴纸，看起来就像被人涂鸦过；然后开着车拍视频，交通标志识别模型在 84.8% 的视频帧里把它读成了「限速 45」。实验室静态照片下是 100%。争论从此不再是「这算不算真问题」，而是「怎么防」。

2023 年她换到大语言模型上做了同一件事。DecodingTrust（第一作者 Boxin Wang，Bo Li 是主导者之一，Song 是共同作者之一）不去争「GPT-4 到底可不可信」，而是把这个含糊的问题拆成八个能分别打分的维度：会不会说脏话和仇恨言论、有没有刻板印象、被改写句子后会不会答错、遇到没见过的题型会不会崩、会不会把训练数据里的隐私吐出来、道德判断是否稳定，等等。这篇拿了 NeurIPS 2023 数据集与评测赛道的杰出论文奖。它真正有用的地方不是分数，是那套拆分——后来很多公司做模型安全评估，报告结构大体沿用了这几类。

读她的工作，主要是拿走「该测什么」，而不是拿走某个具体防御方案。

## 2017–2019 那两篇攻击，今天讲 LLM 安全还得从它们讲起

第一件事叫**数据投毒后门**（backdoor attack）：往训练集里塞几十张照片，照片里的人都戴着同一副特定花纹的眼镜，并且都被标成「这是张三」。训完之后模型在正常情况下一切如常，但只要有人戴上那副眼镜，模型就认成张三。攻击者不需要碰模型权重，只需要污染几十条训练数据。（Chen、Liu、Li、Lu、Song，2017）今天大家担心的「从网上下载的微调数据集不干净」「开源模型可能被人动过手脚」，是同一个问题换了载体。

第二件事叫**训练数据记忆**：The Secret Sharer（Carlini、Song 等，USENIX Security 2019）往训练语料里插进一串编造的信用卡号，训练完成后，靠反复向模型试探哪些数字串它「觉得更顺口」，能把这串号码一字不差地还原出来。这就是今天所有「模型会不会背下我的邮件/病历/源码」争议的技术原型。要点是：模型不需要「打算」泄密，它只是把罕见的字符串记牢了。

## 2026 年她在拆自己搭的台子：攻击成功率这个指标坏在哪

最值得注意的是，她现在正在推翻自己参与建立的那套评测口径。她的组 2026 年的一批工作集中攻击一个业内默认的做法——用「攻击成功率」（ASR：一百次攻击里模型上钩几次）当作安全水平的度量。三条具体的问题：

**一、按单轮对话、单个步骤、每次重置的会话来评估，会系统性低估风险。**一个恶意目标可以被拆成十步：先让智能体列出公司通讯录，再让它整理成表格，再让它「把这份文件发给协作方」——每一步单看都完全正常，只有把整段有状态的、跨多轮的交互合起来看，才看得出这是一次数据外泄。评测框架如果每道题做完就清空上下文，这类攻击永远测不出来。

**二、防御是有代价的，只报攻击成功率等于没报。**他们的测量显示，让攻击成功率下降的防护措施，同时会让模型拒绝本该完成的正常请求、或者把任务做砸；防护越强，这个代价越大。所以「我们把 ASR 从 40% 降到 5%」这句话，不配上正常任务的完成率就没有意义——把所有请求都拒绝掉，ASR 是 0%。

**三、别拿模型自己的话当证据。**很多评测判断攻击是否得手，看的是模型有没有说出「好的，我这就把文件发出去」。他们发现这样判定会明显高估危害——模型经常嘴上答应、实际什么也没做，或者做错了。他们的 DTap 红队平台因此改成核查真实后果：文件到底有没有被发出去。

## 「加个防护栏」为什么不够：沙箱 vs. 逐个操作的授权

她的组 2026 年另一条主张，是用传统系统安全的眼光看智能体。

目前常见做法是给 AI 智能体一个沙箱容器，限制它能碰哪些目录。问题是：一个被注入了恶意指令的智能体，手里握的是**合法凭证**——它本来就有权读那个文件、有权调那个发送接口，沙箱看不出它这次是替谁办事。粗粒度的隔离拦不住它。

另一条常见做法是内容检测：用分类器或规则过滤，扫一遍网页和邮件里有没有可疑指令。他们的结论是这类防御有结构性上限——攻击者知道你部署了哪个检测器之后就照着改写，或者换一条你没设防的通道进来（比如图片里的文字、工具返回值、文档元数据）。

还有一条反直觉的：**只检查「这个动作符不符合用户的目标」，抓不到注入攻击。**用户说「帮我整理这个季度的报销单」，注入的指令让智能体把报销单整理好并抄送一份给外部地址——这个动作和用户目标完全一致，目标一致性检查一路放行。缺的是另外两样东西：这条指令是用户说的还是从网页里读来的（来源追踪），以及每一个具体操作有没有被单独授权。

## 读她的东西时要带的背景

她同时挂着几个身份：Berkeley 计算机系教授、据其个人主页兼任 RDI 中心联合主任；2024 年联合创办 AI 安全公司 Virtue AI；2025 年 6 月定稿的加州州长办公室《California Report on Frontier AI Policy》高级顾问；2026 年 6 月 25 日她本人宣布加入 Meta Superintelligence Labs 任 AI Research 副总裁，Virtue AI 多名成员同行（她加入后与 Berkeley 教职的关系，公开来源没有明确说法）。这意味着同一套评测口径会同时出现在论文、创业公司产品和州级政策文件里。

还有一条实用的读法：她的论文常有十几到二十个作者。署名更多表示「这条线在她的组里跑」，不等于每一节都出自她手——DecodingTrust 就有近二十位作者，第一作者另有其人。引用结论时别都算到她个人头上。

**已核实来源**

- <https://dawnsong.io>
- <https://dawnsong.io/dawn-berkeley.png>
- <https://x.com/dawnsongtweets/status/2070191051873345910>
- <https://www.techradar.com/pro/the-goal-is-not-to-replace-humans-new-meta-ai-research-chief-dawn-song-says-the-next-frontier-is-ai-agents-that-are-economically-valuable>
- <https://www.gov.ca.gov/wp-content/uploads/2025/06/June-17-2025-–-The-California-Report-on-Frontier-AI-Policy.pdf>
- <https://arxiv.org/html/2506.17303v1>
- <https://openaccess.thecvf.com/content_cvpr_2018/html/Eykholt_Robust_Physical-World_Attacks_CVPR_2018_paper.html>
- <https://arxiv.org/abs/1707.08945>
- <https://blog.neurips.cc/2023/12/11/announcing-the-neurips-2023-paper-awards/>
- <https://proceedings.neurips.cc//paper_files/paper/2023/hash/63cb9921eecf51bfad27a99b2c53dd6d-Abstract-Datasets_and_Benchmarks.html>
- <https://decodingtrust.github.io/>
- <https://cryptobriefing.com/dawn-song-meta-superintelligence-labs-vp/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
