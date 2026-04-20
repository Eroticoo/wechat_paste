# 基于和式分式优化的在线主动故障诊断新型输入设计方法

![论文抬头：标题与作者](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/header.png)

- 作者：Bo Min，Feng Xu，James Lam，Chenchen Fan，Lin Lin
- 单位：香港大学机械工程系；清华大学深圳国际研究生院；香港大学深圳研究院；香港理工大学康复科学系
- 关键词：主动故障诊断；线性时不变系统；zonotope；分离趋势；和式分式优化
- DOI / 论文链接：https://doi.org/10.1016/j.automatica.2025.112559

## 1. 研究背景、问题定义与核心思路
### 1.1 研究动机与关键挑战
这篇论文研究的是离散时间线性时不变系统中的**在线主动故障诊断**。与被动诊断不同，作者并不满足于“观察系统自然响应再做判断”，而是希望在每个采样时刻主动设计输入 $u_k$，让不同故障模式对应的输出集合尽快拉开，从而更快排除错误模式。

难点在于，已有基于集合分离趋势的 AFD 方法虽然已经能利用 zonotope 输出集做在线排除，但目标函数往往只关注某种单一的“距离放大”或“尺寸压缩”，对哪些模式对最难区分、哪些模式对已经足够分开，利用得并不充分。于是本文要回答的问题是：**如何设计一个更贴近“成组输出集分离效率”的指标，并把它真正落到可在线求解的输入设计问题上。**

### 1.2 方法框架与核心思路
作者考虑带乘性执行器故障的离散时间系统，并把每个候选故障模式下的状态集与输出集都写成 zonotope 形式。系统主模型为





$$
x_{k+1} = A x_k + B G_i u_k + E w_k,
$$









$$
y_k = C x_k + F \eta_k,
$$





其中 $x_k$ 是状态，$u_k$ 是待设计输入，$y_k$ 是输出，$G_i$ 表示第 $i$ 个执行器故障模式，$w_k$ 与 $\eta_k$ 分别表示有界过程扰动和测量噪声。论文的核心不是单独优化某一个模式，而是让所有候选模式对应的输出 zonotope 形成更有利于**逐步排除**的几何布局。

作者提出的新思路是：不再只看一组输出集之间“总共离得多远”，而是引入一种成对比较的**改进加权分散度**，同时考虑不同 zonotope 的中心距离与总尺寸，并让上一时刻已经获得的分离信息参与本时刻权重更新。这样，输入会优先去拉开当前最关键、最难分辨的模式对。

### 1.3 主要创新点
**创新点 1：提出了更细粒度的输出集分离度量。** 论文把不同输出 zonotope 两两配对，利用“中心距离 / 总尺寸”的比值来刻画每一对集合的可分性，再引入权重形成整体分散度，比单纯放大某个全局距离更贴近在线排除逻辑。

**创新点 2：把输入设计写成和式分式优化问题。** 这一步把“增强分离趋势”从直觉指标推进到可计算优化目标，是本文真正的方法论核心。

**创新点 3：在理论上建立了从和式分式问题到等价参数化问题的桥梁。** 这让原本难处理的目标可以进一步转成可迭代求解的 0-1 混合整数二次规划流程。

**创新点 4：仿真不只展示单次轨迹，还给出 100 次试验统计结果。** 这点很重要，因为它让论文的结论不是“某一次图形看起来更快”，而是“在多组故障区间下平均诊断步数确实更优或至少不差”。

## 2. 核心方法与技术主线解析
### 2.1 整体技术路线
论文的技术链条可以概括为：**系统建模与模式集合递推 $\rightarrow$ 定义输出集分散度 $\rightarrow$ 把输入设计转成和式分式优化 $\rightarrow$ 通过等价变换与迭代算法在线求得输入。**

其中第一步不是背景铺垫，而是后续全部推导的前提。作者在系统模型之后给出了 **Assumption 1**，说明诊断窗口内真实系统只处于某一个固定执行器模式中，这个假设直接支撑后续“观测值不属于某模式输出集就删去该模式”的在线排除逻辑。

![系统模型、Assumption 1 与集合递推起点](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/technical_core_1.png)

在 zonotopic 描述下，模式 $i$ 的状态集与输出集递推可写为





$$
X_{k+1}^i = A X_k^i \oplus B G_i u_k \oplus E W,
$$









$$
Y_{k+1}^i = C X_{k+1}^i \oplus F V.
$$





这里 $X_k^i$ 和 $Y_k^i$ 分别对应故障模式 $i$ 下的状态集合与输出集合，$\oplus$ 表示 Minkowski 和。这个模型块的作用不是“给出记号而已”，而是为后文所有分散度定义、可分性分析和输入优化提供统一的集合传播接口。

### 2.2 关键技术块解析
**技术块 1：Theorem 1 是把和式分式目标真正变成可算问题的桥梁。** 论文先把输入设计目标写成关于多组比值的 sum-of-ratios 问题，再用 **Theorem 1** 说明：如果原问题取得最优解，那么它与一组参数化非分式优化问题之间满足严格的 KKT 关联。这个定理不是附属推导，而是从“好指标”到“能在线实现”的关键桥梁。

![Theorem 1：和式分式优化与参数化问题之间的桥梁](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/theorem_1.png)

从方法链上看，Theorem 1 的意义在于把分散度最大化目标从一个直接难解的问题，转成后续能够反复更新参数并嵌入优化器的形式。也就是说，定理本身并不直接给出输入，但它支撑了后面的算法化求解步骤。

