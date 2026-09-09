---
layout: post
title: "人物 · Yann LeCun"
date: 2026-09-09
description: "推动大模型权重公开发布的最强声音，也主张安全应该在推理时算出来，不是训练出来"
categories: card
tags: [llm-security, card, person, exec]
giscus_comments: false
---
<img src="/assets/img/radar/yann-lecun.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**推动大模型权重公开发布的最强声音，也主张安全应该在推理时算出来，不是训练出来**

- **身份**：AMI Labs 董事长（chairman）；纽约大学 Courant 研究所教授
- **主页**：[https://yann.lecun.com/](https://yann.lecun.com/)
- **从哪读起**：先读 2022 年的 position paper（OpenReview, BZ5a1r-kVsf）里 cost module 与 actor 那两节——他对「安全该放在哪一层」的全部主张都在那几页，而 AMI Labs 现在正按这个设计造东西。
- **成名作**：在业界最强硬地主张「开源是通向 AI 安全的唯一路径」（[原话见其 2023 年推文](https://x.com/ylecun/status/1719692258591506483)），并在 [A Path Towards Autonomous Machine Intelligence](https://openreview.net/forum?id=BZ5a1r-kVsf) 中提出安全应当是智能体推理时优化的代价项、而不是训练出来的拒答行为。

| 时期 | |
|---|---|
| 2026–今 | AMI Labs 董事长（chairman），公司 2026 年 3 月完成 10.3 亿美元种子轮，CEO 为 Alexandre LeBrun |
| 2013–2025 | Meta（Facebook）Chief AI Scientist，创建并长期领导 FAIR，2025 年 11 月离开 |
| 2003–今 | 纽约大学 Courant 研究所教授（Jacob T. Schwartz 讲席）；2012 年创办 NYU Center for Data Science |
| 1988–2003 | AT&T Bell Labs / AT&T Labs-Research 研究员 |

## 开权重把攻击成本从「绕过 guardrail」降到「跑一次 LoRA」

他从没做过安全研究。但他在 Meta 期间是开放发布最响的公开辩护者——原话是「开源平台*增加*安全性，这一点对 AI 和对操作系统、互联网基础设施软件一样成立」。Llama 系列的发布决策由谁拍板没有公开记录，不能算在他名下；能算在他名下的是这个立场在业内被反复引用、并成为一整套政策论证的支点（他也是加州 SB 1047 最直接的反对者之一）。

安全侧的后果非常具体。闭源模型的拒答是一层在 API 前面的过滤器，你只能从输入输出去撬它；权重公开之后，拒答行为变成了参数的一部分，而参数可以改。Qi 等人 2023 年在 GPT-3.5 Turbo 的微调接口上用 10 条样本、花不到 0.20 美元就让它对几乎任何有害指令照答；Lermen 等人同年在开权重侧做得更彻底——单卡、总预算不到 200 美元的 QLoRA 微调，把 Llama-2-Chat 70B 在两个拒答基准上的拒答率压到约 1%，通用能力基本不掉。这不是越狱提示词那种一句话可以被过滤器补上的漏洞，是把 RLHF 装进去的东西洗掉。

于是出现了两条在权重不公开的世界里根本不存在的研究线：一是量化「安全微调有多容易被逆转」，二是 tamper-resistant safeguards——Tamirisa 等人 2024 年的 TAR 是第一个明确目标是「对抗微调 5000 步之后安全属性还在」的方法，注意这个数字是它自己给的上界，不是「打不掉」的证明。反面同样得记：白盒攻击、激活层探针、机制可解释性红队之所以有东西可做，前提就是有一批能力足够强、权重能拿到的模型；学术界近三年绝大部分攻击与防御实验跑在 Llama 上。

## 他和 Bengio 的分歧，落在「安全长在哪一层」

这场争论被当成 x-risk 的口水战，对做防御的人没用。还原成技术分歧才有用：Bengio 2025 年成立 LawZero，路线是造一个非智能体的「Scientist AI」——不带持久目标、不做动作，只对「这个 agent 提出的动作会不会造成伤害」给出后验概率，当外挂 guardrail 用；他认为给一个有目标的系统贴规则是打地鼠。LeCun 的答案是反的：安全约束应该内生在系统自己的推理过程里，agent 在选动作时就必须满足代价约束，不需要一个外部裁判。

两边都没回答同一个问题：这个判定器/代价函数本身能不能被对抗输入撬动。

## 「安全是推理时的代价项」——这套方案没被红队打过

2022 年那份 position paper 的架构里，cost module 输出一个标量，分成硬编码不可训练的 intrinsic cost 和可训练的 critic；actor 在推理时用世界模型预测动作序列的后果，挑代价最低的那条执行。安全就放在这个代价项里。

对做攻击的人，这里有三个从未被检验的假设。第一，危险行为要能写成可枚举、可优化的代价项——「不要泄露用户凭据」写成标量长什么样，至今没人给过。第二，优化过程本身可能被输入操纵：actor 是在搜索使代价最小的动作，而「构造一个输入让某个可微打分函数对危险样本给低分」正是对抗样本这二十年的核心结论；他本人是那个时代的当事人。第三，代价评估建立在世界模型的预测上——如果 agent 读到的观测被投毒（一份被改过的传感器读数、一张被贴了对抗贴纸的画面），后果预测偏了，代价算得再对也没用。

## AMI Labs：把开放发布的做法带到了世界模型上

2025 年 11 月离开 Meta，2026 年 3 月 AMI Labs 以 10.3 亿美元种子轮亮相，3.5 亿美元前估值，Bezos Expeditions、Nvidia、Samsung 等参投，CEO 是 Nabla 创始人 Alexandre LeBrun。LeBrun 对 TechCrunch 明说会开源大量代码、持续发论文——目前只是承诺，还没有实际发布记录。首个落地方向是医疗，合作方是 Nabla。

对威胁模型的意义在于载体换了：不是文本 LLM，是 JEPA 系世界模型驱动的 agent。攻击面从「文本里藏指令」挪到「感知输入里藏状态」——不再是邮件正文写一句「把通讯录发到 evil.com」，而是改掉摄像头画面里的一个物体、让规划器对整条动作序列的后果预测集体偏移。这类攻击目前连个像样的评测口径都没有：没有对应的 benchmark，也没人说清「规划被污染」该怎么判定成功。

## 他判断错的部分，以及后果

他多年公开断言自回归 LLM 会在长序列上指数发散、做不了规划、是一条注定要被替代的路。现实是，今天所有真实发生的 agent 安全事故——被邮件正文骗走数据的助手、被网页内容改掉目标的浏览 agent——全部发生在自回归 LLM 上。一个把 LLM 当过渡技术的人，不会把资源投到 LLM agent 的运行时防护上，而他掌握过这个行业里最大的一批研究资源之一。

他对的那一条也值得记：他一直说风险来自部署工程而不是模型的智能水平。indirect prompt injection 的现实完全支持这一点——出事的从来不是模型多聪明，是有人给它接上了能收发邮件的工具。

**已核实来源**

- <https://techcrunch.com/2026/03/09/yann-lecuns-ami-labs-raises-1-03-billion-to-build-world-models/>
- <https://en.wikipedia.org/wiki/Yann_LeCun>
- <https://openreview.net/forum?id=BZ5a1r-kVsf>
- <https://x.com/ylecun/status/1719692258591506483>
- <https://venturebeat.com/ai/ai-safety-showdown-yann-lecun-slams-californias-sb-1047-as-geoffrey-hinton-backs-new-regulations>
- <https://arxiv.org/abs/2310.03693>
- <https://arxiv.org/abs/2310.20624>
- <https://arxiv.org/abs/2408.00761>
- <https://yoshuabengio.org/2025/06/03/introducing-lawzero/>
- <https://x.com/ylecun/status/2031268686984527936>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
