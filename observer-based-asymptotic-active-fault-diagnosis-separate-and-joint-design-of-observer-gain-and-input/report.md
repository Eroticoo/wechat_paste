# 基于观测器的渐近主动故障诊断：观测器增益与输入的分离设计和联合设计

![论文抬头：标题与作者](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/header.png)

- 作者：Feng Xu，Yiming Wan，Ye Wang，Vicenç Puig
- 单位：清华大学深圳国际研究生院；华中科技大学人工智能与自动化学院；墨尔本大学数学与统计学院；加泰罗尼亚理工大学机器人与工业信息研究所（CSIC-UPC）
- 关键词：主动故障诊断；观测器增益设计；联合增益与输入设计；集值观测器；zonotope
- DOI / 论文链接：https://doi.org/10.1016/j.automatica.2025.112548

## 1. 研究背景、问题定义与核心思路
### 1.1 研究动机与关键挑战
这篇论文研究的是**基于观测器的渐近主动故障诊断**。在 set-based 故障诊断里，很多已有方法把观测器增益设计和输入设计分开处理，因此虽然都能增强诊断能力，但没有真正联合挖掘“观测器结构 + 主动输入”的组合潜力。

本文的核心问题因此不是单独设计某个残差或某个测试信号，而是：**怎样在观测器框架下同时考虑增益与输入，使不同故障模式对应的输出估计集更快与原点分离，从而提高主动故障诊断效率。** 这里的难点在于，故障诊断性能是由集合几何、观测器收敛性和输入激励共同决定的，三者纠缠在一起。

### 1.2 方法框架与核心思路
作者考虑带执行器故障的离散时间系统，并围绕 set-valued observer, SVO 建立诊断框架。主系统模型写为



$$
x_{k+1} = A x_k + B G_i u_k + E \omega_k,
$$





$$
y_k = C x_k + F \eta_k,
$$



其中 $x_k$ 为状态，$u_k$ 为主动设计输入，$y_k$ 为测量输出，$G_i$ 表示故障模式，$\omega_k$ 与 $\eta_k$ 分别表示有界过程扰动与测量噪声。基于此，模式 $i$ 的 SVO 更新与输出集可写成



$$
\hat X_{k+1}^i = (A-L_k^iC)\hat X_k^i \oplus B G_i u_k \oplus L_k^i y_k \oplus E W \oplus (-L_k^i F V),
$$





$$
\hat Y_k^i = C \hat X_k^i \oplus F V.
$$



作者进一步引入“原点对 zonotope 的排斥度”这一新量，用来量化 set-based robust FD 的性能。直观地说，谁能让故障模式输出集更快把原点排除出去，谁就更利于主动诊断。

### 1.3 主要创新点
**创新点 1：提出了原点排斥度这一诊断性能量化指标。** 它把原本偏几何直觉的“集合离原点够不够远”变成了可用于优化设计的量。

**创新点 2：给出了观测器增益分离设计方法。** 这使得观测器本身不再只是一个被动估计器，而成为主动诊断能力的一部分。

**创新点 3：首次实现了 set-based AFD 场景下的观测器增益与输入联合设计。** 这是本文最明确的贡献点，也是标题中的“joint design”所在。

**创新点 4：仿真对比覆盖了检测时间地图、单次 AFD 过程以及不同设计方案之间的比较。** 所以这篇论文的证据部分并不单薄，尤其仿真图非常适合读者快速抓住“联合设计为什么更强”。

## 2. 核心方法与技术主线解析
### 2.1 整体技术路线
本文的技术路线是：**系统与 SVO 建模 $\rightarrow$ 定义原点排斥度 $\rightarrow$ 分离式增益/输入设计 $\rightarrow$ 联合增益与输入设计 $\rightarrow$ 在稳定性约束下实现主动诊断。** 因为这是典型控制与观测器论文，所以系统模型方程组本身必须显式写出，不能只靠截图。

![系统模型、SVO 更新与 Assumption 2.1](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/technical_core_1.png)

其中 **Assumption 2.1** 不是形式性假设，而是后续观测器设计和诊断可行性的前提。没有这类可检测性或可设计性条件，后面所有关于增益优化的讨论都站不住。

### 2.2 关键技术块解析
**技术块 1：Lemma 3.1 给出了后续优化的关键分解。** 该引理把和故障效应相关的集合项拆解成便于分析和优化的结构，因此它不是孤立的小结果，而是后文增益与输入设计能够形成优化问题的桥梁。

![Lemma 3.1：故障效应与集合项分解](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/lemma_1.png)

从证明链关系上看，Lemma 3.1 的作用类似一个代数中介：它把原本难直接操作的集合表达，整理成可用于性能指标和约束处理的分解式。

**技术块 2：Algorithm 1 给出无约束场景下的分离设计与联合设计求解流程。** 作者在这里组织了一个可执行的优化框架，用于分别求解 observer gain only、input only 以及 joint gain-input 的设计方案。算法块下方的 **Proposition 3.3** 还进一步给出稳定性约束，这使得“设计得更激进”不会以牺牲观测器稳定性为代价。

