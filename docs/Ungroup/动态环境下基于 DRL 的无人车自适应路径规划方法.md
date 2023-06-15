> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/79712897)

目录
--

> **1. 前言**  
> ----1.1 知网链接  
> **2. 主要研究内容**  
> **3. D3QN PER 算法**  
> ----3.1 DQN 算法  
> ----3.2 Double DQN 算法  
> ----3.3 Dueling DQN 算法  
> ----3.4 优先经验回放  
> **4. 环境特征融合方案**  
> ----4.1 自身状态信息处理方法  
> ----4.2 激光雷达点云信息处理方法  
> ----4.3 视觉图像信息处理方法  
> ----4.4 环境特征融合方案  
> **5. 实验环境**  
> ----5.1 仿真实验环境的搭建  
> ----5.2 小车的动作空间划分  
> ----5.3 算法参数设置  
> ----5.4 奖励函数的设置  
> **6. 效果展示**  
> ----6.1 无障碍环境  
> ----6.2 静态障碍环境  
> ----6.3 5 个静态障碍 + 5 个动态障碍环境  
> ----6.4 10 个动态障碍环境  
> **7. 完整代码**  
> ----7.1 Software  
> ----7.2 Installation  
> --------7.2.1 解决 pointgrey_camera_driver 编译不过的问题  
> **8. 其他问题**

**1. 前言**
---------

