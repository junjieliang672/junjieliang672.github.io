---
layout: post
title: "人物 · Scott Clinton"
date: 2026-09-10
description: "他不写攻击也不写防御，他负责让 OWASP 的 AI 安全清单按期发得出来、有人认"
categories: card
tags: [llm-security, card, person, indie]
giscus_comments: false
---
<img src="/assets/img/radar/scott-clinton.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**他不写攻击也不写防御，他负责让 OWASP 的 AI 安全清单按期发得出来、有人认**

- **身份**：OWASP GenAI Security Project 联合创始人、联席主席、董事会成员
- **主页**：[https://genai.owasp.org/team/scott-clinton/](https://genai.owasp.org/team/scott-clinton/)
- **从哪读起**：先读 2026-09-01 那篇项目公告（genai.owasp.org 站内），它一次性列了 2026 版 Top 10、新接收的 Agent Control Standard、新赞助商和社区规模——这四件事放在一页上，正好能看清这个项目是怎么运转的。
- **成名作**：把 2023 年那份志愿者写的 LLM Top 10 做成了一个持续发版的机构：[OWASP GenAI Security Project](https://genai.owasp.org/) 的联合创始人兼联席主席，现在有八条并行 initiative、自己的赞助商体系，和 [2026 版 LLM Top 10](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)。

| 时期 | |
|---|---|
| 至今 | OWASP GenAI Security Project 联合创始人、联席主席、董事会成员，负责 strategy / operations / growth |
| 至今 | SCVentures Ltd. President（据其 OWASP 与讲者页自述） |
| 此前约二十年 | 开源商业化与产业联盟方向的高管，参与过 Sun 的 NetBeans、Red Hat 的 gluster/ceph、Huawei Open-SDS、Hortonworks 等项目 |

## 他的角色是排期和拉人，不是写攻击

先把这件事说死：Scott Clinton 名下没有攻击手法，也没有防御机制。OWASP GenAI 的团队页逐字写着 Co-Founder、Co-Chair、Board Member，职责是 strategy、operations、growth。他自述的本职是 SCVentures 的 President，此前二十年做的是开源项目的商业化和产业联盟——NetBeans、gluster、ceph、Hortonworks 这一串。

这条履历解释了他往这个项目里带进来的东西：把一个社区文档按产品线管理。现在 OWASP GenAI 下面挂着并行的多条 initiative——LLM Top 10、Agentic 安全、AI 红队、威胁情报、数据安全、AIBOM、治理、以及一份给安全厂商产品分类的 Solutions Landscape——各自有 lead、有发版日期，还有按季度出的 Exploit Round-up。你在威胁模型文档里写「参考 OWASP LLM Top 10」的时候，引的是这条流水线的产物，不是某个研究者的判断。

## 2026 版换了排序口径

最初的 2023 版排的是当时基本还没在野发生的风险，顺序靠专家共识投票。2026 版（文档 8 月 4 日发布，9 月初正式公告）改成用大量已记录的真实 AI 安全事件来加权——项目自己的说法是「thousands of real-world incidents」，具体条数我没读到原文，这里不给数字。

排序因此明显动了。Excessive Agency 升到第三，理由是生产事故集中在 agent 系统上：模型输出直接去执行 shell 命令、调外部 API、操作数据库事务。Unbounded Consumption 上升四位，对应的是拿长思考链模型和多模态推理引擎做拒绝服务、把别人的推理账单打爆这类事。原来的 System Prompt Leakage 被扩写成 Hidden Context Exposure，覆盖范围从系统提示词扩到 RAG 的 schema、隐藏的策略逻辑这些用户看不见但会被套出来的上下文。

换口径本身有代价，值得引用者知道：「有多少条已登记事件」和「专家觉得多严重」不是一回事。一个风险要被计入，得先有人把它登记成 CVE 或写进某个 AI harm 数据库；那些走不进登记流程的（比如很多间接注入根本不被当成某个产品的漏洞）会被系统性低估。同时 2026 版把每条风险映射到 NIST、MITRE ATLAS、CWE——这一步说明它开始被拿去和合规框架对接，而不只是当宣传材料读。

## Agentic 那条线最后落到一个运行时标准上

Agentic 方向 2025 年 12 月出了 Top 10 for Agentic Applications（ASI01–ASI10，头两条是 agent 目标劫持和工具滥用），2026 年 6 月更新到 2.01。真正的变化是 2026 年 9 月接收捐赠的 Agent Control Standard（ACS，最初由 Zenity 开发）：它不是一份 PDF，而是规定 agent 框架该在哪些执行点暴露 middleware hook——输入、输出、工具调用、规划、记忆、生命周期——以及安全策略怎么以声明式的方式挂到这些 hook 上、在运行时被强制执行。代码和 schema 走 Apache-2.0，在 GitHub 上。

这一步从「告诉你有哪些风险」跨到「规定你的框架要留哪个口子」，也是最容易失败的一步：它的成立完全取决于 LangGraph、CrewAI 这类框架方肯不肯真的把 hook 实现出来。谁实现了、实现到什么程度，我没有查到证据，这里不下判断。

## 赞助商、三万人，以及谁在决定榜单上有什么

同一次公告里公布了新的 Gold Sponsor（F5、WitnessAI）和 Silver Sponsor（Evoke Security、Mondoo），以及社区超过三万人——这两个数字都是项目自己发布的。厂商深度参与是这份榜单能覆盖真实部署形态的原因：知道生产环境里 agent 具体怎么炸的，往往就是在卖检测产品的那些公司。

同时这是这类清单的老问题（Web Top 10 时代就有，不是针对这个项目的指控）：一个条目能不能进榜，和有没有厂商在卖对应的产品之间不是独立的。给一个可操作的读法——看每个条目时先分清它是在描述攻击面，还是在描述一个产品品类。两者混在同一份榜单里，后者的位次比前者更容易被推高。

## 引用时最常引错的三处

一，LLM Top 10 和 Agentic Top 10 是两份不同范围的文档。前者管模型/应用层——提示、输出处理、向量库、隐藏上下文泄露；后者管自主 agent 的行为和权限——目标被劫持、工具被滥用。只引前者做 agent 系统的威胁模型，会整层漏掉工具调用和身份权限。

二，条目是风险类别，不是漏洞实例。没有 CVSS，没有可利用性评分。把「已覆盖 LLM01–LLM10」当验收标准写进合同，验的是文档结构不是系统。

三，写年份。2023、2025、2026 三版的条目名和排序都不一样，System Prompt Leakage 这个条目名在 2026 版里已经改了。一个不带年份的「OWASP LLM01」现在是有歧义的引用。

**已核实来源**

- <https://genai.owasp.org/team/scott-clinton/>
- <https://genai.owasp.org/team_area/board-member/>
- <https://genai.owasp.org/contributors/>
- <https://genai.owasp.org/2026/09/01/owasp-genai-security-project-unveils-2026-top-10-for-llm-applications-new-agent-control-standard-and-sponsors-as-community-tops-30000-members/>
- <https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/>
- <https://genai.owasp.org/resource/agent-control-standard-acs/>
- <https://github.com/Agent-Control-Standard/ACS>
- <https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/>
- <https://genai.owasp.org/2026/04/14/owasp-genai-exploit-round-up-report-q1-2026/>
- <https://sessionize.com/scottclinton/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
