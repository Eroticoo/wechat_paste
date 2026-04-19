# 基于监督对比学习与同方差不确定性的时间序列分类

![论文抬头：标题与作者](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/header.png)

- 关键词：时间序列分类；多变量时间序列分类；监督对比学习；同方差不确定性；多任务学习；时频一致性
- DOI / 论文链接：https://doi.org/10.1109/TNNLS.2025.3607901

## 1. 研究背景、问题定义与核心思路
### 1.1 为何仅靠交叉熵难以支撑鲁棒的多变量时间序列分类
论文讨论的是多变量时间序列分类（MTSC）中的一个典型矛盾：仅用实例级标签和交叉熵训练分类器，虽然能学到判别边界，但对噪声标签、类内时序形变和跨场景分布变化都比较敏感。作者指出，现有对比学习方法虽然能学习不变表示，但若缺少任务引导，直接迁移到具体分类任务时又容易出现“表示学到了、分类没对齐”的问题。

因此，这篇论文要解决的核心问题不是单纯“把精度再抬高一点”，而是如何在**分类目标、时域不变表示、频域补充信息、跨域一致性约束**之间建立一个统一训练框架，使模型既保留监督分类能力，又能利用对比学习提升鲁棒性与泛化性。

### 1.2 方法框架与核心思路
作者提出的框架名为 **U-TFSCL**（uncertainty-based time-frequency supervised contrastive learning）。其基本思路可以概括为三层：

1. 在输入层同时构造时域分支与频域分支，用并行编码器提取两类表示。
2. 在表示层同时施加时域/频域内部的监督对比约束，以及时频之间的一致性对比约束。
3. 在损失层不再手工指定各任务权重，而是用同方差不确定性自动学习各任务的相对重要性。

文中还构造了一个真实世界的人机无人机交互（HDI）手势数据集，包含 20 名受试者和 45,376 条标注序列，用于补足公开 MTSC 数据在真实交互场景中的不足。

从符号上看，时域与频域编码后的共享表示写作

$$
z_i^T = P_T(h_i^T), \quad \tilde{z}_i^T = P_T(\tilde{h}_i^T), \quad
z_i^F = P_F(h_i^F), \quad \tilde{z}_i^F = P_F(\tilde{h}_i^F),
$$

其中 $P_T, P_F$ 是投影头，$h_i^T, h_i^F$ 分别对应时域与频域编码器输出。后续分类器直接使用时频融合表示完成最终预测，这保证了对比学习目标与分类目标是端到端耦合的，而不是两阶段割裂的。

### 1.3 主要创新点
**创新点 1：提出时频双域监督对比学习框架。** 论文不是把频域当作简单附加特征，而是把时域、频域和时频一致性统一放进同一个监督对比学习框架中，让类内紧凑、类间分离和跨域对齐同时发生。

**创新点 2：把同方差不确定性引入对比学习多任务加权。** 文中将原本需要人工调节的多损失权重转化为可学习的不确定性参数，让模型自己决定时间域对比、频率域对比、时频一致性和分类预测这几项任务的权重。

**创新点 3：在真实交互场景上补充了 HDI 数据集证据。** 这让论文不只停留在标准基准集上，还能验证方法在真实手势控制无人机的序列数据上是否成立。

**创新点 4：实验设计覆盖总体性能、消融、标签比例、正样本数和温度敏感性。** 证据链比较完整，不只是给一个最终精度表，而是解释“为什么这个框架有效”。

## 2. 核心方法与技术主线解析
### 2.1 整体技术路线
**技术块 1：U-TFSCL 总体框架。** 图 1 展示了整篇论文的主线。输入序列一方面进入时域编码器，另一方面经 FFT 转到频域编码器；随后在 TFSCL 模块里分别形成时域对比损失 $L_{T,i}$、频域对比损失 $L_{F,i}$ 与一致性损失 $L_{C,i}$；最后在 MTL 模块中结合分类损失 $L_{pre}$ 完成联合优化。这个结构的关键价值在于：作者并没有把频域分支当成后处理，而是让它和时域分支在表示学习阶段就发生交互。

![技术主线：U-TFSCL 总体框架](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/technical_core_1.png)

**技术块 2：总损失的四项组成。** 在进入不确定性加权之前，论文先把整体目标写成加权和：

$$
L_{\text{total}}
= \lambda_D L_{\text{sup}}^D
+ \lambda_{TF} L_{\text{sup}}^{TF}
+ \lambda_{pre} L_{pre}.
$$

