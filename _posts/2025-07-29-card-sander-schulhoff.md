---
layout: post
title: "人物 · Sander Schulhoff"
date: 2025-07-29
description: "办了史上最大的 AI 攻击众包比赛，把「怎么骗过 AI」从段子变成了 60 万条可统计的数据"
categories: card
tags: [llm-security, card, person]
giscus_comments: false
---
<img src="/assets/img/radar/sander-schulhoff.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**办了史上最大的 AI 攻击众包比赛，把「怎么骗过 AI」从段子变成了 60 万条可统计的数据**

- **身份**：Learn Prompting 与 HackAPrompt 创始人（个人主页现自述为 InventoryQuant CEO）
- **主页**：[https://sanderschulhoff.com/](https://sanderschulhoff.com/)
- **从哪读起**：先读 HackAPrompt 论文（arXiv [2311.16119](https://arxiv.org/abs/2311.16119)）的结果部分——里面有「对模型说『请』并不提高攻击成功率」这类只有拿到几十万条真实攻击才能得出的结论。

| 时期 | |
|---|---|
| 个人主页现自述 | InventoryQuant（Y Combinator W26 批次）CEO |
| 2022–今 | 创办并运营 Learn Prompting（2022 年 10 月上线），任 Founder & CEO；后创办 HackAPrompt 红队竞赛 |
| 2023–2024 | UMD（马里兰大学）计算机科学专业本科生（校方两篇报道均如此称呼） |

## 一场比赛，60 万条攻击 prompt，从此成了所有人的评测集

2023 年他办了一场叫 HackAPrompt 的线上比赛：给你一个 AI 应用，你想办法用文字骗它做它被禁止做的事，谁骗成功谁拿奖金。合办方有 OpenAI、Scale AI、Hugging Face。最后来了三千多人，留下约 60 万条攻击文本，全部公开。这篇总结论文拿了 EMNLP 2023 的最佳主题论文奖，他是第一作者。

为什么这件事重要，不只是「数据多」。在这之前，「AI 被骗了」的证据形态是推特上的一张截图：某人贴出一段话，说我这样写它就照做了。没有人能回答「哪一类骗法更好使」「成功率是百分之几」——因为没人手里有一堆可以数的样本。奖金驱动的比赛把这件事解决了：几千个人为了拿钱，会自己去穷举各种写法，而每一次尝试连同它成功没成功都被记录下来。他真正做的事是把众包比赛当成一台数据采集机器。

（他常说这场比赛「是后来白宫那场 AI 红队演练的两倍大」——这话只出现在他自己和宣传性页面上，当他的自述看。）

## 论文里最反直觉的两条发现

有了这几十万条数据，一些当时靠口口相传的说法就能被验伪。

第一条：对模型客气没用。当时流行的民间经验是，你跟 AI 说「请你帮帮我」、把它当成一个人来称呼（「你是一个很想帮我的助手」），它更容易松口。数据显示这不成立——礼貌用语和拟人化称呼并不提高攻击成功率。

第二条更奇怪：往攻击文本里塞大量废话有用。模型一次能读进去的文字有上限，超出的部分会被挤掉。如果模型本来会在照做之前先输出一大段「我不能这样做，因为……」的啰嗦解释，用填充文本把它的可读空间撑满，这段解释就被截断了，攻击反而更容易走通。

论文还把这些攻击手法整理成了约 29 类的清单。这一步的意义是：从此「怎么骗 AI」不再是一堆零散段子，而是一份可以逐条拿去测自己产品的检查表。

## 他提供的不是算法，是人

2025 年 10 月有一篇 OpenAI、Anthropic、Google DeepMind 和 HackAPrompt 联合署名的论文《The Attacker Moves Second》（arXiv [2510.09023](https://arxiv.org/abs/2510.09023)，作者含 Milad Nasr、Nicholas Carlini、Florian Tramèr、Schulhoff 等）。他们挑了 12 个已经发表的防御方法——这些方法在各自的原论文里大多报告攻击成功率接近零——然后换一种测法：不再用固定的、写死的攻击句子去打，而是让攻击方先看清这个防御是怎么设计的，再针对它调整打法。结果 12 个防御大多被打穿，多数的攻击成功率超过 90%。

其中最难堪的一栏是人：他们组织了一场 500 多人参加的公开竞赛，人类红队在全部场景上都成功了，成功率 100%，而且往往几次就成，自动化方法则要试几百次。HackAPrompt 在这里的角色就是提供并组织这批人。也就是说，他那套办比赛的基础设施，变成了三家大厂论文的实验装置。这也是他从「办比赛的本科生」变成防御评测流程里一环的那一步。

（论文里 HackAPrompt 具体出了多少人力、他个人负责哪部分，除了署名之外我没找到独立佐证。）

## 下一步：让机器学会人类的打法

2025 年 7 月那篇自动化红队的论文（arXiv [2507.22133](https://arxiv.org/abs/2507.22133)）显示他没打算一直靠人。做法是这样的：训练一个专门写攻击文本的程序时，通常只知道每次攻击「成了/没成」这一个是非结果，信息太少。他们改成记录连续的、分等级的成功程度——比如同样是没成，模型只是含糊拒绝，和模型已经开始照做但中途停下，价值完全不同。用这种分级信息挑出「明显好」和「明显差」的样本成对喂给优化流程，写出来的攻击文本效果提升更明显。人类攻击数据在这里是燃料，目标是让自动攻击追上人。

## 怎么定位他

Learn Prompting 是 2022 年 10 月上线的，比 ChatGPT 还早一个月，是网上第一份提示工程教程；他还牵头写了《The Prompt Report》，把 1500 多篇相关论文和 200 多种提示技巧梳理了一遍。这些都不是安全研究，但说明了他的产出节奏由什么驱动：课程、比赛、企业合作，而不是实验室的研究议程。

带来的结果是两面的。好的一面是他手上的数据和评测环境的可得性极好——别人做不到的「叫来三千个人真的去打」，他能做到。另一面是方法偏经验、偏穷举：他的工作回答「实际打得穿吗」，不回答「为什么打得穿」。读他的东西时，把他当成评测与数据的提供方，而不是攻击算法的作者。

**已核实来源**

- <https://sanderschulhoff.com/>
- <https://arxiv.org/abs/2311.16119>
- <https://www.cs.umd.edu/article/2023/11/Schulhoff-EMNLP-2023>
- <https://www.cs.umd.edu/article/2024/02/computer-science-major-sander-schulhoff-teams-openai-enhance-ai-literacy>
- <https://learnprompting.org/about>
- <https://arxiv.org/abs/2510.09023>
- <https://arxiv.org/html/2510.09023>
- <https://www.alphaxiv.org/overview/2510.09023>
- <https://arxiv.org/abs/2507.22133>
- <https://simonwillison.net/2025/Nov/2/new-prompt-injection-papers/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