**技术块 2：Algorithm 1 给出在线输入设计的实际求解流程。** 在定理建立的桥梁之上，作者进一步组织出每个采样时刻可执行的更新流程：根据当前集合信息形成参数，迭代求解参数化优化，再输出最优输入 $u_k^\ast$。因此，Algorithm 1 是把理论桥梁落到工程实现的那一步。

![Algorithm 1：在线输入设计求解流程](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/algorithm_1.png)

如果把本文看成一个完整系统，那么 **Assumption 1 提供可诊断前提，Theorem 1 提供可解性桥梁，Algorithm 1 则提供在线执行机制**。这三者并不是平行堆叠，而是前提、转换和实现三层关系。

## 3. 仿真结果与对比分析
### 3.1 仿真设置与对比对象
论文给出的是离散时间执行器故障场景下的在线 AFD 数值实验，并把本文方法与 Tan et al. (2022) 的既有方法进行对比。这里最值得关注的不是“最终是否诊断成功”，而是**在第几个采样时刻能把错误模式排除掉**，因为这直接反映在线主动输入设计是否真的提升了集合分离效率。

**证据 1：Fig. 3 直接展示了本文方法如何在早期采样时刻拉开集合。** 图中考察的是第一个故障执行器模式、故障大小 $f_1=0.6$ 的场景，作者展示了 $k=1$ 与 $k=2$ 两个时刻的输出集合几何关系。可以看出，随着输入在线更新，不同候选模式的 zonotope 很快产生可用于排除的分离。

![Fig. 3：本文方法在第一个故障模式下的在线 AFD 结果](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/figure_2.png)

这张图证明的是“本文输入确实把集合 separation tendency 推高了”，而不是只在数值指标上做了事后解释。对读者来说，这也是最直观的一张仿真图。

**证据 2：Fig. 4 给出与 Tan et al. (2022) 的过程级对比。** 对比方法在相同故障场景下直到 $k=3$ 仍保留更多重叠和歧义，而本文方法在更早阶段就形成了更强的排除趋势。由于两张图都展示了 mode-wise 输出集的形状变化，所以这是比最终统计表更“机理化”的对比证据。

![Fig. 4：与 Tan et al. (2022) 的在线诊断过程对比](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/figure_3.png)

### 3.2 主要结果与对比说明
在更大范围的统计结果上，作者没有停留在单次图形展示，而是做了 100 次试验平均步数比较，这让结论更稳。

**证据 3：Table 2 说明在多组 $f_1$ 区间下，本文方法多数情况下需要更少的平均诊断步数。** 尤其在中等故障幅值区间如 $f_1 \in [0.45,0.55]$、$[0.55,0.65]$ 时，本文方法分别需要 2 步和 2 步，而对比方法需要 2.76 步和 3 步。这说明改进分散度并非只对极端故障有效，恰恰在更难区分的中间区间更有价值。

![Table 2：不同 \(f_1\) 区间下的平均诊断步数](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/table_1.png)

**证据 4：Table 3 进一步检验了在其他故障参数区间组合变化下的方法稳定性。** 这里作者改变的是 $f_2,f_3,f_4$ 的设置并统计 100 次试验结果。结果显示，本文方法在多数组合下仍然保持不差甚至更优的平均诊断速度，这支持了“并非只针对单一场景调参”的判断。

![Table 3：不同 \(f_2/f_3/f_4\) 区间组合下的平均诊断步数](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@c4975436ddc585ea99b9adac2936d5975d9b8155/a-new-input-design-method-for-online-active-fault-diagnosis-based-on-sum-of-ratios-optimization/images/table_2.png)

综合来看，本文的证据链比较完整：**Fig. 3 和 Fig. 4 回答“为什么更快”，Table 2 和 Table 3 回答“是不是普遍更快或至少更稳”**。这也是这篇 brief paper 最有说服力的部分。

## 4. 面向不同对象的后续建议
1. 面向入门者
   标题：先把“集合排除”与“输入设计”这条主线读顺
   *核心建议：先围绕系统模型、Assumption 1、Theorem 1 和 Fig. 3 画一张“输入如何改变输出集几何关系”的流程图，再回头理解 sum-of-ratios 目标，入门会顺很多。*
   数学推导难度：中

2. 面向硕博学生
   标题：把单模式假设放宽到切换故障或多执行器联合故障
   *核心建议：最值得延展的方向是研究当真实故障模式切换、或多个执行器同时故障时，本文的分散度指标和在线优化框架还能否保持可解性与加速诊断能力。*
   数学推导难度：中高

3. 面向教授
   标题：把这篇文章作为“集合驱动输入设计”专题的起点论文
   *核心建议：带学生时可先要求复现 Fig. 3、Fig. 4 与 Table 2，再追问 Theorem 1 为什么必需，这样能先建立几何直觉，再进入分式优化与混合整数建模。*
   数学推导难度：中

## 5. 总结与评价
这篇论文的价值不在于单独提出了一个新指标，而在于它把**集合分离趋势、在线主动故障诊断、和式分式优化求解**真正串成了一条完整技术链。对 online AFD 这类既要实时、又要可解释的任务而言，这种“从集合几何到可算优化”的组织方式很有代表性。

从证据强度看，本文最值得保留的不是 abstract 里的结论句，而是 Section 3 的仿真链条：Fig. 3 和 Fig. 4 展示了更快排除的几何过程，Table 2 和 Table 3 又把这种优势扩展到多组统计场景。因而，这是一篇理论转换清楚、仿真支撑也比较扎实的主动故障诊断论文。