这里 $L_{\text{sup}}^D$ 代表时域或频域内部的监督对比损失，$L_{\text{sup}}^{TF}$ 代表时频一致性监督对比损失，$L_{pre}$ 则是最终分类的交叉熵损失。这个写法本身并不新，但它清楚说明了论文要同时优化“判别性”和“不变性”两件事，而不是只做其中之一。

### 2.2 关键技术块解析
**技术块 3：时频增强不是装饰，而是构造监督对比任务的前提。** 图 2 对应的是时域随机 dropout 增强和频域随机扰动增强。作者想要的不是两条完全独立的支路，而是通过增强得到原始视图与增强视图，再把“同类样本更近、异类样本更远”的监督对比机制分别施加到时域和频域上。换句话说，这张图说明了正样本对与负样本对是如何被制造出来的，这是后续所有损失项成立的前提。

![关键技术块：时频域增强示意](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/technical_core_2.png)

**技术块 4：监督对比损失与时频一致性损失。** 图 3 中的公式页给出了论文真正的表示学习核心。式 (2) 把普通对比学习扩展成监督对比学习：同标签样本都可进入正样本集合 $P(i)$；式 (3) 进一步把时域投影表示 $z_i^T$ 与频域投影表示 $z_i^F$ 拉到共享潜空间中，构造时频一致性约束；式 (4) 与式 (5) 则把分类损失与三类对比损失合在一起。

$$
L_{\text{sup}}^D
= - \sum_{i \in I}\frac{1}{|P(i)|}\sum_{p \in P(i)}
\log
\frac{\exp(h_i^D \cdot h_p^D / \tau)}
{\sum_{a \in A(i)} \exp(h_i^D \cdot h_a^D / \tau)}.
$$

这部分的意义在于：作者没有只在时域做判别，也没有只做跨模态对齐，而是把**时域内部聚类、频域内部聚类、时频之间同类对齐**三种几何约束一起推到特征空间里。

![关键技术块：监督对比与时频一致性损失](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/technical_core_3.png)

**技术块 5：同方差不确定性驱动的自动加权。** 图 4 对应式 (9) 与式 (10)，这是论文与一般“多损失相加”方法真正拉开差距的地方。作者把各任务权重从人工指定的 $\lambda$，转成由任务不确定性决定的可学习参数 $\sigma_D, \sigma_{TF}, \sigma_{pre}$；再用 $s=\log(\sigma^2)$ 做重参数化，避免训练时数值不稳定。

$$
L_{\text{total}}
= \sum_{D \in \{T,F\}}
\left(
\frac{1}{e^{s_D}} L_D + \frac{s_D}{2}
\right)
+ \frac{1}{e^{s_{TF}}} L_{TF} + \frac{s_{TF}}{2}
+ \frac{1}{e^{s_{pre}}} L_{pre} + \frac{s_{pre}}{2}.
$$

这意味着：如果某一任务更难、更不确定，它对应的有效权重会自动降低；如果某一任务更稳定、更可靠，它在训练中的贡献就会被放大。论文借此避免了人工调权的高成本，也使得时域、频域与分类三类目标能够在同一个优化过程中自动协调。

![关键技术块：同方差不确定性加权公式](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/technical_core_4.png)

## 3. 实验结果与对比分析
### 3.1 基准结果与总体性能
**证据 1：Table III 说明 U-TFSCL 在自身框架测试中已经具备很强的总体性能。** 论文在 AW-A、AW-B、HAR-A、HAR-B、G-A 和自建 G-B 数据集上报告了 Accuracy、Precision、Recall、F1 四项指标。表中可见，标准测试设置下平均准确率达到 **96.53%**，LOSO 设置下平均准确率达到 **79.80%**。这说明该框架不仅适合常规随机划分，也能在更严格的跨主体泛化设定下维持可接受性能。

![实验结果：U-TFSCL 框架总体性能](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/table_1.png)

**证据 2：Table VII 给出与主流 MTSC 模型的横向比较。** 这张表最能体现论文的“外部竞争力”。按论文正文总结，在标准测试设置下，U-TFSCL 在六个数据集里取得 **5 个第一、1 个第二**；平均准确率达到 **96.65%**，明显高于 ConvTran 的 **94.76%** 和 1D-Resnet 的 **93.77%**。在 LOSO 设置下，U-TFSCL 的平均准确率为 **79.55%**，也高于 1D-Resnet 的 **72.88%** 和 ConvTran 的 **72.83%**。这部分证据可以支持“框架确实具有 SOTA 级竞争力”的结论。

![实验结果：与主流 MTSC 模型的横向对比](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/table_3.png)

