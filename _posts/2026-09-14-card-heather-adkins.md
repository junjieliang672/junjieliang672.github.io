---
layout: post
title: "人物 · Heather Adkins"
date: 2026-09-14
description: "在 Google 决定「AI 找到的漏洞」要拿出什么证据才算数，并押注漏洞发现会先倒向防守方"
categories: card
tags: [llm-security, card, person, exec]
giscus_comments: false
---
<img src="/assets/img/radar/heather-adkins.webp" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**在 Google 决定「AI 找到的漏洞」要拿出什么证据才算数，并押注漏洞发现会先倒向防守方**

- **身份**：Google，VP, Security Engineering
- **主页**：[https://blog.google/authors/heather-adkins/](https://blog.google/authors/heather-adkins/)
- **从哪读起**：先看 [un]prompted 2026 上她和 Four Flynn 的合讲 [Evaluating Threats & Automating Defense](https://www.youtube.com/watch?v=B_7RpP90rUk)——Google 在这里第一次把「AI 找漏洞」的验收标准和「AI 打补丁」的验证流水线摊开讲。
- **成名作**：美国 CISA [Cyber Safety Review Board](https://www.cisa.gov/resources-tools/groups/cyber-safety-review-board-csrb) 副主席（连任两届，2025 年 1 月随委员会全员被解职），主持完成 Log4j、Lapsus$、微软 Exchange/Storm-0558 三份国家级事故复盘——这是美国第一次尝试用类似航空事故调查的方式、不追责地拆解一起网络事故。

| 时期 | |
|---|---|
| 至今 | Google，VP, Security Engineering（blog.google 作者页逐字）；多处第三方页面另称其领导 Google Office of Cybersecurity Resilience |
| 2021–2025.1 | CISA Cyber Safety Review Board 副主席（Deputy Chair），2025 年 1 月 21 日随委员会全体成员被解职 |
| 2020 | 《Building Secure and Reliable Systems》（O'Reilly）合著者 |

## 她拍板的是「什么算证据」，不是「用什么模型」

Big Sleep 的技术来自 DeepMind 和 Project Zero，CodeMender 也不是她写的。她的位置在另一头：Google 自家攻击面、这些系统的产出、以及对外披露的口径，最后由同一条汇报线签字。对 LLM security 读者有用的不是她管多少人，而是她是少数几个能决定「一份 AI 生成的漏洞报告要长成什么样才允许发出门」的人——这条链路上卡住的从来不是模型能力，是上游维护者收到报告时信不信。

## 门槛：造不出可复现的 PoC，就不算一个发现

2025 年 8 月，Big Sleep 一次性报出 20 个开源项目的漏洞，FFmpeg、ImageMagick 都在里面。Google 的公开说法是这些漏洞由系统自主发现并自主复现，但每一份在提交给上游之前都过了一遍人工专家复核。在 [un]prompted 2026 的讲法里，判定标准被说成：为每个 finding 构造一个能跑通的 proof-of-vulnerability，因此 false positive 为零（这是 Google 自述，没有第三方复现或独立审计；这段引述本身也来自会议现场摘要，不是官方转录）。

为什么这个口径值得单独拿出来说：现在绝大多数「LLM 能不能找漏洞」的工作，指标是某个 benchmark 上的通过率——模型报了一个位置，标注说这里确实有洞，就记一分。Adkins 这边换了个更贵的判定：报告要带一份能让维护者复现崩溃的输入，不然不发。同一个系统在这两种口径下的「成绩」可以差出一个数量级。

配套的 CodeMender 走的是对称逻辑：补丁生成不难，难的是证明补丁没把语义改坏。据现场摘要，它的验证是四道关——fuzzing、形式化验证、differential testing（同一批输入喂给打补丁前后的两个版本，比对输出是否逐位一致）、再加一遍 LLM 复审。

## 这套门槛立刻挨了一记，而反对意见不是技术上的

2025 年 11 月，FFmpeg 维护者公开把 Big Sleep 的报告称作「CVE slop」。争议的那个漏洞在 LucasArts Smush 解码器里，影响 1995 年一款游戏开头十几帧的播放。维护者的话很直接：付高薪工程师的公司把活推给志愿者，真想降风险就连补丁一起交，不要只图攒一份「我们检测到了」的记录。

这件事说明「可复现 PoC」解决的只是报告的技术可信度，没解决接收端的容量问题——当发现侧的边际成本被 AI 压到接近零、修复侧还是无偿人力时，提高报告质量并不会让这个缺口变小。

## 她的威胁模型是对称的，而她没有押 prompt injection

她 2024 年对 The Record 的说法是「我们在用大模型，他们也在用」，并明确认为国家级行为体已经把 LLM 用在漏洞挖掘上。据 [un]prompted 现场摘要，她的判断更进一步：AI 系统很快会把每个系统里的每个漏洞都找出来。

从这个判断推出的动作不是「加强检测」，而是换架构——她反复讲内存安全问题从 1960 年代就已经清楚，不该在旧地基上继续打补丁。这条路线跟学术圈的分工差别值得注意：她不押注 prompt injection 在现有 agent 架构下有解，她押的是把内存安全漏洞的存量清空，让一个已经被间接注入、已经在执行恶意指令的 agent，撞不到可用来提权的内存错误。想知道 Google 的钱实际投在哪一侧（发现+修补 vs. 模型层对齐防御），这是最直接的信号。

## SAIF/CoSAI 给了什么、没给什么

她是 Coalition for Secure AI 的主推者之一（2024 年发布，挂在 OASIS 下）。2025 年 9 月，Google 把 SAIF 的数据捐给 CoSAI，成了 CoSAI Risk Map。SAIF Risk Assessment 是一份覆盖数据投毒、prompt injection、模型来源篡改的风险自评清单加治理流程——它不是防御机制，不设任何攻击者预算假设（比如「攻击者能不能改训练数据」「有几次查询机会」它一概不问），也不定义任何评测判定标准。当技术方案看会失望；它的用处是让采购方和供应商在合同里指着同一份清单说话。捐出去而不是留在 Google 名下，也是同一个理由：一份只有一家公司用的清单，在采购谈判里没有分量。

## 她做过国家级事故复盘，而那套机制现在不存在了

CSRB 副主席任上，她和主席 Rob Silvers 一起出了 Log4j、Lapsus$、微软 Exchange/Storm-0558 几份报告。Storm-0758 那份对微软的内部工程实践下了相当重的结论。2025 年 1 月 20 日，Salt Typhoon 调查还在进行中，全体成员被解职，委员会实质停摆。

这一节和 AI agent 安全的关系是直接的：当第一起「agent 被间接注入、造成真实损失」的事故发生时，谁来做那份不追责的技术复盘、报告写到什么颗粒度、厂商肯不肯交日志——目前业内唯一近距离干过这件事的人里就有她，而她干这件事所依托的那个机制已经被关掉了。

**已核实来源**

- <https://blog.google/authors/heather-adkins/>
- <https://techcrunch.com/2025/08/04/google-says-its-ai-based-bug-hunter-found-20-security-vulnerabilities/>
- <https://www.youtube.com/watch?v=B_7RpP90rUk>
- <https://medium.com/@thecyberarchive/top-8-talks-from-un-prompted-2026-every-security-practitioner-should-know-3e85681b9a09>
- <https://itsfoss.com/news/ffmpeg-google-fiasco/>
- <https://news.slashdot.org/story/25/11/11/1947215/ffmpeg-to-google-fund-us-or-stop-sending-bugs>
- <https://therecord.media/healther-adkins-interview-future-generations>
- <https://www.securityweek.com/dhs-disbands-cyber-safety-review-board-ending-one-of-cisas-few-bright-spots/>
- <https://en.wikipedia.org/wiki/Cyber_Safety_Review_Board>
- <https://www.oasis-open.org/2025/09/16/google-donates-secure-ai-framework-saif-data-to-coalition-for-secure-ai/>
- <https://www.rsaconference.com/experts/heather-adkins>
- <https://defcon.org/html/defcon-34/dc-34-speakers.html>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
