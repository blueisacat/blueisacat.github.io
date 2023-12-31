---
layout: default
title: ML入门-算法篇(1)
parent: ML
---

### 算法篇

机器学习领域有一条"没有免费的午餐"定理。简单解释下的话，它是说没有任何一种算法能够适用于所有问题，特别是在监督学习中。

例如，你不能说神经网络就一定比决策树好，反之亦然。要判断算法优劣，数据集的大小和结构等众多因素都至关重要。所以，你应该针对你的问题尝试不同的算法。然后使用保留的测试集对性能进行评估，选出较好的算法。

当然，算法必须适合于你的问题。就比如说，如果你想清扫你的房子，你需要吸尘器，扫帚，拖把。而不是拿起铲子去开始挖地。

大的原则

不过，对于预测建模来说，有一条通用的原则适用于所有监督学习算法。

机器学习算法可以描述为学习一个目标函数f，它能够最好的映射出输入变量X到输出变量Y。有一类普遍的学习任务。我们要根据输入变量X来预测出Y。我们不知道目标函数f是什么样的。如果早就知道，我们就可以直接使用它，而不需要再通过机器学习算法从数据中进行学习了。

最常见的机器学习就是学习Y=f(X)的映射，针对新的X预测Y。这叫做预测建模或预测分析。我们的目标就是让预测更加精确。

针对希望对机器学习有个基本了解的新人来说，下面将介绍数据科学家们最常使用的10种机器学习算法。

#### 1.线性回归

线性回归可能是统计和机器学习领域最广为人知的算法之一。

以牺牲可解释性为代价，预测建模的首要目标是减小模型误差或将预测精度做到最佳。我们从统计等不同领域借鉴了多种算法，来达到这个目标。

线性回归通过找到一组特定的权值，称为系数B。通过最能符合输入变量x到输出变量y关系的等式所代表的线表达出来。

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_0.png)

例如：y=B0+B1 * x。我们针对给出的输入x来预测y。线性回归学习算法的目标是找到B0和B1的值。

不同的技巧可以用于线性回归模型。比如线性代数的普通最小二乘法，以及梯度下降优化算法。线性回归已经有超过200年的历史，已经被广泛的研究。根据经验，这种算法可以很好的消除相似的数据，以及去除数据中的噪声。它是快速且简便的首选算法。

#### 2. 逻辑回归

逻辑回归是另一种从统计领域借鉴而来的机器学习算法。

与线性回归相同。它的目的是找出每个输入变量的对应参数值。不同的是，预测输出所用的变换是一个被称作logistic函数的非线性函数。

logisitc函数像一个大S。它将所有值转换为0到1之间的数。这很有用，我们可以根据一些规则将logistic函数的输出转换为0或1(比如当小于0.5时则为1)，然后一次进行分类。

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_1.png)

正是因为模型学习的这种方式，逻辑回归做出的预测可以被当作输入为0和1两个分类数据得概率值。这在一些需要给出预测合理性的问题中非常有用。

就像线性回归，在需要移除与输出变量无关的特征以及相似特征方面，逻辑回归可以表现得很好。在处理二分类问题上，它时一个快速高效的模型。

#### 3. 线性判别分析

逻辑回归是一个二分类问题的传统分类算法。如果需要进行更多的分类，线性判别分析算法(LDA)是一个更好的线性分类方法。

对LDA的解释非常直接。它包括针对每一个类的输入数据得统计特性。对于单一输入变量来说包括：

1. 类内样本均值

1. 总体样本变量

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_2.png)

通过计算每个类的判别值，并根据最大值来进行预测。这种方法假设数据服从高斯分布(钟形曲线)。所以它可以较好的提前去除离群值。它是针对分类模型预测问题的一种简单有效的方法。

#### 4. 分类与回归树分析

决策树是机器学习预测建模的一类重要算法。

可以用二叉树来解释决策树模型。这是根据算法和数据结构建立的二叉树，这并不难理解。每个节点代表一个输入变量以及变量的分叉点(假设是数值变量)

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_3.png)

树的叶节点包括用于预测的输出变量y。通过树的各分支到达叶节点，并输出对应叶节点的分类值。

树可以进行快速的学习和预测。通常并不需要对数据做特殊的处理，就可以使用这个方法对多种问题得到准确的结果。

#### 5. 朴素贝叶斯

朴素贝叶斯是一个简单，但是异常强大的预测建模算法。

这个模型包括两种概率。它们可以通过训练数据直接计算得到：

> 每个类的概率；给定x值情况下每个类的条件概率。


根据贝叶斯定理，一旦完成计算，就可以使用概率模型针对新的数据进行预测。当你的数据为实数时，通常假设服从高斯分布(钟形曲线)。这样你可以很容易的预测这些概率。


![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_4.png)


之所以称作朴素贝叶斯，是因为我们假设每个输入变量都是独立的。这是一个强假设，在真实数据中几乎时不可能的。但对于很多复杂问题，这种方法非常有效。


#### 6. K最近邻算法

K最近邻算法(KNN)是一个非常简单有效的算法。KNN的模型表示就是整个训练数据集。

