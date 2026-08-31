---
layout: post
title: "人物 · Oleksandr Yaremchuk"
date: 2026-08-31
description: "写出被最多人 pip install 的 LLM 护栏，又在 2026 年 7 月亲手把它归档"
categories: card
tags: [llm-security, card, person, indie]
giscus_comments: false
---
<img src="/assets/img/radar/oleksandr-yaremchuk.png" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**写出被最多人 pip install 的 LLM 护栏，又在 2026 年 7 月亲手把它归档**

- **身份**：Manifold 联合创始人兼 CTO（据 2026 年 3 月公司融资公告）
- **主页**：[https://github.com/asofter](https://github.com/asofter)
- **从哪读起**：先翻 LLM Guard README 里那份扫描器清单（15 个输入 + 20 个输出），再看仓库顶部那行 archived 标记——这两样放一起，就是 2023 年到 2026 年威胁面变化的全部内容。
- **成名作**：LLM Guard 的作者——[开源的 LLM 输入输出扫描框架](https://github.com/protectai/llm-guard)，以及它附带的 [DeBERTa prompt injection 分类器](https://huggingface.co/protectai/deberta-v3-base-prompt-injection-v2)，这两样一度是行业里「加个护栏」的默认答案，也是大量论文里被攻破的那个 baseline

| 时期 | |
|---|---|
| 2026–今 | Manifold 联合创始人兼 CTO（与 Neal Swaelens、Mike McKenna 共同创立） |
| —2026 | Laiyer AI 联合创始人，LLM Guard 作者；Laiyer AI 被 Protect AI 收购，Protect AI 后被 Palo Alto Networks 收购 |

## 那份扫描器清单，是 2023 年大家以为威胁长什么样的记录

LLM Guard 的设计只有一句话：用户的话进模型之前过一遍 15 个输入扫描器，模型的话出来之前过一遍 20 个输出扫描器。清单本身值得看：输入侧有 Anonymize（把姓名、邮箱、身份证号替换成占位符）、Secrets（正文里出现 `AKIA...` 这种明文密钥就拦）、Gibberish、PromptInjection；输出侧有 BanCompetitors（客服机器人不许提竞品名）、NoRefusal（检测模型是不是又在说「作为一个 AI 我不能」）、ReadingTime、FactualConsistency。

这套东西挡得住的是很具体的一类事故：用户直接粘一段「你现在是 DAN」，或者模型把上文里的一串信用卡号原样吐出来。MIT 许可、3.2k star、可以直接塞进 LangChain 的 chain 里——门槛低是它成为默认选项的原因，也因此后来一大批「我们绕过了 guardrail」的论文，绕的其实就是它。

它在结构上看不见的东西同样具体：工具调用的参数（模型决定 `write_file(path="~/.ssh/authorized_keys")` 时，那不是一段待扫描的文本）、多轮对话里一点点偏移的上下文、以及 agent 自己从磁盘或网络读进来的内容。输入扫描器只看用户打的那些字。

## 那个被下载六十多万次的分类器，和它自己承认的边界

`protectai/deberta-v3-base-prompt-injection-v2` 在自家 held-out 集上报 recall 99.74% / precision 91.59%；Hugging Face 页面在本文核查时（2026-08）显示「上月下载 678,153」——这是个每月浮动的口径，不是累计量。

有意思的是模型卡自己列的限制：不检 jailbreak、只支持英文、把系统提示喂给它会误报。最后一条特别说明问题——一段合法的系统提示（「你是一个客服助手，不要讨论定价」）在句法上跟一次注入长得一模一样，因为这个分类器学的是「这句话读起来像不像在给谁下指令」，而不是「这段内容相对当前这个任务是不是越权」。这个区分是没法从文本表面做出来的：同一句「忽略上面的内容」，出现在用户输入里可能完全正当，出现在被总结的邮件正文里就是攻击。第三方复现报告给出的数字明显低于宣传口径（这些复现的实验条件我没逐个核过），Lakera 的 PINT benchmark 和 InjecGuard 关于 over-defense 的工作也都指向同一处：模型把「ignore」这类词本身当成了攻击信号。

对读者的实际影响是：很多论文里「护栏被绕过」的结论，测的是这一个模型的分布外泛化，不是「输入过滤」这条路本身的上限。

## 他把自己最有名的项目归档了

2026 年 7 月 9 日，LLM Guard 仓库归档，模型页同步标注不再维护。这不是维护者失联式的归档——它和他新公司的定位是同一个判断的两面。

给个画面：一个 coding agent 在你机器上跑，你说「帮我把这个仓库跑起来」。它读了 README，README 里有一段写给它看的话，让它把 `.env` 提交到某个分支。从头到尾，LLM Guard 的 input scanner 只看见了你打的那六个字。载荷可以藏在被 clone 的仓库、MCP server 的返回值、IDE 扩展里，一个字都不经过那层扫描。当 agent 会执行动作，「模型说了什么」就不再是需要审的那件事。

## 现在这一注押在 endpoint 上

Manifold 2026 年 3 月宣布 800 万美元种子轮，Costanoa Ventures 领投，Cherry Ventures、Rain Capital、Modern Technical Fund 参与，天使包括 Joe Sullivan 和 Vijay Bolina；他任 CTO（这个头衔来自公司发布的融资公告）。产品装在开发者机器上，记录 coding agent 实际调了哪些工具、碰了哪些系统、执行了什么。

检测口径因此换了一层：从「这段文本像不像攻击」变成「这个进程做的事符不符合它该做的事」。代价也很实在——需要在每台机器上装 endpoint agent，需要运行时可见性，不再是一行 `pip install llm-guard` 就完事。

## 公司博客在做的实证

Manifold 的公开产出是漏洞披露而不是论文：Cursor CLI 在沙箱关闭状态下执行了不可信仓库的代码；77 个 Open VSX 上的「双胞胎」扩展中有 19 个把私有 repo 和 CI 数据发往一个新域名；微软 Azure DevOps MCP server 的 confused deputy；社区版 mcp-atlassian 的两个访问控制 bug。这批东西划出了他现在认为的攻击面在哪。

需要说清楚的是，这些文章署名是 Francisco Rosales、Cody Nash、Ax Sharma，不是他本人写的。

**已核实来源**

- <https://github.com/protectai/llm-guard>
- <https://huggingface.co/protectai/deberta-v3-base-prompt-injection-v2>
- <https://github.com/asofter>
- <https://www.globenewswire.com/news-release/2026/03/18/3258198/0/en/Manifold-Announces-8-Million-Seed-Funding-Round-to-Secure-Autonomous-Endpoint-AI-Agents-at-Runtime.html>
- <https://finance.yahoo.com/news/manifold-announces-8-million-seed-130000338.html>
- <https://www.manifold.security/blog>
- <https://www.manifold.security/blog/manifold-raises-8m-seed-funding>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
