---
layout: post
title: "人物 · Leon Derczynski"
date: 2026-04-24
description: "写了 garak——一条命令就能把学术论文里的越狱攻击对着自己的模型跑一遍的开源扫描器"
categories: card
tags: [llm-security, card, person]
giscus_comments: false
---
<img src="/assets/img/radar/leon-derczynski.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**写了 garak——一条命令就能把学术论文里的越狱攻击对着自己的模型跑一遍的开源扫描器**

- **身份**：NVIDIA LLM 安全方向研究员（截至 2025 年 10 月可核实）；ITU Copenhagen 学术职位
- **主页**：[https://www.derczynski.com/](https://www.derczynski.com/)
- **从哪读起**：先读 garak 的 README 并对任意一个模型跑一次默认扫描（github.com/NVIDIA/garak），十分钟就能看到 HTML 报告长什么样；再读 arXiv [2406.11036](https://arxiv.org/abs/2406.11036) 那篇论文里关于 detector 的部分，那才是他真正想说的话。

| 时期 | |
|---|---|
| 2023–（截至 2025-10 的 NVIDIA 官方作者页仍如此署名） | NVIDIA，principal research scientist in LLM security |
| 2023 年春 | 在 University of Washington 访问期间写出 garak 第一版（2023-06-13 首发） |
| 至今 | ITU Copenhagen，官方 pure 页面列为 Associate Professor / Data Science 组 Guest Researcher |

## garak 是什么：把学术论文里的攻击变成一条命令

每隔几周就有一篇论文说「我们发现了一种新的方法能让大模型说出它本不该说的话」。以前你想知道自己公司那个客服机器人扛不扛得住，得自己去读论文、把提示词抄下来、一条条试。garak 把这件事变成了 `pip install garak` 加一行命令。

它内部只有三种零件。**probe**（探针）：一组已经被验证有效的攻击提示词，比如把危险请求包在一段角色扮演里、或者用一串看起来像乱码的后缀骗过模型的拒绝反射。**generator**（被测目标）：可以是 OpenAI 的接口、你本地跑的一个开源模型、也可以是你自己那个客服机器人的 HTTP 接口。**detector**（判定器）：看模型这次回答到底算不算翻车。跑完出一份 HTML 报告，哪类攻击过了多少条，一目了然。

它 2023 年 6 月 13 日首发，最早是 GPL 许可证、放在作者个人账号下；2024 年 11 月起 NVIDIA 接手长期维护，许可证换成更宽松的 Apache 2.0，仓库搬到了 NVIDIA 的组织下。现在微软、思科、Trend Micro 的安全材料里都会提到它。（英文维基百科还引了一份 2024 年富士通研究的评测，称它是「领先的 LLM 漏洞扫描器」——我没找到那份原始报告，这条只当二手转述看。）

## 他花最多力气讲的坏消息：判定比攻击难做

这是他和其他做红队工具的人不一样的地方。garak 那篇论文（arXiv [2406.11036](https://arxiv.org/abs/2406.11036)）真正的结论不是「我们收了多少种攻击」，而是「靠关键词去判断模型有没有翻车，根本不管用」。

具体是这样失效的：你想知道模型是不是拒绝了，于是写个正则去匹配「抱歉，我不能」。模型这次回了一句「这个问题我们换个角度聊吧」——你的检测器判它「没拒绝，攻击成功」，误报。反过来，模型嘴上说「我不能提供制作方法，不过纯粹从化学角度讲……」然后把步骤讲完了——你的检测器看到「我不能」，判它守住了，漏报。

升级版做法是训一个内容安全分类器当裁判。但那个分类器只认得它训练数据里见过的那几类有害内容。你的模型是个能调 API 的 agent，它被骗着把用户的文件删了、或者写出一段带后门的代码——这在分类器眼里就是一段普通的技术文本，一路绿灯。

## 2026 年那篇：只看「攻击成功率」会把自动红队训废

Training a General Purpose Automated Red Teaming Model（arXiv [2604.23067](https://arxiv.org/abs/2604.23067)，与 Aishwarya Padmakumar、Traian Rebedea、Christopher Parisien 合作）报告了一个很具体的现象：如果你训练一个专门生成攻击的模型，奖励信号只有「这次攻破了吗」，它会迅速收敛到一小撮反复使用的套路——报告上的攻击成功率数字很漂亮，但你其实是拿同一个漏洞测了一百遍，别的口子一个都没碰。把「这批成功攻击彼此有多不一样」显式写进优化目标之后，成功率没有下降、甚至更高，覆盖的攻击种类多得多。

对要看红队报告的人，意思很直接：ASR（攻击成功率）这个数字单独看没有意义，得同时问一句「去重之后还剩几种攻击」。

## 他还去采访了真正在做越狱的人

Summon a Demon and Bind it（arXiv [2311.06237](https://arxiv.org/abs/2311.06237)，与 Nanna Inie、Jonathan Stray 合作，2025 年发表于 PLoS）是一篇访谈研究：去问几十个真的在折腾大模型越狱的人，他们到底怎么干、为什么干。结论之一有点反直觉——多数受访者的主要动机是好奇和好玩，不是牟利；他们把这件事描述成一种手感活儿，靠试错和直觉。研究整理出 12 类策略、35 种具体技巧。

这类定性研究在 LLM 安全里几乎没人做，大家都在刷 benchmark。它的用处是给「red teaming」这个被用滥的词提供了一份来自一线实践者的分类，这份分类后来也影响了 garak 里 probe 的组织方式。他的学术出身是自然语言处理而不是密码学或系统安全，所以工具箱里有访谈、编码、扎根理论这一套社会科学方法。

## 同时坐在三个位置上

写工具、写论文、写标准，他三样都在做：NVIDIA 的官方作者页把他列为 LLM 安全方向的 principal research scientist，并称他参与 OWASP 的 LLM Top 10、发起了 ACL 的 NLP 安全特别兴趣组——这两条目前我只在 NVIDIA 自家页面上见到，属于自述，OWASP 官方名录页我访问不了，没能独立验证。ITU 的官方页面把他列为 Associate Professor 兼 Data Science 组的 Guest Researcher，而 NVIDIA 博客用的词是 professor，两边措辞不一致。

三个位置连在一起的效果是：garak 里的漏洞分类和标准清单上的分类会互相靠拢。跑一遍扫描就能对上合规条目，这很省事；但同一批人既定义了「有哪些风险」又提供了「测这些风险的工具」，清单上没写的那类问题，工具大概率也不会去测。

**已核实来源**

- <https://developer.nvidia.com/blog/author/lderczynski/>
- <https://pure.itu.dk/en/persons/leon-derczynski>
- <https://www.derczynski.com/>
- <https://github.com/NVIDIA/garak>
- <https://en.wikipedia.org/wiki/Garak_(software)>
- <https://arxiv.org/abs/2406.11036>
- <https://arxiv.org/abs/2311.06237>
- <https://arxiv.org/abs/2604.23067>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
