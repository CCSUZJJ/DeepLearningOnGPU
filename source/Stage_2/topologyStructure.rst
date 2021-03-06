深度网络拓扑结构
****************

.. graphviz::
   
   digraph{
      a [label="初始层"]
      b [label="隐层1（中层特征：边缘？？）"]
      c [label="隐层2（高层特征：shape？）"]
      d [label="决策层"]
      a->b  [label="卷积层"]
      c->b  [label="auto-coder"]
      b->d  [label="跨层网络实现多尺度特征"] 
      b->c  [label="使用sparse coding层层抽象"]
      c->d  [label="全局特征决策"]
   }


神经元结构
==========

#. 神经元从结构功能上来说：包括感知神经元和激活神经元。
#. 从感知神经元包括：线性的，卷积，限制玻尔兹曼机。
#. 激活函数包括（只要是连续即可）：sigmod，tanh，softmax，retified linear 。


每一种结构都是基于数学推理或者仿生学的原理。
并且网络能够自动的组成连接的拓扑，这才是人脑的过程。并且是动态的。
而现在的网络是静态的过程。


网络框架
========

#. 基于限制波尔兹曼机

   - 卷积限制玻欠兹曼机
   - 三阶因子玻尔兹曼机

#. 基于自编码

   - 去噪自编码器
   - 变换自编码器

#. 卷积神经网络
    
   - 卷积神经网络
   - 卷积分解神经网络

.. include:: Stage_2/toplogyStructure/cnn.rst
.. include:: Stage_2/toplogyStructure/RBM.rst
.. include:: Stage_2/toplogyStructure/SVM.rst

