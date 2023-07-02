> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/90771255)

原问题：

[softmax 和 cross-entropy 是什么关系？](https://www.zhihu.com/question/294679135/answer/885285177)

softmax 虽然简单，但是其实这里面有非常的多细节值得一说。

我们挨个捋一捋。

1. 什么是 Softmax？
---------------

首先，softmax 的作用是把 一个序列，变成概率。

![](https://pic3.zhimg.com/v2-4366f8c16f1b13e7b206b0835c76a532_b.jpg)![](https://pic3.zhimg.com/v2-d45280059cf96b650bcc02c5c43607ba_b.jpg)

他能够保证：

1.  所有的值都是 [0, 1] 之间的（因为概率必须是 [0, 1]）
2.  所有的值加起来等于 1

从概率的角度解释 softmax 的话，就是

![](https://pic1.zhimg.com/v2-31ffb2a55a3cc70ca705f6302880fa64_b.jpg)

2. 文档里面跟 Softmax 有关的坑
---------------------

这里穿插一个 “小坑”，很多 deep learning frameworks 的文章里面 （PyTorch，TensorFlow）是这样描述 softmax 的，

> take logits and produce probabilities

很明显，这里面的 `logits` 就是 全连接层（经过或者不经过 activation 都可以）的输出，`probability`就是 softmax 的输出结果。 这里 `logits` 有些地方还称之为 `unscaled log probabilities`。这个就很意思了，unscaled probability 可以理解，那又为什么 全连接层直接出来结果会和 log 有关系呢？

![](https://pic1.zhimg.com/v2-860a23011be16e17b290ca742ba7a044_r.jpg)

原因有两个：

1.  因为 全连接层 出来的结果，其实是无界的（有正有负），这个跟概率的定义不一致，但是你如果他看成 概率的 log，就可以理解了。
2.  softmax 的作用，我们都知道是 normalize probability。在 softmax 里面，输入 $a_i$a_i 都是在指数上的 $e^{a_i}$e^{a_i} ，所有把 $a_i$a_i 想成 log of probability 也就顺理成章了。

3. Softmax 就是 Soft 版本的 Max
--------------------------

好的，我们把话题拉回到 softmax。

softmax，顾名思义就是 soft 版本的 max。我们来看一下为什么？

举个栗子，假如 softmax 的输入是：

$[1.0, 2.0, 3.0]$[1.0, 2.0, 3.0]

softmax 的结果是：

$[0.09, 0.24, 0.67]$[0.09, 0.24, 0.67]

我们稍微改变一下输入，把 3 改大一点，变成 5，输入是

$[1.0, 2.0, 5.0]$[1.0, 2.0, 5.0]

softmax 的结果是：

$[0.02, 0.05, 0.93]$[0.02, 0.05, 0.93]

可见 softmax 是一种非常明显的 “马太效应”：强（大）的更强（大），弱（小）的更弱（小）。假如你要选一个最大的数出来，这个其实就是叫 hardmax。那么 softmax 呢，其实真的就是 soft 版本的 max。

这种 soft 版本的 max 在很多地方有用的上。因为 hard 版本的 max 好是好，但是有很严重的梯度问题，求最大值这个函数本身的梯度是非常非常稀疏的（比如神经网络中的 max pooling），经过 hardmax 之后，只有被选中的那个变量上面才有梯度，其他都是没有梯度。这对于一些任务（比如文本生成等）来说几乎是不可接受的。所以要么用 hard max 的变种，比如 Gumbel，

[Categorical Reparameterization with Gumbel-Softmax](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1611.01144)

亦或是 ARSM

[ARSM: Augment-REINFORCE-Swap-Merge Estimator for Gradient Backpropagation Through Categorical Variables](https://link.zhihu.com/?target=http%3A//proceedings.mlr.press/v97/yin19c.html)

，要么就直接 softmax。

4. Softmax 的实现以及数值稳定性
---------------------

softmax 的代码实现看似是比较简单的，直接套上面的公式就好

```
def softmax(x):
    """Compute the softmax of vector x."""
    exps = np.exp(x)
    return exps / np.sum(exps)

```

但是这种方法非常的不稳定。因为这种方法要算指数，只要你的输入稍微大一点，比如：

$[1000, 2000, 3000]$[1000, 2000, 3000]

分母上就是

$e^{1000}+e^{2000}+e^{3000}$e^{1000}+e^{2000}+e^{3000}

很明显，在计算上一定会溢出。解决方法也比较简单，就是我们在分子分母上都乘上一个系数，减小数值大小，同时保证整体还是对的

![](https://pic1.zhimg.com/v2-65642e869ce8ff4fb574c2c6e20e0ccc_b.jpg)

把常数 C 吸收进指数里面

![](https://pic4.zhimg.com/v2-a2e85b6b42d7700d5b3007b46f5b15b3_b.jpg)![](https://pic3.zhimg.com/v2-1d5bf301b001e9b5946abe24c59efefa_b.jpg)

这里的 D 是可以随便选的，一般可以选成

![](https://pic1.zhimg.com/v2-7cf62366d484731378e3d2677e83ca1c_b.jpg)

具体实现可以写成这样

```
def stablesoftmax(x):
    """Compute the softmax of vector x in a numerically stable way."""
    shiftx = x - np.max(x)
    exps = np.exp(shiftx)
    return exps / np.sum(exps)

```

这样一种实现数值稳定性已经好了很多，但是仍然会有数值稳定性的问题。比如输入的值差别过大的时候，比如

$[-1000, 1, 10000]$ [-1000, 1, 10000]

这种情况即使用了上面的方法，可能还是报 `NaN` 的错误。但是这个就是数学本身的问题了，大家使用的时候稍微注意下。

一种可能的替代的方案是使用 `LogSoftmax` （然后再求 `exp`），数值稳定性比 softmax 好一些。

$log(S_j) = log(\frac{e^{a_j}}{\sum_{k=1}^Ne^{a_k}})\\ = log(e^{a_j}) - log(\sum_{k=1}^Ne^{a_k})\\ =a_j - log(\sum_{k=1}^Ne^{a_k})$log(S_j) = log(\frac{e^{a_j}}{\sum_{k=1}^Ne^{a_k}})\\ = log(e^{a_j}) - log(\sum_{k=1}^Ne^{a_k})\\ =a_j - log(\sum_{k=1}^Ne^{a_k})

可以看到，`LogSoftmax`省了一个指数计算，省了一个除法，数值上相对稳定一些。另外，其实 `Softmax_Cross_Entropy` 里面也是这么实现的

5. Softmax 的梯度
--------------

下面我们来看一下 softmax 的梯度问题。整个 softmax 里面的操作都是可微的，所以求梯度就非常简单了，就是基础的求导公式，这里就直接放结果了。

![](https://pic4.zhimg.com/v2-50d4ab695e967314b5d1def143584ee3_b.jpg)![](https://pic1.zhimg.com/v2-c442b82205277f6788f8bb8f75466578_b.jpg)

所以说，如果某个变量做完 softmax 之后很小，比如 $S_j = 0$S_j = 0 ，那么他的梯度也是非常小的，几乎得不到任何梯度。有些时候，这会造成梯度非常的稀疏，优化不动。

6. Softmax 和 Cross-Entropy 的关系
------------------------------

先说结论，

**softmax 和 cross-entropy 本来太大的关系，只是把两个放在一起实现的话，算起来更快，也更数值稳定。**

cross-entropy 不是机器学习独有的概念，本质上是用来衡量两个概率分布的相似性的。简单理解（只是简单理解！）就是这样，

如果有两组变量：

$a = [1,2,3]\\ b= [2,4,6]$a = [1,2,3]\\ b= [2,4,6]

如果你直接求 L2 距离，两个距离就很大了，但是你对这俩做 cross entropy，那么距离就是 0。所以 cross-entropy 其实是更 “灵活” 一些。

那么我们知道了，cross entropy 是用来衡量两个概率分布之间的距离的，softmax 能把一切转换成概率分布，那么自然二者经常在一起使用。但是你只需要简单推导一下，就会发现，softmax + cross entropy 就好像

“往东走五米，再往西走十米”，

我们为什么不直接

“往西走五米” 呢？

cross entropy 的公式是

![](https://pic3.zhimg.com/v2-82e18146198b754e07f4ead2189d0bbe_b.jpg)

这里的 $log(q(k))$log(q(k)) 就是我们前面说的 `LogSoftmax`。这玩意算起来比 softmax 好算，数值稳定还好一点，为啥不直接算他呢？

所以说，这有了 PyTorch 里面的 [torch.nn.CrossEntropyLoss](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/stable/nn.html%23crossentropyloss) (输入是我们前面讲的 logits，也就是 全连接直接出来的东西)。这个 CrossEntropyLoss 其实就是等于 [torch.nn.LogSoftmax](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/stable/nn.html%3Fhighlight%3Dlog%2520softmax%23torch.nn.LogSoftmax) **+** [torch.nn.NLLLoss](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/stable/nn.html%23nllloss)。