> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/428448728)

讲 Relu 函数前需要先了解关于激活函数的概念和作用。

### 一、什么是激活函数?

首先了解一下神经网络的基本模型

![](https://pic3.zhimg.com/v2-ab0e1432a74a2335e082fb676e63f27a_r.jpg)

如上图所示，神经网络中的每个神经元节点接受上一层神经元的输出值作为本神经元的输入值，并将输入值传递给下一层，输入层神经元节点会将输入属性值直接传递给下一层（隐层或输出层）。在多层神经网络中，上层节点的输出和下层节点的输入之间具有一个函数关系，这个函数称为激活函数。  
**简单来说，激活函数，并不是去激活什么，而是指如何把 “激活的神经元的特征” 通过函数把特征保留并映射出来，即负责将神经元的输入映射到输出端。**  

**二、为何引入非线性的激活函数？**

如果不用激活函数，在这种情况下每一层输出都是上层输入的线性函数。容易验证，无论神经网络有多少层，输出都是输入的线性组合，与没有隐藏层效果相当，这种情况就是最原始的感知机（Perceptron）了。因此引入非线性函数作为激活函数，这样深层神经网络就有意义了（不再是输入的线性组合，可以逼近任意函数）。最早的想法是 sigmoid 函数或者 tanh 函数，输出有界，很容易充当下一层输入。

**三、引入 ReLu 的原因**

第一，采用 sigmoid 等函数，算激活函数时（指数运算），**计算量大**，反向传播求误差梯度时，求导涉及除法，计算量相对大，而采用 Relu 激活函数，整个过程的计算量节省很多。

第二，对于深层网络，sigmoid 函数反向传播时，很容易就会出现 **梯度消失** 的情况（在 sigmoid 接近饱和区时，变换太缓慢，导数趋于 0，这种情况会造成信息丢失），从而无法完成深层网络的训练。

第三，ReLu 会使一部分[神经元](https://link.zhihu.com/?target=https%3A//www.baidu.com/s%3Fwd%3D%25E7%25A5%259E%25E7%25BB%258F%25E5%2585%2583%26tn%3D24004469_oem_dg%26rsv_dl%3Dgh_pl_sl_csd)的输出为 0，这样就造成了 **网络的稀疏性**，并且减少了参数的相互依存关系，**缓解了过拟合**问题的发生。

_当然现在也有一些对 relu 的改进，比如 prelu，random relu 等，在不同的数据集上会有一些训练速度上或者准确率上的改进，具体的大家可以找相关的 paper 看。_

_多加一句，现在主流的做法，会在做完 relu 之后，加一步 batch normalization，尽可能保证每一层网络的输入具有相同的分布 [1]。而最新的 paper[2]，他们在加入 bypass connection 之后，发现改变 batch normalization 的位置会有更好的效果。大家有兴趣可以看下。_

**四、Relu 非处处可导（在 0 不可导），为什么作为激活函数？**

relu 函数是常见的激活函数中的一种，表达形式如下：

![](https://pic3.zhimg.com/v2-6a574652bbf12899a2530ba87973870a_b.png)

从表达式可以明显地看出：**Relu 其实就是个取最大值的函数。**

**relu、sigmoid、tanh 函数曲线**

![](https://pic1.zhimg.com/v2-6d9cc2a14532415e92071310eaeabecc_r.jpg)

**sigmoid 的导数**

![](https://pic3.zhimg.com/v2-e7a2572521992be228b69f33ef5c2e1e_r.jpg)

**relu 的导数**

![](https://pic2.zhimg.com/v2-9fb1381d9a4e4bc838a296de6b4bca09_r.jpg)

结论：

第一，sigmoid 的导数只有在 0 附近的时候有比较好的激活性，在正负饱和区的梯度都接近于 0，所以这会造成梯度弥散，而 relu 函数在大于 0 的部分梯度为常数，所以不会产生梯度弥散现象。

第二，relu 函数在负半区的导数为 0 ，所以一旦神经元激活值进入负半区，那么梯度就会为 0，而正值不变，这种操作被成为单侧抑制。（也就是说：**在输入是负值的情况下，它会输出 0，那么神经元就不会被激活。这意味着同一时间只有部分神经元会被激活，从而使得网络很稀疏，进而对计算来说是非常有效率的。**）正因为有了这单侧抑制，才使得神经网络中的神经元也具有了稀疏激活性。尤其体现在深度神经网络模型 (如 CNN) 中，当模型增加 N 层之后，理论上 ReLU 神经元的激活率将降低 2 的 N 次方倍。

![](https://pic4.zhimg.com/v2-0f7370d4bc97810fe9904f94a4bf0613_r.jpg)

第三，relu 函数的导数计算更快，程序实现就是一个 if-else 语句，而 sigmoid 函数要进行浮点四则运算。

综上，relu 是一个非常优秀的激活函数。

**五、Relu 函数的优势**

1、没有饱和区，不存在梯度消失问题，防止梯度弥散；

2、稀疏性；

3、没有复杂的指数运算，计算简单、效率提高；

4、实际收敛速度较快，比 Sigmoid/tanh 快很多；

5、比 Sigmoid 更符合生物学神经激活机制。

**六、relu 也存在不足**

就是训练的时候很” 脆弱”，很容易就”die” 了。举个例子：一个非常大的梯度流过一个 ReLU 神经元，更新过参数之后，这个神经元再也不会对任何数据有激活现象了。如果这个情况发生了，那么这个神经元的梯度就永远都会是 0。实际操作中，如果你的 learning rate 很大，那么很有可能你网络中的 40% 的神经元都”dead” 了。 当然，如果你设置了一个合适的较小的 learning rate，这个问题发生的情况其实也不会太频繁。  

**七、为什么通常 Relu 比 sigmoid 和 tanh 强，有什么不同？**

主要是因为它们 gradient 特性不同。sigmoid 和 tanh 的 gradient 在饱和区域非常平缓，接近于 0，很容易造成 vanishing gradient 的问题，减缓收敛速度。vanishing gradient 在网络层数多的时候尤其明显，是加深网络结构的主要障碍之一。相反，Relu 的 gradient 大多数情况下是常数，有助于解决深层网络的收敛问题。Relu 的另一个优势是在生物上的合理性，它是单边的，相比 sigmoid 和 tanh，更符合生物神经元的特征。

而提出 sigmoid 和 tanh，主要是因为它们全程可导。还有表达区间问题，sigmoid 和 tanh 区间是 0 到 1，或着 - 1 到 1，在表达上，尤其是输出层的表达上有优势。

ReLU 更容易学习优化。因为其分段线性性质，导致其前传，后传，求导都是分段线性。而传统的 sigmoid 函数，由于两端饱和，在传播过程中容易丢弃信息：

参考：

[ReLU 为什么比 Sigmoid 效果好 - alexanderkun - 博客园](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/alexanderkun/p/6918029.html)[为什么引入 ReLU 激活函数_Du_Shuang 的博客 - CSDN 博客](https://link.zhihu.com/?target=https%3A//blog.csdn.net/Du_Shuang/article/details/87868357%3Futm_medium%3Ddistribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-1.no_search_link%26spm%3D1001.2101.3001.4242.2)[Relu 浅谈_jiangjunlanzhoulan 的博客 - CSDN 博客_relu 导数](https://link.zhihu.com/?target=https%3A//blog.csdn.net/jiangjunlanzhoulan/article/details/79690137)[浅析激活函数之 Relu 函数](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/338afb1389c9)