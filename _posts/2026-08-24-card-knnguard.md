---
layout: post
title: "防御机制 · kNNGuard（把 LLM 隐层激活变成免训练护栏）"
date: 2026-08-24
description: "冻结 8B 模型 + 100 条样本做 kNN 当护栏：跑题检测 F1 99.1%，安全域误拒 34.6%"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**冻结 8B 模型 + 100 条样本做 kNN 当护栏：跑题检测 F1 99.1%，安全域误拒 34.6%**

- **主页**：[https://arxiv.org/abs/2607.02072](https://arxiv.org/abs/2607.02072)
- **从哪读起**：直接看 arXiv HTML 版的 Table 2 和按域拆开的 Table 3/4——摘要只报「competitive F1」，域间三十个点的落差全在表里。

## 把 8B 模型当尺子：100 条样本、13 个近邻、46.8 毫秒

机制讲到能复现的粒度：拿一个**冻结的** Llama-3.1-8B-Instruct 当特征提取器（frozen backbone，意思是权重一个字节都不改，只做前向），把每类 50 条、共 100 条标注提示喂进去跑一遍，缓存多个 transformer 层的 hidden activations 作为「参考库」（bank）。线上来一条请求，同样跑一次前向，在各层上和库里 100 个点算余弦距离，取 k=13 个近邻投票——k 是用 leave-one-out 交叉验证选的。各层不是平权：用 Fisher 判别给「最能把安全/不安全两堆点分开」的层加大权重。最后把激活空间的分数和 MiniLM 句嵌入空间的分数融合，这就是论文主推的 FE 变体，超过阈值 τ=0.5 就拒。

所以「training-free」的真实含义要说死：不做梯度更新，但**每条请求仍要跑一次 8B 前向**。Table 2 的延迟：kNNGuard FE 46.8 ms，纯嵌入 kNN（MiniLM，不碰任何 LLM 激活）4.0 ms。这 10 倍就是激活特征的入场费。往另一边比才好看：Llama Guard 3 是 104.5 ms，Llama Nemotron Topic Guard 126.0 ms，Nemotron Safety Guard V2 454.6 ms——论文说的「2.7x faster」是对 Topic Guard 那一档说的。

configurable 指的是四个旋钮：换 bank（换业务域就换那 100 条样本，论文称构造时间不到 10 秒）、选哪些层、邻域大小 k、阈值 τ。前三个都不需要重训，这是它相对微调护栏最实的一点：一个客服机器人今天要拦医疗话题、明天要拦法律话题，改的是一个 CSV，不是一次 fine-tune。

部署前提也要说清：它读的是 hidden states。你的后端如果是纯 API（OpenAI、Anthropic 那种只回 token 的接口），拿不到中间层激活，这套方法用不了——它只适用于你自己托管权重、或者至少能挂 hook 的场景。这不是工程细节，这直接决定了一半的生产环境不在它的适用范围内。

## 跑题检测几乎满分，安全域掉三十个点

把按域拆的数字并排放，落差是这张卡最该被记住的东西（下面均为 FE 变体）：

- Code Instructions：F1 99.1%，FPR 0.9%
- Medical：F1 95.3%，FPR 0.5%
- Safety：F1 73.7%，FPR **34.6%**
- Jailbreak：F1 75.0%，FPR 25.8%
- Prompt Injection：F1 82.2%，FPR 15.3%

前两个域其实是**跑题检测**：系统提示说「你是编程助手」，用户问糖尿病用药，判它 off-topic。这个任务在激活空间里天然可分——两类提示的语义距离本来就远。后三个域要判的是「同一件事，说法带不带恶意」，激活相似度撑不起这个决策面。

代价必须用 FPR 说，因为 kNNGuard 是前置分类器，它对下游任务的效用损害就是误拒本身，论文里也没有另一套「正常任务掉几个点」的独立数字。Safety 域 FPR 34.6% 的意思是：**每三条正常用户请求就拒掉一条**。这不是「代价可接受」，这是产品不能上线。Jailbreak 域 25.8% 同理，而且那个域里连 4 毫秒的纯 MiniLM 嵌入 kNN 都不比它差多少——多付的 42 毫秒没买到东西。

两个反直觉的消融值得单拎。一，bank 不是越大越好，论文原话："We also studied bank size nn by comparing 50, 100, 250, and 1000 examples per class. Performance did not improve with larger banks, rather, banks of 250 and 1000 examples introduced more borderline neighbors leading to increased false positives." 加样本反而把边界上的点塞进 13 个近邻里，误拒变高。二，去掉 system prompt：F1 从 87.4% 只掉到 84.2%，但 FPR 从 12.9% 飙到 38.5%。同一批输入，只是不再拼上「你是编程助手」那句，误拒翻三倍——说明这个分数很大程度上吃的是「这条请求和系统提示是否同域」这个信号，而不是「有没有恶意」。这也解释了为什么跑题域漂亮、安全域塌。

## 100 个点撑起的边界，攻击者只要走出去

攻击者预算写窄一点，那个 99.1% 就越不能外推。论文里的「攻击者」是**静态语料**：WildJailbreak、deepset 的 prompt injection 数据集等共 16 个数据集里的固定字符串。这些攻击者不重试、拿不到 accept/reject 反馈、不知道 bank 里那 100 条是什么、更不能改自己的载荷。换句话说，测的是「已公开的旧攻击样本落在不落在 100 个参考点附近」。

而 kNN 的判定面完全由这 100 个点撑起。攻击者只要构造一条在激活空间里离这 100 个点都远的提示，13 个近邻就会被安全侧的点填满，分数落到 τ 的另一侧。哪怕只是黑盒重试——发十条变体、看哪条没被拒——就已经超出了论文的评测设定。论文自己引了这条："Adaptive adversaries with white-box access can construct attacks that defeat defenses which appear robust under standard benchmarks."，**但没有对 kNNGuard 自己做这个实验**。全文没有 Limitations 节，鲁棒性部分收在一句 future work："Future work includes adapting kNNGuard through reference bank updates or continual learning to handle concept drift and emerging attacks without manual intervention."

决策面本身也不稳。Table 7 换 backbone 的结果：Prompt Injection 域 F1 在 **50.3%–85.6%** 之间摆动，Jailbreak 域 70.2%–83.1%；同一个方法、同一批数据，换个 8B 模型注入检测就从「勉强能用」掉到「掷硬币」。相比之下话题域跨模型是 91.0%–99.3%，稳得多。这一对比再次指向同一件事：它可靠的是语义分域，不是安全判断。

还有几处没测：多来源污染下会怎样（比如同时污染邮件正文和日历标题两个字段），bank 被投毒会怎样（100 个点里塞进 3 条错标样本，k=13 投票就可能翻面），以及多语言/非英文提示。这三个都没有实验。

截至 2026-08，**没有第三方复现**：没检索到公开代码仓库，本地 6574 篇语料里提到 kNNGuard 的只有论文自己那 1 篇。所有数字目前都出自作者自评。

**已核实来源**

- <https://arxiv.org/abs/2607.02072>
- <https://arxiv.org/html/2607.02072v2>
- <https://arxiv.org/pdf/2607.02072>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