![Algorithm 1 与 Proposition 3.3：无约束联合设计及稳定性条件](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/algorithm_1.png)

这一块的角色非常关键：Lemma 3.1 提供分解桥梁，Algorithm 1 把桥梁转成求解流程，Proposition 3.3 则给出可实施边界。三者共同支撑了 section 3.2 中的 joint design。

**技术块 3：Algorithm 2 处理受约束场景下的区间搜索设计。** 当输入或设计变量存在约束时，论文并没有停留在理论描述，而是继续给出 constrained case 的算法流程，使得本文不只适用于“理想可调”场景。

![Algorithm 2：受约束情形下的设计流程](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/algorithm_2.png)

因此，本文的 section 2 不是简单并列几个算法，而是围绕“如何让观测器增益与主动输入一起提升诊断能力”逐层推进：先建立集合分解，再给无约束算法，再补上受约束实现。

## 3. 仿真结果与对比分析
### 3.1 仿真设置与对比对象
这篇论文的仿真部分很值得重视，因为它不是单一曲线，而是从**检测时间分布图、过程图到不同设计方案对比图**三层展开。对读者来说，这比只看最终成功率更能说明联合设计是否真正挖出了观测器方案的潜力。

**证据 1：Fig. 6 给出了 previous method 与 proposed method 的 fault-detection-time 地图。** 图中不同子图对应不同设计情形和采样时刻，直观展示了在参数平面上哪些区域更早触发故障检测。本文方法对应的可快速检测区域更大，这是联合设计优于既有方案的第一层视觉证据。

![Fig. 6：previous 与 proposed 方法的故障检测时间地图](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/figure_1.png)

**证据 2：Fig. 7 把两类方法的 FD time 做了更集中对比。** 这张图比 Fig. 6 更适合用来读“结论”：在相同场景下，本文方法能把检测时间压到更低，说明原点排斥度指标确实被转化成了更快的诊断效率。

![Fig. 7：两类方法的故障检测时间对比](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/figure_2.png)

### 3.2 主要结果与对比说明
**证据 3：Fig. 10 展示了第一个执行器故障 \(G_1\) 下的 AFD 过程结果。** 这张图里既有输出集几何变化，也有排斥度或相关性能量的时序变化，因此它连接了“诊断是否更快”和“为什么更快”这两个问题。读者可以直接看到联合设计如何让错误模式更快被排除。

![Fig. 10：第一个执行器故障 \(G_1\) 下的 AFD 结果](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/figure_3.png)

**证据 4：Fig. 12 对比了 section 3.2 与 section 3.3 中不同设计方法的结果。** 这张图的重要性在于它不是简单重复前面的单次过程，而是把“分离设计”和“联合设计”放到同一对比框架下，使标题中的 separate vs. joint 真正落到了可比较的仿真证据上。

![Fig. 12：分离设计与联合设计方法的对比结果](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/observer-based-asymptotic-active-fault-diagnosis-separate-and-joint-design-of-observer-gain-and-input/images/figure_4.png)

整体来看，Section 3 的仿真证据层次很清楚：**Fig. 6 和 Fig. 7 证明检测更快，Fig. 10 说明 AFD 过程确实更利于模式排除，Fig. 12 则回答联合设计是否真的优于只做分离设计。** 这正好对应了本文最想强调的贡献。

## 4. 面向不同对象的后续建议
1. 面向入门者
   标题：先把“原点排斥度”理解成一个几何化诊断指标
   *核心建议：入门时不要一开始就陷进所有算法细节，先从系统模型、SVO 更新和 Fig. 6/7 去理解“为什么把输出集更快推离原点就意味着更容易诊断”。*
   数学推导难度：中

2. 面向硕博学生
   标题：把联合设计推广到更复杂约束与更多故障类型
   *核心建议：可以进一步研究多执行器联合故障、切换故障或更复杂输入约束下，Algorithm 1 和 Algorithm 2 的可解性、保守性与实时性之间如何平衡。*
   数学推导难度：高

3. 面向教授
   标题：把这篇论文作为“观测器设计与主动诊断耦合”专题案例
   *核心建议：指导学生时可先复现 Fig. 6、Fig. 7 和 Fig. 12，再要求他们解释 Lemma 3.1 与 Proposition 3.3 如何分别支撑性能提升和稳定性约束，这样能把仿真直觉与理论链条真正接起来。*
   数学推导难度：中高

## 5. 总结与评价
这篇论文最突出的地方，是它把 set-based AFD 里原本常被分开处理的两件事真正耦合了起来：一方面利用观测器增益塑造集合传播，另一方面利用主动输入增强模式区分能力。相比只做单侧设计，这种 joint design 的研究价值明显更高。

从最终证据看，本文不是只靠一张结果图支撑主张，而是用检测时间地图、单次 AFD 过程图以及分离/联合设计对比图搭起了一条完整仿真链。因此，这是一篇理论主张鲜明、仿真证据也比较到位的主动故障诊断论文，尤其适合作为后续做 observer-based AFD 研究的参考起点。