对于新数据点的预测则是，寻找整个训练集中K各最相似的样本(邻居)，并把这些样本的输出变量进行总结。对于回归问题可能意味着平均输出变量。对于分类问题则可能意味着类值的众数(最常出现的那个值)。

诀窍是如何在数据样本中找出相似性。最简单的方法就是，如果你得特征都是以相同的尺度(比如说都是英寸)度量的，你就可以直接计算它们互相之间的欧氏距离。

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_5.png)

KNN需要大量空间来存储所有的数据。但只是需要进行预测的时候才开始计算(学习)。你可以随时更新并组织训练样本以保证预测的准确性。

在维数很高(很多输入变量)的情况下，这种通过距离或相近成都进行判断的方法可能失败。这会对算法的性能产生负面的影响。这被称作维度灾难。我建议你只有当输入变量与输出预测变量最具有关联性的时候使用这种算法。

#### 7. 学习矢量量化

K最近邻算法的缺点是你需要存储所有训练数据集。而学习矢量量化(LVQ)是一个人工神经网络算法。它允许你选择需要保留的训练样本个数，并且学习这些样本看起来应该具有何种模式。

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_6.png)

LVQ可以表示为一组码本向量的集合。在开始的时候进行随机选择。通过多轮学习算法的迭代，最后得到与训练数据集最相配的结果。通过学习，码本向量可以像K最近邻算法那样进行预测。通过计算新数据样本与码本向量之间的距离找到最相似的邻居(最符合码本向量)。将最佳的分类值(或回归问题中的实数值)返回作为预测值。如果你将数据调整到相同的尺度，比如0和1，则可以得到最好的结果。

如果你发现对于你的数据集，KNN有较好的效果，可以尝试一下LVQ来减少存储整个数据集对存储空间的依赖。

#### 8. 支持向量机

支持向量机(SVM)可能是最常用并且最常被谈到的机器学习算法。

超平面是一条划分输入变量空间的线。在SVM中，选择一个超平面，它能最好的将输入变量空间划分为不同的类，要么是0，要么是1。在2维情况下，可以将它看作一根线，并假设所有输入点都被这根线完全分开。SVM通过学习算法，找到最能完成类划分的超平面的一组参数。

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_7.png)

超平面和最近邻的数据点的距离看作一个差值。最好的超平面可以把所有数据划分为两个类，并且这个差值最大。只有这些点与超平面的定义和分类器的构造有关。这些点被称作支持向量。是它们定义了超平面。在实际使用中，优化算法被用于找到一组参数值使差值达到最大。

SVM可能使一种最为强大的分类器，它值得你一试。

#### 9. Bagging和随机森林

随机森林是一个常用并且最为强大的机器学习算法。它是一种集成机器学习算法，称作自举汇聚或bagging。

bootstrap是一种强大的统计方法，用于数据样本的估算。比如均值。你从数据中采集很多样本，计算均值，然后将所有均值再求平均。最终得到一个真实均值的较好的估计值。

在bagging中用了相似的方法。但是通常用决策树来代替对整个统计模型的估计。从训练集中采集多个样本，针对每个样本构造模型。当你需要对新的数据进行预测，每个模型做一次预测，然后把预测值做平均得到真实输出的较好的预测值。

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_8.png)

这里的不同在于在什么地方创建树，与决策树选择最优分叉点不同，随机森林通过加入随机性从而产生次优的分叉点。

每个数据样本所创建的模型与其他的都不相同。但在唯一性和不同性方面依然准确。结合这些预测结果可以更好的得到真实的输出估计值。

如果在高方差的算法(比如决策树)中得到较好的结果，你通常也可以通过袋装这种算法得到更好的结果。

#### 10. Boosting和AdaBoost

Boosting是一种集成方法，通过多种弱分类器创建一种强分类器。它首先通过训练数据建立一个模型，然后再建立第二个模型来修正前一个模型的误差。再完成对训练集完美预测之前，模型和模型的最大数量都会不断添加。

AdaBoost是第一种成功的针对二分类的boosting算法。它是理解boosting的最好的起点。现代的boosting方法是建立再AdaBoost之上。多数都是随机梯度boosting机器。

![](../../assets/images/ML/attachments/[ML入门]算法篇(1)_image_9.png)

AdaBoost与短决策树一起使用。当第一颗树创建之后，每个训练样本的树的性能将用于决定，针对这个训练样本下一颗树将给予多少关注。难于预测的训练数据给予较大的权值，反之容易预测的样本给予较小的权值。模型按顺序被建立，每个训练样本权值的更新都会影响下一颗树的学习效果。完成决策树的建立之后，进行对新数据的预测，训练数据的精确性决定了每棵树的性能。

因为重点关注修正算法的错误，所以移除数据中的离群值非常重要。

#### 总结

当面对各种机器学习算法，一个新手最常问的问题是"我该使用哪个算法"。要回答这个问题需要考虑很多因素：

> 数据的大小，质量和类型；完成计算所需要的时间；任务的紧迫程度；你需要对数据做什么处理。


在尝试不同算法之前，就算一个经验丰富的数据科学家也不可能告诉你哪种算法性能最好。虽然还有很多其他的机器学习算法，但这里列举的是最常用的几种。如果你是一个机器学习的新手，这几种是最好的学习起点。