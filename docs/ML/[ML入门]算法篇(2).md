---
layout: default
title: [ML入门]算法篇(2)
parent: ML
---

### 算法篇

#### 1. 决策树

根据一些feature进行分类，每个节点提一个问题，通过判断，将数据分为两类，再继续提问。这些问题是根据已有数据学习出来的，再投入新数据的时候，就可以根据这棵树上的问题，将数据划分到合适的叶子上。

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_0.png)

#### 2. 随机森林

在源数据中随机选取数据，组成几个子集

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_1.png)

S矩阵是源数据，有1-N条数据，A B C是feature，最后一列C是类别

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_2.png)

由S随机生成M各子矩阵

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_3.png)

这M个子集得到M个决策树

将新数据投入到这M个树中，得到M个分类结果，计数看预测成哪一类的数目最多，就将此类别作为最后的预测结果

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_4.png)

#### 3. 逻辑回归

当预测目标是概率这样的，值域需要满足大于等于0，小于等于1的，这个时候单纯的线性模型是做不到的，因为在定义域不在某个范围之内时，值域也超出了规定区间。

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_5.png)

所以此时需要这样的形状的模型会比较好

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_6.png)

那么怎么得到这样的模型呢?

这个模型需要满足两个条件，大于等于0，小于等于1

大于等于0的模型可以选择绝对值，平均值，这里用指数函数，一定大于0

小于等于1用除法，分子是自己，分母是自身加上1，那一定是小于1的了

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_7.png)

再做一下变形，就得到了logistic regression模型

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_8.png)

通过元数据计算可以得到相应的系数了

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_9.png)

最后得到logistic的图形

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_10.png)

#### 4. SVM

要将两类分开，想要得到一个超平面，最优的超平面是到两类的margin达到最大，margin就是超平面与离它最近一点的距离，如下图，Z2>Z1，所以绿色的超平面比较好

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_11.png)

将这个超平面表示成一个线性方程，在线上方的一类，都大于等于1，另一类小于等于-1

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_12.png)

点到面的距离根据图中的公式计算

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_13.png)

所以得到total margin的表达式如下，目标是最大化这个margin，就需要最小化分母，于是变成了一个优化问题

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_14.png)

举个例子，三个点，找到最优的超平面，定义了weight vector = (2,3) - (1,1)

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_15.png)

得到weight vector为(a,2a)，将两个点带入方程，带入(2,3)另其值=1，带入(1,1另其值=-1，求解出a和截距w0的值，进而)得到超平面的表达式

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_16.png)

a求出来后，带入(a,2a)得到的就是support vector

a和w0带入超平面的方程就是support vector machine

#### 5. 朴素贝叶斯

举个例子，在NLP的应用中，给一段文字，返回情感分类，这段文字的态度是positive，还是negative

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_17.png)

为了截崛这个问题，可以只看其中的一些单词

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_18.png)

这段文字，将仅由一些单词和它们的计数代表

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_19.png)

原始问题是：给你一句话，它属于哪一类

通过bayes rules变成一个比较简单容易求得的问题

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_20.png)

问题变成，这一类中这句话出现的概率是多少，当然，别忘了公式里的另外两个概率

例子：单词love在positive的情况下出现的概率是0.1，在negative的情况下出现的概率是0.001

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_21.png)

#### 6. K最邻近

给一个新的数据时，离它最近的k个点中，哪个类别多，这个数据就属于哪一类

例子：要区分猫和狗，通过claws和sound两个feature来判断的话，圆形和三角形是已知分类的了，那么这个star代表的哪一类呢

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_22.png)

k=3时，这三条线链接的点就是最近的三个点，那么圆形多一些，所以这个star就是属于猫

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_23.png)

#### 7. K均值

想要将一组数据，分为三类，粉色数值大，黄色数值小

最开始先初始化，这里面选了最简单的3，2，1作为各类的初始值

剩下的数据里，每个都与三个初始值计算距离，然后规类到离它最近的初始值所在类别

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_24.png)

分好类后，计算每一类的平均值，作为新一轮的中心点

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_25.png)

几轮之后，分组不再变化了，就可以停止了

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_26.png)

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_27.png)

#### 8. AdaBoost

adaboost时boosting的方法之一

boosting就是把若干个分类效果并不好的分类器综合起来考虑，会得到一个效果比较好的分类器。

下图，左右连个决策树，单个看是效果不怎么好的，但是把同样的数据投入进去，把两个结果加起来考虑，就会增加可信度

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_28.png)

adaboost的例子，手写识别中，在画板上可以抓取到很多features，例如始点的方向，始点和终点的距离等等

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_29.png)

training的时候，会得到每个feature的weight，例如2和3的开头部分很像，这个feature对分类起到的作用很小，它的权重也就会较小

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_30.png)

而这个alpha角就具有很强的识别性，这个feature的权重就会较大，最后的预测结果是综合考虑这些feature的结果

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_31.png)

#### 9. 神经网络

Neural Networks适合一个input可能落入至少两个类别里

NN由若干层神经元，和它们之间的联系组成

第一层是input层，最后一层是output层

在hidden层和output层都有自己的classifier

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_32.png)

input输入到网络中，被激活，计算的分数被传递到下一层，激活后面的神经层，最后output层的节点上的分数代表属于各类的分数，下图例子得到分类结果为class 1

同样的input被传输到不同的节点上，之所以会得到不同的结果是因为各自节点有不同的weights和bias

这也就是forward propagation

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_33.png)

#### 10. 马尔可夫

例子，根据这一句话"the quick brown fox jumps over the lazy dog"，要得到markov chain

步骤，先给每一个单词设定成一个状态，然后计算状态间转换的概率

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_34.png)

这是一句计算出来的概率，当你用大量文本去做统计的时候，会得到更大的状态转移矩阵，例如the后面可以连接的单词，及相应的概率

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_35.png)

生活中，键盘输入法的备选结果也是一样的原理，模型会更高级

![](../../assets/images/ML/attachments/[ML入门]算法篇(2)_image_36.png)