![](https://pic4.zhimg.com/v2-cfa492ac7f52bf9a19b1f9aaa1344d4f_r.jpg)

初衷是，在环境不是完全已知的情况下，希望行星车具备一定的自适应能力，应对环境发生的变化。所以，基于深度强化学习（Deep Reinforcement Learning, DRL）理论提出了端到端的路径规划方法，直接从传感器信息映射出动作指令再发布给行星车。同时采用不同的神经网络结构分别处理不同的传感器信息，最后将环境特征融合在一起，构成基于 D3QN PER 的多传感器行星车路径规划方法。

*   **1.1 知网链接**

[动态环境下多传感器行星车自适应路径规划方法研究 - 中国知网](https://link.zhihu.com/?target=https%3A//kns.cnki.net/kcms/detail/detail.aspx%3Fdbcode%3DCMFD%26dbname%3DCMFD202001%26filename%3D1019647184.nh%26v%3DLnfcL8gimWUKqAWs5BBPgscFtqofLskq1kK376iP2aHfthjGTFB21BTRBgMNZaU7)

**2. 主要研究内容**
-------------

主要研究内容如图：

![](https://pic2.zhimg.com/v2-be53a8e68e1d0528678001e9a7d2e6c1_r.jpg)

技术途径：

![](https://pic4.zhimg.com/v2-8e22292a612853784d29a52aad79e8f7_r.jpg)

**3. D3QN PER 算法**
------------------

关于强化学习的一些基础知识，推荐大家去看专栏中关于 David Silver 课程的笔记：

[搬砖的旺财：David Silver 增强学习——笔记合集（持续更新）](https://zhuanlan.zhihu.com/p/50478310)

*   **3.1 DQN 算法**

![](https://pic3.zhimg.com/v2-28577d0980dab663e016fc58bbedd19e_r.jpg)

直接来看 DQN 的损失函数：

$\mathcal{L}(\boldsymbol{w})=\mathbb{E}\left[\left(r+\gamma \max _{a^{\prime}} {\hat Q}\left(s^{\prime}, a^{\prime}, w^{-}\right)-{Q}(s, a, w)\right)^{2}\right]$\mathcal{L}(\boldsymbol{w})=\mathbb{E}\left[\left(r+\gamma \max _{a^{\prime}} {\hat Q}\left(s^{\prime}, a^{\prime}, w^{-}\right)-{Q}(s, a, w)\right)^{2}\right]

其中， ${\hat Q}${\hat Q} 是目标 Q 网络，参数为 $w^{-}$w^{-} ，负责生成训练过程中的目标，即目标 Q 值 $r+\gamma \max _{a^{\prime}} {\hat Q}\left(s^{\prime}, a^{\prime}, w^{-}\right)$r+\gamma \max _{a^{\prime}} {\hat Q}\left(s^{\prime}, a^{\prime}, w^{-}\right) ； $Q$Q 是当前 Q 网络，参数为 $w$w ；值得注意的是， ${\hat Q}${\hat Q} 和 $Q$Q 的网络结构完全一致。

每训练 C 步， 将当前 Q 网络的参数完全复制给目标 Q 网络，那么，接下来 C 步参数更新的目标将由更新后的目标 Q 网络负责提供。

具体的实施步骤请参见上图～

*   **3.2 Double DQN 算法**

![](https://pic1.zhimg.com/v2-d3f7bf52583f4e2dc5f782a584062914_r.jpg)

Double DQN 算法主要是为了解决 DQN 算法中严重的过估计问题，它将目标 Q 值中动作的选择和动作的评估分开，让它们使用不同的 Q 网络。

先看 Double DQN 中的目标 Q 值： $r+\gamma {\hat Q} \left( s',\textrm{argmax}_{a'} {Q}\left(s', a^{\prime}, w\right);w^{-} \right)$r+\gamma {\hat Q} \left( s',\textrm{argmax}_{a'} {Q}\left(s', a^{\prime}, w\right);w^{-} \right) 。

当前 Q 网络负责选择动作，而带有 “延迟效应” 的目标 Q 网络用来计目标 Q 值，具体的实施步骤请参见上图～

*   **3.3 Dueling DQN 算法**

![](https://pic2.zhimg.com/v2-829cd0b10e7f313580d1dbe2b2674331_r.jpg)

Dueling DQN 相比于 DQN 在网络结构上做出了改进，在得到中间特征后 “兵分两路”，一路预测状态值函数，另一路预测相对优势函数，两个相加才是最终的动作值函数。

*   **3.4 优先经验回放**

在 DQN 中使用经验回放的动机是：作为有监督学习模型，深度神经网络要求数据满足独立同分布假设；但样本来源于连续帧，这与简单的 RL 问题相比样本的关联性增大了不少。 假如没有经验回放，算法在连续一段时间内基本朝着同一个方向做梯度下降，那么在同样的步长下这样直接计算梯度就有可能不收敛。所以经验回放的主要作用是：

> 克服经验数据的相关性，减少参数更新的方差；  
> 克服非平稳分布问题。

经验回放的做法是从以往的状态转移（经验）中随机采样进行训练，如下图所示，这样操作使得数据利用率高，可以理解为同一个样本被多次使用。

![](https://pic2.zhimg.com/v2-d80b61c2cd13ed1e771d9d41d01e6cf9_r.jpg)

优先经验回放（Prioritized Experience Replay, PER）的思路来源于优先清理（Prioritized sweeping），它以更高的频率回放对学习过程更有用的样本，这里使用 TD-error 来衡量作用的大小。TD-error 的含义是某个动作的估计价值与当前值函数输出的价值之差。TD-error 越大，则说明当前值函数的输出越不准确，换而言之，能从该样本中 “学到更多”。 为了保证 TD-error 暂时未知的新样本至少被回放一次，将它放在首位，此后，每次都回放 TD-error 最大的样本。

不过，单独使用 TD-error 进行 PER 也存在许多问题。首先，只有被重放样本的 TD-error 被更新，那么，TD-error 较小的样本在第一次被回放后，由于不更新， 它的 TD-error 会一直被认为很小（其实参数更新后该样本的 TD-error 可能会变大)， 所以该样本很久都不会再重新被回放。再者，对噪声敏感，bootstrapping 会进一步加剧这个问题。最后，PER 的样本会集中在一个小范围内，多样性的缺失容易导致过拟合。

为了解决上述问题，结合纯粹的贪婪优先和均匀随机采样。确保样本的优先级正比于 TD-error，与此同时，确保最低优先级的概率也是非零的。

首先，定义采样经验 $i$i 的概率为：

$P(i)=\frac{p_{i}^{\alpha}}{\sum_{k} p_{k}^{\alpha}}$P(i)=\frac{p_{i}^{\alpha}}{\sum_{k} p_{k}^{\alpha}}

其中， $p_{i}$p_{i} 是第 $i$i 个经验的优先级；指数 $\alpha$\alpha 决定使用优先级的多少，当 $\alpha=0$\alpha=0 时是均匀随机采样的情况。

具体的，采用成比例的优先（Proportional prioritization），即

$p_{i}=\left|\delta_{i}\right|+\varepsilon$p_{i}=\left|\delta_{i}\right|+\varepsilon

其中， $\delta_{i}$\delta_{i} 为 TD-error； $\varepsilon$\varepsilon 是为了防止经验的 TD-error 为 0 后不再被回放。在实现时采用二叉树（Sum tree）结构，它的每个叶子节点保存了经验的优先级，父节点存储了叶子节点值的和。这样，头节点的值就是所有叶子结点的总和。采样一个大小为 $k$k 的 minibatch 时，范围 $\left[0, p_{\text {total}}\right]$\left[0, p_{\text {total}}\right] 被均分为 $k$k 个小范围，每个小范围均匀采样，这样，各种经验都有可能被采样到。

为了消除由采样带来的偏差，加入重要性采样（Importance-sampling）。重要性采样权重为 $\omega_{j}=\left(\frac{1}{N} \cdot \frac{1}{P(j)}\right)^{\beta}$\omega_{j}=\left(\frac{1}{N} \cdot \frac{1}{P(j)}\right)^{\beta} 。其中， $\beta$\beta 用于调节偏差程度，因为在学习的初始阶段有偏差是无所谓的，但在后期就要消除偏差。为了归一化重要性采样权重，使 $\omega_{j}=(\frac{\left(N\cdot P(j)\right)^{-\beta}}{\textrm{max}_{i}\omega_{i}})$\omega_{j}=(\frac{\left(N\cdot P(j)\right)^{-\beta}}{\textrm{max}_{i}\omega_{i}}) 。

来看一下 PER 和 Double DQN 结合后的伪代码：

![](https://pic2.zhimg.com/v2-b3bc3ce470a4dccc7ffa0302408932e5_r.jpg)

4. 环境特征融合方案
-----------

*   **4.1 自身状态信息处理方法**

![](https://pic4.zhimg.com/v2-09efef130eaf90fbf507300667e6897b_r.jpg)

自身状态信息的表示方法请参见上图，它可以表示为一个数组 $[v_t,\omega_t,d_t,\theta_t]$[v_t,\omega_t,d_t,\theta_t] ，其中， $v_t$v_t 和 $\omega_t$\omega_t 为 $t$t 时刻小车的速度和角速度信息， $d_t$d_t 和 $\theta_t$\theta_t 为 $t$t 时刻小车相对终点的距离和角度信息。自身状态信息时刻指引着小车向终点移动。

*   **4.2 激光雷达点云信息处理方法**

LIDAR 产生的点云属于长序列信息，比较难直接拆分成一个个独立的样本来通过 CNN 进行训练。所以采用 LSTM 网络来处理 LIDAR 点云信息，其中 cell 单元为 512 个，具体的网络结构如下图所示：

![](https://pic3.zhimg.com/v2-d6bc7912418bd04534c1cb373f6b1172_r.jpg)

为了方便，控制 LIDAR 输出的点云信息为 360 维，更新频率是 50 赫兹，探测范围为 $[-90^o,90^o]$[-90^o,90^o] ，探测距离为 $[0.1, 10.0]$[0.1, 10.0] ，单位是米，如下图所示：

![](https://pic4.zhimg.com/v2-a2e1fa0d4d854ffed927d697c43dd777_r.jpg)

*   **4.3 视觉图像信息处理方法**

尽管 LIDAR 擅长测量障碍物的距离和形状，但它实际上并不能用于确定障碍物的类型。计算机视觉可通过分类来完成这项任务， 即给定相机的图像，可以标记图像中的对象。

本文采用 CNN 处视觉图像信息。输入图像 $1024\ \textrm{x}\ 728$1024\ \textrm{x}\ 728 经过预处理和叠帧（4 帧）后变为 $80\ \textrm{x}\ 80\ \textrm{x}\ 4$80\ \textrm{x}\ 80\ \textrm{x}\ 4 的单通道灰色图。

![](https://pic2.zhimg.com/v2-2e4317bae22130d97a955d950e6baef1_r.jpg)

这里共采用了三个卷积层，它们分别是 conv1、conv2 和 conv3，具体参数如下：

![](https://pic1.zhimg.com/v2-00535e21178c802a2ef87cc624da9fdc_r.jpg)

卷积层的输入输出示意图如下：

![](https://pic1.zhimg.com/v2-4d48ba4819d0e89a2260108b7ae31730_r.jpg)

*   **4.4 环境特征融合方案**

![](https://pic2.zhimg.com/v2-a4e68b8c8ca19a631f3ff5d3ea7f8325_r.jpg)![](https://pic1.zhimg.com/v2-0e696cb0b76d7b8722ef861c284e28c8_r.jpg)

**5. 实验环境**
-----------

*   **5.1 仿真实验环境的搭建**

基于 ROS 及 Gazebo 搭建的仿真实验环境的框架如图所示：

![](https://pic4.zhimg.com/v2-b3f37ac6ca4c6b74c685dedbc1c6f7df_r.jpg)

基于此，可以改变 Gazebo 中的仿真环境，例如加入不同的障碍，也可以改变小车搭载的传感器。

关于如何使 Gazebo 的仿真速度加快 10 倍，请参考：

[搬砖的旺财：将 Gazebo 的仿真速度加快 10 倍！！！](https://zhuanlan.zhihu.com/p/59702590)

*   **5.2 小车的动作空间划分**

![](https://pic2.zhimg.com/v2-bb8f77772aef936e3326d46ffef732e1_r.jpg)

*   **5.3 算法参数设置**

![](https://pic2.zhimg.com/v2-d29c000909c88c9bbe4790ae81cf5139_r.jpg)

$\epsilon$\epsilon - 贪婪法的 $\epsilon$\epsilon 参数从初始值 1.0 开始按照训练步数线性下降  $\epsilon=\epsilon-\frac{1.0}{\text { Num training }}$\epsilon=\epsilon-\frac{1.0}{\text { Num training }} 。

很有意思的一件事情，经过测试发现，记忆池的大小和训练步数保持 10 倍的关系是最好的～

![](https://pic1.zhimg.com/v2-dcb81f757eda4ac56dd43dca67b63e88_r.jpg)

$\beta$\beta 参数按照训练步数线性下降：  $\beta=\beta+\frac{1.0-\beta_{\text {init}}}{\text {Num training }}$\beta=\beta+\frac{1.0-\beta_{\text {init}}}{\text {Num training }} 。

*   **5.4 奖励函数的设置**

![](https://pic2.zhimg.com/v2-289361da4a3a063765b3db90fb755cd1_r.jpg)

6. 效果展示
-------

*   **6.1 无障碍环境**

*   **6.2 静态障碍环境**

*   **6.3 5 个静态障碍 + 5 个动态障碍环境**

*   **6.4 10 个动态障碍环境**

**7. 完整代码**
-----------

请点击：[CoderWangcai/DRL_Path_Planning](https://link.zhihu.com/?target=https%3A//github.com/CoderWangcai/DRL_Path_Planning)

*   **7.1 Software**

> Ubuntu 16.04  
> ROS Kinect  
> Python 2.7.12  
> Tensorflow 1.12.0

*   **7.2 Installation**

> git clone [https://github.com/CoderWangcai/DRL_Path_Planning.git](https://link.zhihu.com/?target=https%3A//github.com/CoderWangcai/DRL_Path_Planning.git)  
> **（PS：这一步请耐心等待，训练好的 model 比较大～）**  
> cd DRL_Path_Planning  
> catkin_make  
> source devel/setup.bash  
> roslaunch multi_jackal_tutorials ten_jackal_laser_add_apriltag.launch

出现的便是下面的场景：

![](https://pic1.zhimg.com/v2-84f5ed4eb67920c93a14bed8b64adc9c_r.jpg)

然后，把 DRL_Path_Planning/src/tf_pkg/scripts/D3QN_PER_image_add_sensor_dynamic_10obstacle_world_30m_test.py 中的路径补充完整：

> # 静态障碍下训练好的模型  
> self.load_path = '.../jackal/src/tf_pkg/scripts/saved_networks/10_D3QN_PER_image_add_sensor_obstacle_world_30m_2_2019_06_02'

**（PS：如果需要运行其他 python 文件，也请将其中的路径补充完整～）**

另起一个终端：

> python D3QN_PER_image_add_sensor_dynamic_10obstacle_world_30m_test.py

搞定啦，出现的便是训练好的 10 个动态障碍环境～

*   **7.2.1 解决 pointgrey_camera_driver 编译不过的问题**

catkin_make 可能会遇到关于 pointgrey_camera_driver 的问题：

![](https://pic4.zhimg.com/v2-6bb47d5890ffee6aa93c76edd1f2697f_r.jpg)

解决方法如下：

> cd src/  
> git clone [https://github.com/ros-drivers/pointgrey_camera_driver](https://link.zhihu.com/?target=https%3A//github.com/ros-drivers/pointgrey_camera_driver)  
> cd ..  
> catkin_make

![](https://pic2.zhimg.com/v2-6307dcbc5eed5418b488259a69938505_r.jpg)

编译到上面这里的时候卡住了，没关系，ctrl+c，然后把刚刚 Download 的 pointgrey_camera_driver 文件夹删除，因为现在已经生成相关的消息了，它不起作用了。

> catkin_make

没问题啦，整个文件夹都编译过了～

**8. 其他问题**
-----------

如有问题请提 issue～

特别鸣谢 YP 师兄，师兄在整个毕设的过程中，给了我很多技术上的支持，最后也帮我反复修改论文的结构～

请大家批评指正，谢谢～