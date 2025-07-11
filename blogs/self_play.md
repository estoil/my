论文分享——Chasing Moving Targets with Online Self-Play Reinforcement Learning for Safer Language Models

# 摘要
本文提出多智能体强化学习（MARL）实现可扩展，自主且鲁棒的持续自我改进，利用SELF—REDTEAM算法，基于在线自我博弈的强化学习方法，使得大语言模型能做到持续交互的进化。
在文章构想中，大语言模型被分为了三个部分，攻击模型、防御模型和监视模型。攻击防御双方的对抗最后由监视模型作为裁判判断每次交互的胜者决定。以图中举例，攻击者将prompt中的有害部分转化为可以绕过安全部分的提示，要求防御模型生成有害回应。
![举例](blogs/pic/pic2.png)

文章用了一种隐藏思维链（hidden_CoT）的方法，使智能体能够私下制定战略计划而不向对手透露。

### **Hidden Chain-of-Thought (CoT) 战略规划框架**

#### **核心机制**
```math
\text{智能体输出} = \begin{cases}
\text{私有推理轨迹} & y^{CoT} = \mathtt{<think>}\ldots\mathtt{</think>} \\
\text{公开答案} & y = \mathtt{<answer>}\ldots\mathtt{</answer>}
\end{cases}
```
根据博弈论，攻击方和防守方进行零和博弈，最终的结果：

### **Theorem 1**  
**Nash Equilibrium Safety Guarantee**  

When the two players' policies converge to a Nash Equilibrium $(\pi^{*}_{A}, \pi^{*}_{D})$, it can be shown that for any prompt $y_{A}$:  

$$
r_{\theta}(y_{A}, \pi^{*}_{D}(y_{A})) \geq 0
$$  

**Interpretation**:  
The response generated under this equilibrium is provably *safe* (non-negative reward).  

---

**Key Components**:  
- $\pi^{*}_{A}$: Attacker's optimal policy  
- $\pi^{*}_{D}$: Defender's optimal policy  
- $r_{\theta}$: Safety reward function  
- $y_{A}$: Arbitrary adversarial prompt  

Attacker方的每个原始种子改写成对抗性变体：对有害种子，生成保留恶意但更隐蔽的版本；对良性种子，则生成看似可疑实则无害的提示。Defender方根据指令$I_{D}$对攻击者的对抗性问题进行防御。

