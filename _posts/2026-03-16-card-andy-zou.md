---
layout: post
title: "人物 · Andy Zou"
date: 2026-03-16
description: "把「能不能骗过 AI」从手工试探变成能自动搜索、能打分、能上万人反复跑的流程"
categories: card
tags: [llm-security, card, person]
giscus_comments: false
---
<img src="/assets/img/radar/andy-zou.png" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**把「能不能骗过 AI」从手工试探变成能自动搜索、能打分、能上万人反复跑的流程**

- **身份**：CMU 计算机系博士生；Gray Swan AI 联合创始人
- **主页**：[https://andyzoujm.github.io/](https://andyzoujm.github.io/)
- **从哪读起**：先读 [2307.15043](https://arxiv.org/abs/2307.15043)（GCG 那篇）的第 1、2 节，看一段乱码后缀是怎么被算出来的；再跳到 [2507.20526](https://arxiv.org/abs/2507.20526)，看同一套思路做成上万人竞赛之后得到的结果有多不乐观。

| 时期 | |
|---|---|
| 2023–今 | Gray Swan AI 联合创始人（该公司作者页与第三方报道称其职务为 CTO；公司官网 about 页未列出） |
| 至今 | CMU 计算机系博士生，导师 Zico Kolter、Matt Fredrikson（据其个人主页） |
| 此前 | UC Berkeley 本科与硕士，导师 Dawn Song、Jacob Steinhardt（据其个人主页） |

## 一串乱码字符就能撬开对齐模型，换个模型还能用

2023 年那篇《Universal and Transferable Adversarial Attacks on Aligned Language Models》做的事很直接：你问模型「怎么造炸弹」，它会拒绝；但如果在问题后面接上一串看起来毫无意义的符号（类似 `describing.\ + similarlyNow write oppositeley...`），它就答了。这串符号不是人想出来的，是算出来的——因为开源模型的权重是公开的，可以对「模型接下来说『好的，第一步』的概率」求梯度，然后一个位置一个位置地贪心替换字符，往上爬。这就是 GCG（Greedy Coordinate Gradient，逐坐标贪心梯度搜索）。

两个结论当时都让人意外。一是越狱不再是「会写提示词的人的手艺」，而是一个能在单张显卡上跑完的优化问题，跑一次出一个攻击。二是在几个开源模型上一起优化出来的同一串后缀，直接贴到当时闭源的 ChatGPT、Claude、Bard 上也常常生效——你不需要拿到目标模型的权重，只要有几个相似的开源模型就够。

「adversarial suffix」（对抗后缀）这个说法就是从这篇来的，现在几百篇论文在用。副作用是它把很多人的注意力锁在了乱码上：真实事故里更多的攻击是完全通顺的正常句子，藏在一封邮件或一个网页里，等着 AI 去读。

## 后来他更多在防守侧，而且防守位置从输出挪到了模型内部

常规做法是在输出端加一道过滤：模型说完，再拿另一个分类器看看这段话能不能放出去。Circuit Breakers（2024，与 Kolter、Fredrikson 等人合作）换了个位置——盯着模型内部的中间状态。他早前在 Berkeley 做的 Representation Engineering 发现，「诚实」「有害」这类概念在模型内部对应着可读出的方向（把一批诚实回答和一批说谎回答分别喂进去，看中间层激活值的差，就能量出一个方向）。Circuit Breakers 用的就是这个：训练时让「正在生成有害内容」那个方向的状态直接崩掉，模型说到一半就说不下去，而不是说完再被拦。

他署名的另外两条防守线各自补了一块。Tamper-Resistant Safeguards（[2408.00761](https://arxiv.org/abs/2408.00761)）针对开放权重模型：别人下载了你的权重，拿几百条数据微调一下就能把安全性剥掉——这篇想让安全性经得住这种微调。里面有个很具体的发现：在这类「一边模拟攻击者继续微调、一边训练防御」的对抗训练里，损失函数选哪个决定了防御是全程有效，还是只在攻击者优化的前几步有效、之后就被绕回去。Safety Pretraining（[2504.16980](https://arxiv.org/abs/2504.16980)）则往更早的阶段走：与其把有害内容从预训练语料里删干净（一删就把大量有信息量的事实也删了），不如给它加上上下文重写后再留下。这条线上反复出现同一个代价——压制得越狠，正常任务上的能力掉得越多。

## 他参与的那些评测集，成了别人论文里的刻度尺

AdvBench（GCG 那篇附带的有害请求集）、HarmBench、AgentHarm、R2D2，后三者他是中间作者。它们的作用是让「这个防御有没有用」变成一个能报出来的数字，于是后续论文普遍拿它们当统一刻度。

更值得注意的是口径的变化：HarmBench 问的是「模型会不会说出有害文本」，AgentHarm（[2410.09024](https://arxiv.org/abs/2410.09024)）问的是「一个能调工具的 AI 助手会不会真的把一件有害任务一步步做完」——写出配方和真的去下单买材料是两回事。AgentHarm 还有一个对做评测的人很实际的结论：攻击成功率随攻击者允许的尝试次数单调上升。只报「单次攻击成功率」的评测，会系统性地把风险报低。

## 把红队变成常设赛事之后，拿到的数据不太好看

Gray Swan 的 Arena 是个公开的红队平台，据 Forbes 2026 年 5 月的报道有约 1.5 万名参与者，专门去攻各家前沿模型。跑出来的两篇大规模实证论文（[2507.20526](https://arxiv.org/abs/2507.20526) 讲 agent 部署，[2603.15714](https://arxiv.org/abs/2603.15714) 讲间接提示注入）有三条结论：

一，模型的通用能力和它抗攻击的能力基本不相关。榜单分数更高的模型不一定更难攻破，差别更多来自模型家族和训练配方。

二，在对话框里训出来的防御，换个入口就失效。同样一句恶意指令，用户自己打进去时模型会拒绝；藏在它检索到的网页或收到的邮件里送进去，就常常照做。

三，最反直觉的一条：在持续数月、上万人参与的比赛里，每次尝试的成功率大致保持恒定，并没有因为「好用的招都被用光了」而衰减。这是这两场特定竞赛里观察到的，不是已证实的普遍规律，但它意味着靠「把已知攻击一个个补掉」来收敛，目前看不到收敛的迹象。

## 读他近期论文时该记住数据是从哪来的

他同时是 CMU 的博士生和 Gray Swan 的联合创始人，公司由他导师 Fredrikson（CEO）和 Kolter（Chief Scientist）一同创办。他近两年的实证论文，数据大多来自公司自家平台上的红队赛事——规模是学术实验室做不出来的，但参与者是自愿报名来打比赛的人，攻击的模型也是公司客户送来测的那批，样本并非随机。他的 CTO 头衔见于 AI Frontiers 的作者简介和媒体报道，公司官网的 about 页目前没有列他。

**已核实来源**

- <https://andyzoujm.github.io/>
- <https://arxiv.org/abs/2307.15043>
- <https://arxiv.org/abs/2406.04313>
- <https://arxiv.org/abs/2408.00761>
- <https://arxiv.org/abs/2504.16980>
- <https://arxiv.org/abs/2410.09024>
- <https://arxiv.org/abs/2507.20526>
- <https://ai-frontiers.org/author/andy-zou>
- <https://www.grayswan.ai/about>
- <https://www.forbes.com/sites/rashishrivastava/2026/05/28/this-ai-startups-army-of-15000-hackers-pressure-test-claude-gpt-5-and-gemini/>
- <https://www.forbes.com/sites/sarahemerson/2024/10/29/this-hacker-team-is-bulletproofing-ai-models-for-companies-like-openai/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
