---
layout: default
title: ML基础-概率论(1)-三门问题
parent: ML
---

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_0.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_1.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_2.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_3.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_4.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_5.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_6.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_7.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_8.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_9.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_10.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_11.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_12.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_13.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_14.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_15.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_16.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_17.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_18.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_19.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_20.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_21.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_22.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_23.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_24.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_25.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_26.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_27.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_28.png)

从上图可以看出，我们总共面临着6种不同的子局面。这些子局面的获奖几率各是多少呢？其实不难得出结论：

1. 选到有奖品的门显然，这时候如果不换门，获奖几率是100%；如果换门，获奖几率是0%。

1. 选到空门A这时候，空门B已经被打开，所以换门的获奖几率是100%，不换门的获奖几率是0%。3. 选到空门B和情况2同理，空门A已经被打开，所以换门的获奖几率是100%，不换门的获奖几率是0%。接下来，让我们把上述的各种概率总结到图中：

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_29.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_30.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_31.png)

不换门的获奖率 = （1/3 X 100%）+（1/3 X 0%）+（1/3 X 0%）=1/3

换门的获奖率 = （1/3 X 0%）+（1/3 X 100%）+（1/3 X 100%）=2/3

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_32.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_33.png)

![](../../assets/images/ML/attachments/[ML基础]概率论(1)-三门问题_image_34.png)