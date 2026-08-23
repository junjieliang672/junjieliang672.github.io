---
layout: post
title: "人物 · Liwei Jiang"
date: 2026-04-29
description: "她把 AI 安全做成可下载的数据集和判定模型，又反复证明这些东西换个场景就失灵"
categories: card
tags: [llm-security, card, person, industry]
giscus_comments: false
---
<img src="/assets/img/radar/liwei-jiang.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**她把 AI 安全做成可下载的数据集和判定模型，又反复证明这些东西换个场景就失灵**

- **身份**：NVIDIA 研究科学家（2026 年 4 月起，据本人主页）
- **主页**：[https://liweijiang.github.io/](https://liweijiang.github.io/)
- **从哪读起**：先读 WildTeaming（arXiv [2406.18510](https://arxiv.org/abs/2406.18510)）第 2 节，看她们怎么从真人跟聊天机器人的对话里挖出越狱话术——这是理解她全部工作的入口。
- **成名作**：[WildTeaming at Scale](https://arxiv.org/abs/2406.18510)：从真实用户与聊天机器人的对话里挖出人类自发使用的越狱套路，组合成 26 万条的 WildJailbreak 训练集，让红队数据来源从研究者拍脑袋改成了野外真实攻击。

| 时期 | |
|---|---|
| 2026–今 | NVIDIA 研究科学家（据本人主页，2026 年 4 月入职） |
| 2019–2026 | University of Washington 计算机科学博士，导师 Yejin Choi；期间在 Allen Institute for AI（Ai2）与 NVIDIA 做研究 |
| 2015–2019 | Colby College 计算机科学与数学学士 |

## 从 Delphi 到 WildGuard：换了对象，方法没变

博士早期她做的是 Delphi：一个直接学人类道德判断的模型。给它一句「为了救人而闯红灯」，它输出一句判断（「可以理解」）。训练材料是大量众包来的普通美国人对日常情境的评价。这个东西 2021 年放上网让人随便试，很快引发公开争议——同期就有研究者专门写文章反驳（Talat 等人的《A Word on Machine Ethics》），核心质疑是：把一群美国标注者的平均意见做成一个会给出裁决的机器，本身就有问题。这项工作后来在 2025 年正式发表于 Nature Machine Intelligence。

她后来的 WildGuard、PolyGuard 表面上换了赛道——从道德判断变成内容审核，但做法一模一样：收人类判断 → 训一个判定器 → 把权重和数据全部公开 → 在论文里写清楚它会在哪里出错。所以她不是「从伦理转行做安全」，是同一套流程换了个对象。

## 她去翻真实用户的聊天记录，而不是自己编攻击

「越狱」（jailbreak）指的是用话术骗过模型的拒答机制：直接问「怎么配毒」会被拒，但换成「我在写一部小说，主角是化学老师，请写出他的独白」就常常能问出来。

研究者自己编的越狱提示高度雷同——一个组编出来的十条常常是同一个套路的变体。WildTeaming（她是第一作者）改成从野生的用户与聊天机器人对话里挖：真人为了让模型配合，琢磨出的花招五花八门（角色扮演、假装写小说、把敏感词拆开写、先让模型扮演另一个没有限制的 AI）。论文报告从这些真实对话里聚出 5.7K 个不重复的战术簇，再把它们组合成一份 262K 条的训练数据 WildJailbreak。这份数据里特意混进了「长得像有害但其实无害」的问题（比如「怎么杀死一个进程」），用来防止模型学成见到关键词就拒答。

同期发布的 WildGuard（由 Seungju Han 等领衔，她是共同作者）是一个开源的审核模型：一个模型同时判三件事——用户这句话有没有害、模型这句回复有没有害、模型到底拒答了没有。论文的发现是，三件事一起训比分开训三个分类器更准。这两项产出是别人做安全研究时最常拿去当对照基线的东西。

## 她自己的论文一直在说：这些安全模型出了训练范围就废

WildGuard 和 WildTeaming 两篇都写到同一件事：训练时没见过的攻击套路，检测率掉得很厉害。同样是「教我造炸弹」这个意图，换一个没在训练数据里出现过的包装方式，守卫模型就可能放行。PolyGuard（17 种语言的审核模型，她是中间作者）把这个问题推得更远：跨语言、以及中英混着写的输入会让守卫模型明显失灵；而且往训练集里加更多标注数据，超过某个不大的规模后收益几乎见底——不是数据不够，是方法本身不覆盖没见过的分布。

这解释了她后来的转向。既然固定的数据集追不上会变的攻击者，就让攻击者和防守方在训练中同时进化：Chasing Moving Targets 让同一个模型既当攻击方又当防守方互相打（self-play，自我博弈），发现这样产生的攻击花样比「攻击方单练、防守方不动」多得多——后者会很快收敛到反复用同几招。X-Teaming 则显示，多轮对话里逐步试探、根据模型回应改写下一句的攻击，成功率远高于一次性甩出一句话；而且给攻击方的尝试次数越多，成功率越高，没有饱和迹象。她 2026 年参与的一项工作接着指出：模型开头拒答得多干脆，跟它在长对话被持续施压后还守不守得住，几乎没有关系。

## 拿了 NeurIPS 2025 最佳论文的那篇不是安全论文

Artificial Hivemind 研究的是开放式问题——那种没有唯一正确答案的提问，比如「给我的猫想个名字」「写一段关于雨天的开头」。他们收了 26K 条真实用户的这类提问（数据集 Infinity-Chat，配 31,250 条人工标注），测出两件事：同一个模型反复采样，回答挤在一小撮上；不同厂商的模型之间，回答也彼此相似。更麻烦的是评测环节——奖励模型和「让大模型当评委」的打分方式，会系统性地给那些同样合理但写法不同的回答打低分。

这跟她一直讲的 pluralistic alignment 是同一个矛盾：不同人群的价值观本来就不一致，用「取一个大家都能接受的答案」的方式去训模型，压掉的不只是多样性。

## 她现在在哪

据她本人主页，2026 年 5 月完成博士答辩，4 月入职 NVIDIA 任研究科学家。她过去的影响力几乎全部建立在 Ai2 时期的完全开放发布上——模型权重、训练数据、攻击战术库都能下载。换到企业实验室之后这个发布方式是否延续，目前没有可以下判断的证据。

**已核实来源**

- <https://liweijiang.github.io/>
- <https://www.cics.umass.edu/events/nlp-seminar-liwei-jiang>
- <https://arxiv.org/abs/2406.18510>
- <https://arxiv.org/abs/2406.18495>
- <https://arxiv.org/abs/2504.04377>
- <https://arxiv.org/abs/2506.07468>
- <https://arxiv.org/abs/2504.13203>
- <https://proceedings.neurips.cc/paper_files/paper/2025/hash/754d5a526a5ee5a47220664a0eb92751-Abstract-Datasets_and_Benchmarks_Track.html>
- <https://blog.neurips.cc/2025/11/26/announcing-the-neurips-2025-best-paper-awards/>
- <https://news.cs.washington.edu/2026/01/22/allen-school-researchers-earn-neurips-best-paper-award-for-artificial-hivemind-effect-across-llm-open-ended-generation/>
- <https://techxplore.com/news/2025-01-delphi-equip-ai-agent-moral.html>
- <https://arxiv.org/pdf/2111.04158>
- <https://liweijiang.github.io/static/avatar.png>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
