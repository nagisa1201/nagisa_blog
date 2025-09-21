---
title: Transformer及其变体
categories: 
  - 技术
  - 深度学习
  - 论文学习
tags: [深度学习,大模型,论文学习]
date: 2025-09-21
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  Transformer及其变体学习笔记
</div>

# 为什么需要Transformer变形金刚
## RNN的好与坏
![](/blog-img/Transformer_attachment/62583c38b0047e14114f2a2fa16a3200.png)
- ***RNN的计算方式***[矩阵方式]
![](/blog-img/Transformer_attachment/b352765450e9e0dfc7b589e433c2acdd.png)
矩阵行列不变的计算，将t-1的部分处理后并入t时刻的传入矩阵、
- ***计算公式***
![](/blog-img/Transformer_attachment/078f964d833e63b0d098ffab64e58e26.png)
### RNN缺点
- 无法捕捉长期依赖
- 无法并行，必须一步一步顺序计算

# Transformer模型（基础数学理解）
## 模型的编码、计算方式核心
 - 将词向量用Wq、Wk、Wv相乘的到三个变换词向量![](/blog-img/Transformer_attachment/ab79e57f001bfa392565d8abd181605e.png)
 - *关系性判断*：变换词向量之间的相似度![](/blog-img/Transformer_attachment/8e333ea461f4c0ad50fc47813535069d.png)![](/blog-img/Transformer_attachment/99a36557d4621b892295e8fe64f9c323.png)![](/blog-img/Transformer_attachment/a55ab935cac3c3c998e61584c9080c96.png)
### ***Attention注意力机制，Multi-Head Attention多头注意力机制***![](/blog-img/Transformer_attachment/01f91330a58b5d0dd813951488f89424.png)
   - Dot单头注意力：Q、K矩阵相乘后经过√dk缩放，经过Softmax处理后与V矩阵相乘
   - 多头：将QKV矩阵经过多个权重矩阵，拆分到多个头中分别经过注意力机制运算（单头运算），之后合并经过一次矩阵运算
### 训练流程![](/blog-img/Transformer_attachment/4232c6bfa122b3406c3bcb4d617e89d5.png)
#### 编码器段
- 输入词句，词嵌入引入位置编码
- 经过多头注意力，残差和归一化处理
- 送入一个全连接神经网络，再进行残差和归一化处理，把结果送入多头注意力机制的两个输入中作为KV矩阵 
#### 解码器段
- 输入翻译文本，词嵌入引入位置编码
- 多头注意力，残差和归一化处理送入多头注意力机制Q矩阵 ***（注意）：Masked代表存在掩码，会遮盖当前翻译字/词的后面词句，模拟真实推理时的逐字翻译***
- 之后残差归一化后进入全连接神经网络，再次残差归一化，后进入一层线性变化神经网络投射大表格中，再进入softmax层计算概率
# Transformer所需知道的名词（更深层的理解Transformer）
#词嵌入与位置编码 #自注意力机制 #softmax_function
## 编码器部分Encoder
### 词嵌入与位置编码：将单词编码为数字、编码单词的位置
- 将一句话中的每个单词经过词嵌入（反向传播确定权重参数），将单词变化为向量
- 位置编码则是利用正余弦函数得到位置向量，***可以保持对整句话中词序的追踪***
### 自注意力机制：编码单词之间的关系
***解决词与词关系的机制***![](/blog-img/Transformer_attachment/2c2b4b50b7d2c16ca4ac226dfb8a07b2.png)
> *Q,V,K为三种不同的参数矩阵，是允许复用的*，称为***注意力单元***，其中Q,K矩阵（query、key）计算相似权重，进入softmax层算出相似概率，V矩阵承载信息数据，与概率加权后得到最终结果，注意力权重确定后，V 被加权聚合，生成上下文感知的新表示。
### 残差连接：使该模型可以相对容易和快速的训练，每一个注意力单元都需要添加
***解决自注意力流程中需兼顾词嵌入和位置编码信息不丢失的问题***
![](/blog-img/Transformer_attachment/187ac73485d9c6cdf20736dd253d01f7.png)
>旁路残差连接，注意力机制接管处理词与词之间的关系时无需保留词嵌入和位置编码信息，而是通过处理过后的残差介入来加入这一关系。
### 总结
**实际上，对于每个单词而言，只是不停地复制更多如上相同的单元进行编码。通过每个单词的单元化处理，Transformer可以同时编码计算，并行处理，同时计算每一步而非顺序单独计算。**
## 解码器Decoder
### 编码器解码器注意力层：解决不漏掉编码器输入句子的重要词句的问题
>Q:为什么编码器解码器注意力层（交叉注意力层）的K、V输出来源于解码器输出后再次进行Key和Value的计算？
>A:​​Key专注于相似度计算，Value专注于信息传递。Key当决定某个单词应该被翻译为什么时，决定每个输入单词应当使用的百分比（Key的结果经过了Softmax层转化为概率）。Value决定内容（与概率相乘后）。***这样的交叉注意力层是为了翻译时不要漏掉编码器输入句子的重要词句***![](/blog-img/Transformer_attachment/1a04ac12c3fb720bf0c9ebf6a7b0c10b.png)

