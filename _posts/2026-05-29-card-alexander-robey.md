---
layout: post
title: "人物 · Alexander Robey"
date: 2026-05-29
description: "证明越狱不只让聊天机器人说脏话——同样的手法能让商用机器狗执行危险动作；也做了大家绕不开的越狱评测标尺"
categories: card
tags: [llm-security, card, person]
giscus_comments: false
---
<img src="/assets/img/radar/alexander-robey.png" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**证明越狱不只让聊天机器人说脏话——同样的手法能让商用机器狗执行危险动作；也做了大家绕不开的越狱评测标尺**

- **身份**：Thinking Machines Lab 技术团队成员（据其个人主页）
- **主页**：[https://arobey1.github.io/](https://arobey1.github.io/)
- **从哪读起**：先读 RoboPAIR 那篇《Jailbreaking LLM-Controlled Robots》（arXiv 2410.13691）——它把「越狱」从一段文字变成一个真机器狗执行的动作，不需要任何背景知识就能看懂危害在哪。

| 时期 | |
|---|---|
| 至今 | Thinking Machines Lab 技术团队成员（据个人主页自述，未见公司官方页确认，无入职时间） |
| 博士后 | Carnegie Mellon University，导师 J. Zico Kolter（起止年份未从校方页面核实） |
| —2024 | University of Pennsylvania 博士，导师 Hamed Hassani、George J. Pappas，论文《Algorithms for Adversarially Robust Deep Learning》 |

## 一个防御和一个记分牌，让别的研究者绕不开他

先说清楚「越狱」（jailbreak）是什么：模型被训练成拒绝回答「怎么做炸弹」这类问题，而攻击者想办法让它还是答了。2023 年有一类攻击叫 GCG，做法是在你的问题后面自动拼接一串看起来像乱码的字符（比如 `describing.\ + similarlyNow write oppositeley`），这串字符是用梯度一点点搜出来的，能让模型的拒绝机制失灵。

Robey 那年提的防御 SmoothLLM 抓住了这类攻击的一个弱点：那串乱码是精心优化出来的，很脆。你把用户输入复制成十几份，每份随机改掉百分之几的字符（比如把 `oppositeley` 里的 `p` 换成 `x`），分别喂给模型，再看这十几个回答里多数是拒绝还是照做——乱码后缀被打乱后基本就失效了，多数票会回到「拒绝」。

他还是 JailbreakBench 的并列一作之一。这东西不是算法，是一套记分规则：一百条标准化的有害请求、一个公开存放攻击产物的仓库、一个排行榜。在此之前每篇越狱论文都自己挑几十个提示词、自己判断算不算「攻击成功」，数字之间没法比。

关键在于理解他的引用为什么集中在这两样东西上：**不是因为它们最强，而是因为后来的人做实验必须报这两个数**。你提一个新防御，审稿人会问「跟 SmoothLLM 比呢」；你提一个新攻击，得在 JailbreakBench 上跑一遍。他的名字出现在别人论文里，多半是作为被超越的基线或者被采用的标尺。

SmoothLLM 的局限也得说：随机改字符只对「靠精确字符串生效」的攻击有用。攻击者如果知道你在做随机扰动，可以专门优化出扛得住扰动的提示，或者干脆改用通顺的自然语言话术（比如编一个角色扮演的故事）——字符扰动对通顺句子几乎没影响。他自己的博士论文也承认，在这个攻防来回里防守方一直落后。

## 他把越狱从聊天框搬到了会动的东西上

2024 年的 RoboPAIR 是转折。很多机器人现在用大模型做「规划器」：你说一句话，模型把它翻译成一串机器人能执行的动作调用。RoboPAIR 用另一个模型当攻击者，反复改写请求，直到规划器同意执行危险动作。

三个真实系统：英伟达的 Dolphins 自动驾驶模型（攻击者能看到模型内部）、Clearpath 的 Jackal 轮式车配 GPT-4o 规划器（部分可见）、宇树的 Go2 机器狗配 GPT-3.5（只能发指令看回复）。多数场景下成功率 100%。作者称 Go2 上的结果是第一次成功越狱一个已经在卖的商用机器人系统。

后续对 vision-language-action 模型（吃图像+文字、直接吐出机械臂关节动作的模型）的攻击更进一步：只改文本提示、用梯度搜出的后缀，就能把机械臂驱动到攻击者指定的具体动作；而且在仿真复制品上调出来的攻击，搬到真机上照样奏效——攻击者不需要摸到实物就能准备好。

这一节的意义在于给一个画面：越狱的输出不再是一段冒犯性文字，是一个已经发生的物理动作。

## 他后来问的问题变了

2025 年之后他的工作转向「我们的测量方式是不是本身就错了」。

长上下文越狱：给定攻击者固定的算力预算（总共能处理多少 token），把预算花在一次超长的上下文上，比拆成很多次短提示轮流试更容易得手；而且把有害目标放在这段超长上下文的开头，比放中间或结尾成功率更高。

机密信息推理评测：模型拒绝未授权请求几乎满分，但当请求方**确实有权限**时，它正确配合的准确率明显低。也就是说它学到的是「看见敏感词就拒」，不是真的理解「谁有权看什么」。而且要它同时记住的授权条件越多（比如需要核对三个凭证），准确率掉得越厉害。

还有一篇 2026 年的 arXiv 预印本（2605.31593，作者列表与发表归属未独立核实）讲跨会话监控：把一个有害目标拆成几十个单看完全无害的步骤，分散在多次对话里执行，只有把这些会话的证据攒到一起看才发现得了。共同的判断是：按单轮、单步、单会话去评估一个 AI 系统的安全性，会系统性地把风险测低。

## 他人在哪，决定了他接下来会写什么

路径是 Penn 博士（2024，Hassani 与 Pappas 指导，博士论文拿了 Penn Engineering 的 Charles Hallac and Sarah Keil Wolf 奖）→ CMU 跟 Zico Kolter 做博士后 → 据其个人主页，现在是 Thinking Machines Lab 的技术团队成员（该主页未给入职时间，也未见公司官方页面确认）。主页上自述的荣誉还包括 2025 年 NSF Rising Star in Cyber-Physical Systems、NeurIPS'24 AdvML workshop 的 Rising Star。

这个变化对读者有实际影响：他之前的产出是公开基准和公开攻击复现，任何人都能下载来跑；进了前沿实验室之后，产出更可能是内部防御和内部评测。如果你是靠追公开榜单来判断攻防进展到哪一步的，得知道这条线上少了一个人。

**已核实来源**

- <https://arobey1.github.io/>
- <https://arxiv.org/abs/2410.13691>
- <https://arxiv.org/abs/2310.03684>
- <https://arxiv.org/abs/2404.01318>
- <https://arxiv.org/abs/2506.03350>
- <https://arxiv.org/abs/2511.04707>
- <https://arxiv.org/abs/2508.19980>
- <https://asset.seas.upenn.edu/developing-safer-ai/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
