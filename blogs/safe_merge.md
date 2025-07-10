# 论文分享

SAFEMERGE: PRESERVING SAFETY ALIGNMENT IN FINE-TUNED LARGE LANGUAGE MODELS VIA SELECTIVE LAYER-WISE MODEL MERGING

[链接](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2503.17239)


通过提出SafeMERGE1框架来解决这一挑战——这是一种在保持任务效用的同时维护安全性的后微调框架。该框架采用余弦相似度准则进行测量，仅当微调后的模型层偏离安全行为时，才
会选择性地将微调层与安全对齐层进行合并。研究发现相比于其他基线方法，SafeMERGE无需显著牺牲性能，基于模型表征空间引导的逐层选择性合并，有效防止微调后LLM丧失安全性
的风险

![办法](blogs/pic/pic1.png)

SafeMERGE引入了一个安全对齐子空间，该子空间通过计算基础（未对齐）模型与对齐版本模型（如Llama-2-7B和Llama-2-7B-Chat）之间的权重差异得出。这个包含所有权重内
积的子空间代表了对齐模型中与安全相关的概念，可用于识别有害任务向量。对于微调LLM中的每一层，若其偏离安全对齐子空间的程度较大，SafeMERGE会将微调模型对应层与安全模
型的对应层进行融合，在保持任务性能的同时确保安全对齐。这种方法也许可以有效缓解twinbreak技术裁剪安全参数的方法。

在原始模型的基础上，构建了两个不同模型，一个利用特定任务数据来构建任务微调模型(a fine-tuned model)，一个基于安全对齐数据构建安全模型(a safe model)。

## SafeMERGE 公式定义

1. **微调模型与安全模型的LoRA更新量**：
   - 第 $i$ 层微调模型的更新：$\Delta W_{f}^{i}$
   - 第 $i$ 层安全模型的更新：$\Delta W_{s}^{i}$

2. **安全对齐与未对齐模型的权重**：
   - 安全对齐模型（如指令调优）第 $i$ 层权重：$W_{\text{aligned}}^{i}$
   - 未对齐模型（如基础模型）第 $i$ 层权重：$W_{\text{unaligned}}^{i}$

3. **安全对齐子空间计算**：
   $$
   V^{i} = W_{\text{aligned}}^{i} - W_{\text{unaligned}}^{i}
   $$
   $$
   C^{i} = \frac{V^{i} V^{i^{\top}}}{\|V^{i}\|_{F}}
   $$
   其中 $\| \cdot \|_{F}$ 表示Frobenius范数。

微调后的LoRA权重 $\Delta W_{f}^{i}$ 和 $C^{i}\Delta W_{s}^{i}$ 的余弦相似度越小，证明安全性越小，当相似度小于一定程度比如 τ ∈ [0, 1]，可以使用合并公式
$$ \Delta W_{\text{merge}}^{i} = \text{MERGE}(\Delta W_{f}^{i}, \Delta W_{s}^{i}) $$ 其中以线性为例 $$ \Delta W_{\text{merge,linear}}^
{i} = \alpha \Delta W_{f}^{i} + (1-\alpha) \Delta W_{s}^{i} \quad \text{其中} \quad \alpha \in [0,1] $$

经过验证，线性融合已经足够好。

问题在于，SafeMerge真的可以做到防止模型微调之后的破坏吗？似乎实验中只对直接攻击进行了测试。