### 3.2 消融与敏感性分析
**证据 3：Table V 直接验证了不确定性加权优于人工调权。** 在 AW-B 数据集上，标准设置下“不确定性”策略的准确率为 **94.76%**，高于“Equal”配置的 **93.97%**，也高于几组手工权重设置；LOSO 设置下，不确定性策略达到 **89.93%**，同样优于 Equal 的 **89.47%** 和各组手工配置。论文正文进一步指出，该提升大约在 **0.5%** 左右，虽然不是数量级跃迁，但足以说明自动权重学习确实优于固定经验权重。

![消融结果：MTL 模块中的不确定性加权](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/table_2.png)

**证据 4：Fig. 4 说明标签比例越高，监督对比学习收益越稳定。** 图中横轴是参与损失计算的标签比例，纵轴是 AW-B 上的准确率。随着标签比例从 0% 提升到 100%，标准测试与 LOSO 测试都呈现稳步上升趋势，其中从 0% 到 25% 的提升最明显。这说明作者的监督对比机制确实在利用标签信息增强类内聚合，而不是简单重复交叉熵的作用。

![敏感性分析：标签比例对性能的影响](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/figure_1.png)

**证据 5：Fig. 5 说明正样本数量并非越多越好，而是存在有效区间。** 当每个 anchor 的最大正样本数从 1 增加到 3 时，论文报告标准设置与 LOSO 设置分别提升 **6.49%** 和 **13.38%**；再继续增加时，收益开始趋于平缓。这个结果很重要，因为它表明监督对比学习中的“正样本扩张”并不是线性增益，过多正样本会受到 batch 大小和类别分布的限制。

![敏感性分析：最大正样本数对性能的影响](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/figure_2.png)

**证据 6：Fig. 6 说明温度参数在 0.07 或 0.1 附近效果最好。** 图中可以看到，当温度从极低值上升到 0.07/0.1 时，模型性能明显改善；之后整体仍保持稳定，没有出现剧烈崩塌。作者据此认为 U-TFSCL 相比部分对比学习方法，对温度参数的鲁棒性更强，这对工程复现是个积极信号。

![敏感性分析：温度参数对性能的影响](https://cdn.jsdelivr.net/gh/Eroticoo/wechat_paste@main/time-series-classification-based-on-supervised-contrastive-learning-and-homoscedastic-uncertainty/images/figure_3.png)

## 4. 面向不同对象的后续建议
1. 面向入门者
标题：先把“时频双分支 + 三类损失 + 自动加权”这条主线读顺
*核心建议：先对照 Fig. 1、式 (2)(3) 与式 (9)(10) 建立一张自己的方法流程图，再只在 AW-B 数据集上复现标准测试设置，重点理解“监督对比学习为什么能补交叉熵的短板”以及“不确定性加权为什么能替代人工调权”。*
数学推导难度：中

2. 面向硕博学生
标题：把自动加权机制扩展到更复杂的时序多任务场景
*核心建议：这篇论文最值得继续做的方向，不是简单换 backbone，而是研究当任务之间相关性更弱、标签更稀疏或存在跨域偏移时，不确定性加权是否仍能稳定工作；可以尝试把同方差不确定性替换为更细粒度的异方差或样本级不确定性建模。*
数学推导难度：中高

3. 面向教授
标题：把该文作为“表示学习与任务优化统一建模”的教学案例
*核心建议：带学生阅读时，建议把任务拆成三层检查点：第一层只复现 Table VII 的总体结果，第二层复现 Table V 与 Fig. 4-6 的消融和敏感性，第三层再讨论是否需要把时频双域、自动加权和真实场景数据集三部分同时保留；这样更容易控制研究范围并减少学生在大而全复现中的挫败感。*
数学推导难度：中

## 5. 总结与评价
这篇论文的价值，在于它把监督对比学习从“辅助表示学习技巧”推进成了一个**时域判别、频域补充、时频对齐和任务加权统一优化**的完整框架。与很多只报告最终精度的方法相比，它的优势在于证据链较完整：既有总体对比，也有针对权重分配、标签比例、正样本数量和温度参数的消融说明，因此“为什么有效”讲得比一般经验型方法更清楚。

从论文证据来看，U-TFSCL 的核心贡献不是某一个单独组件，而是三者联动：**时频双域监督对比学习**负责学习更稳的表示，**同方差不确定性加权**负责自动协调多任务目标，**HDI 真实场景数据集**负责把结论从标准基准推进到实际交互任务。就当前文中结果而言，这一组合是成立且有说服力的。