>***Q:交叉注意力层中Query的来源？***
>***A:交叉注意力层的Query直接继承自解码器前一层的输出向量，经过一个可学习的Wq矩阵得到。***

>**Q:交叉注意力层后还存在一个Linear层和Softmax层的意义？**
>![](/blog-img/Transformer_attachment/a957f71ea70e6a9ee83efca42565f2a9.png)
>**A:Linear层主要是将注意力单元输出的高维语义向量映射到词汇空间，解决“抽象特征无法直接对应词语”的问题；而Softmax层主要是通过概率的方法解决训练时损失函数返回和后续推理输出最大概率项。**
### 基础模型总述
![](/blog-img/Transformer_attachment/9f6ea6dc98e1d4fd5b2279a683df9c6c.png)

# VIT(Vision Transformer):证明了Transformer在计算机视觉的可用性与其通用性
![](/blog-img/Transformer_attachment/65df98cd3741867b6a124a71a80f4ced.png)
## 主要改变Embedding层：图像转化为Token序列
- 将图像转化为词句，把patch作为一个语义单元，**对应文本里的一个Token。因为一个像素点只存在RGB三个信息，用高维向量来刻画太浪费了。**
>**Q:一个图像如何转化成一个embedding向量呢？**
>A:***①patch embedding：将原始图像划分为多个同维patch，每个相当于句子中的一个单词***
>***②经过一个全连接层，将patch序列压缩成一个向量***
>***③position embedding：即加入tokens的位置信息为后面self attention做准备***
>***④在向量开头加上class token目的是便于后期做分类***
>   ![](/blog-img/Transformer_attachment/07bbbd51fd66e36993327623e9498050.png)
>   **需要说明的是E表示patch embedding，Xclass是class token拼接，Epos是加上position embedding**
>   之后作为输入喂给Transformer网络
## VIT总流程
![](/blog-img/Transformer_attachment/4f82996a4c2e553666a2bd0ebdc58305.png)
- MLP Head是分类头
- Transformer Encoder与Transformer中的原生编码器极其相似
# Cross-Attention：交叉注意力
***[原论文出处](https://arxiv.org/pdf/2106.05786)***
>在Transformer中，CrossAttention实际上是指编码器和解码器之间的交叉注意力层。
>**Cross-Attention的输入来自于两个不同序列，一个作为查询Query，另一个作为Key和Value。这使得CrossAttention可以处理跨域任务（如多模态结合，跨模态交互）**
## Cross-Attention算法
![](/blog-img/Transformer_attachment/06bd20ff64afe7f502d62bdd827816a2.png)
# Swin Transformer：使用滑动窗口的分层VIT
***[原论文出处](https://arxiv.org/pdf/2103.14030)***
![](/blog-img/Transformer_attachment/e822279b113800e2b8ad203e9fe78586.png)
## 解名：Hierarchical Vision Transformer using Shifted Windows
- ***Vision Transformer表明是对VIT的改进，解决了VIT在Embedding步时直接拆分图片为patch时对图片信息裁剪的丢失问题***
- **Hierarchical分层在于（如图Architecture所示）类似于CNN中的深度卷积一样，不断的倍缩长宽，加深通道，类似于提高感受野，获得了图像信息的更多维信息，提高了信息的利用，保留了更多特征。**
- **Shifted Windows的原因Swin Transformer将图片划分为多个H,W更小的小单元进行处理，但每个小单元单独为注意力单元，互相之间没有通信和关联，这样必然丢失关联性信息，滑动窗口是用来解决这一问题的。**![](/blog-img/Transformer_attachment/5c4358c18f9451514883cf5f65c4b0bb.png)
- ***分成小窗口不仅仅大幅减少了计算的复杂度也使得信息损失变小***
## Shifted Windows的思想，本质
![](/blog-img/Transformer_attachment/ecd623cc962a6552ea3cf52af8955553.png)
### 滑动、补偿、掩膜机制
- 用NxN的窗口（此处以4x4为例，原论文是以7x7为多）进行滑动计算注意力![](/blog-img/Transformer_attachment/0f57390fba16f80fa76b08843838f247.png)
- 滑动后①是可以直接计算的，但其中包含了前一次滑动窗口未向右向下滑动的不同区块信息，**相当于进行了不同图片块的信息融合**
- 其中还有②④⑥并不完整，将A、B、C补偿在37985所在的地方
- ***但是补充完后存在与其相邻块无关联的问题（因为是仅仅强行拼接），所以在计算移动后滑动窗口的注意力时，部分交互注意力需要用掩膜的方法舍弃掉，不需要这些无效数据，将不要的部分Mask掉***![](/blog-img/Transformer_attachment/3b27e4ae931fb68942228b9d5eefdcf1.png)
							④和⑤的部分的注意力计算图示
![](/blog-img/Transformer_attachment/b1da1b1fa09c7cf37859081d8d4f6434.png)
							②和③部分的注意力计算图示
![](/blog-img/Transformer_attachment/d1c03c1bbf10244a31c12a2220fe1878.png)
						**⑥⑦⑧⑨处注意力计算图示（Window3）**
## 模型与代码解释
# 了解神经网络需要知道的名词
#反向传播 #卷积层
## CNN：卷积神经网络![](/blog-img/Transformer_attachment/a31ec328fe7b280fccb62709e6e29ded.png)
### CNN出现要解决的问题——全连接网络的弊端![](/blog-img/Transformer_attachment/139385cced3e3a56dc94a3bc9c9a0c4b.png)
- **参数爆多、训练数据量大，且极容易过拟合
- **每个节点都与下一层的所有节点相连，极其冗余**
### 重点：卷积核，卷积核的本质是一种过滤器，可以是任何情况![](/blog-img/Transformer_attachment/aaf480ddf3601d05b15554053a8996c8.png)
通过卷积核遍历提取的方法，***在保证提取同一张图片的卷积核不变情况下，把一张图片的特征提取到Feature Map中，且将所需学习的参数大幅度的减小（即卷积核内的所有参数）***
- 卷积核在遍历整个图像时，卷积核的区间叫做**感受野**，**感受野越大，越能提取更大尺寸的特征。并且可以有多层叠加，卷积层数越深，对复杂特征表达能力越强**![](/blog-img/Transformer_attachment/db5c763f130dc8a76bcdb6ee804a1bf6.png)
### 非线性激活函数：处理非线性问题，ReLU函数——将负数变为0![](/blog-img/Transformer_attachment/abfb65019c9b0aceb90d46629fdcb335.png)
- 卷积核提取的特征图（矩阵）中，将负数变为0
### 池化层（Pooling）：抓住主要矛盾，剔除次要矛盾![](/blog-img/Transformer_attachment/e5f9df56d1a2fbb51aa29b452414e5b7.png)
- Max Pooling：目的是提取最显著特征。
- Average Pooling：目的是降采样等。
## 反向传播Back Propagation
### 神经网络的本质
- **输入数据经过线性加权组合**
- **激活函数实现非线性变换**
- **多层加权迭代得到最后输出**
***神经网络初始一般都为随机参数，学习的目的是为了赋予其优化的更好参数***![](/blog-img/Transformer_attachment/1fed6d51eef53d2c4cd339ca6ab91593.png)
- 损失函数的最小化是对最优输出的追求，即本次输出的反馈。
- 梯度下降中，参数求偏导的意义是找最坏参数，η是学习率参数，-号是为了减小误差，w+的迭代是为了寻找最优关系。
- ***上述的所有参数、函数的选择，在非线性系统的过程变化中都会对结果可能存在巨大的影响。选取科学的参数、函数对这个系统输出值的合理化至关重要。***
