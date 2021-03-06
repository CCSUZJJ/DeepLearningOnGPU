深度学习
========

深度学习与前些年神经网络的区别，那就是网络层数要比以前多了。这样的话功能就大大加强了。

网络一组layer来组成，layer 又由一组node来组成，层与层之间联接是由forword与backwrod连接组成。每一层都solver，以及module. 而这里blobs 就是node. 
foward 用来把input -> output同时计算出 loss cost,而backword 而是根据loss cost来计算梯度来提取下一步w,b，优化到什么程度为止呢，取决于error 要求和iter numbers.

每一个小的神经网络连接再加上形成大的神经网络，就实现了脑的功能。

Blobs 是由包着数据本身（什么意思？），而数据本身四维的数组(Num,Channels,Height and Width) 有点类似于CUDA的Thread分配。Blobs更像是CUDA Array 的机制。Blobs统一管理CPU与GPU的内存。(n,k,h,w)正好每个线程计算输入都是由Blobs来的。输入传输，利用google proto来进传输。 `Google Protocol Buffer 的使用和原理 <http://www.ibm.com/developerworks/cn/linux/l-cn-gpb/>`_  google 这种机制就其名字一样，（把什么当作？）当作协议包来解析，就像网络包一样，每一个包不需要通用，只要自己能认识自己就行了。就像网络协议一样，只规定了协议包结构。就自然解析就容易了。无非两种TLV,并且进一步利用编码本身实现压缩，并且元编程实现类的自动化定义与实现。并且还支持动态编译JIT,这也就意味着没有只要有源码就行了。

同时Blobs,保存两种数据，一种是Data,一种是diff,简单的说Data是往前传，而diff计算梯度是往后传。

这个是把编码与元编程技术结合起来的。 

caffe 这个神经网络还可以snapshotting与resume,这个就非常的方便。

深度网络
========

本质是一层一层向上抽象的过程。同时一个由无序到有序的过程。但是过程的路径又不指一条，就像人每个人的思维过程也是一样的。所以每一层的每个神经网络的抽象可以不同。并且再加上一顺序就会有各种各样的结果。到这个过程中就像需要有一个像分词库一样的东东。如果只是简单组合就能得到一属性，还是排列才能进行判定， 排列与组合是不同层次的判别路线。

深学习就是在解决如抽象概念这个难题。

caffe流程
=========

caffe的设计原则与自己的原则一样的，分层的模块化的设计。把一个大大的问题不断的breakdown,变成几个已经解决的小问题。
#. 数据格式的转换，目前支持 lmdb,leveldb.
#. 定义 network
   
   .. graphviz::
      
      digraph flow{
          Input->Forward->Output;
          Output->Backword->Input;

      }
#. 定义 solver
#. training
#. testing

开发流程添加几个头文件与与源文件。然后实现setup,initial, resharp,forward,backword,然后就注册就行了。


数据的准备
==========

在网络 里主要有  ``data_para`` 与 ``transform_para`` . 

#. from database
   levelDB or LMDB
#. directly from memory
#. from files on disk in HDF5
#. common image formats

而预处理包括,通过指定 TransformationParameterS.

#. mean subtraction,
#. scaling
#. random cropping
#. mirroring

数据格式
========

key,value格式，leveldb,lmdm等等就是这种。

`hdf5 <http://www.hdfgroup.org/products/java/hdfview/>`_ 
用来层次化的数据格式，相当于面向对象结构串行化。 如果其看成是例如IP的解析则更容易理解，例如Mongo那种理解，就相当于TLV格式的包结构。
Hierarchical Data format http://en.wikipedia.org/wiki/Hierarchical_Data_Format.

深度学习和并行计算
==================

早期的神经网络，因无法处理两层以上的网络，都是浅层的。目前的深度学习，因采取逐层预训练和全局微调策略，能够实现深层次的网络结构（目前通常需要10层以上的隐含层，才能够取得较好的建模效果），但实际上训练一层的神经网络是非常困难的。这是因为当前的深度学习网络中存在上百万、甚至是上千万的参数需要经过后向反馈来调整，并且深度学习需要大量的训练数据来确保分类和预测的准确性，也就意味着几百万甚至是几千万的输入数据需要运行在前向或者后向反馈中，因此计算量是非常巨大的。另一方面，深度学习由大量相同的、并行的神经元组成，能够并行映射到GPU中，能够提供巨大的计算加速。这里使用NVIDIA公司提供的深度学习平台：CAFFE。 实验已表明，当训练“reference imagenet”时， 并行计算能够获得10倍于intel ivyBridge CPU的加速。

NVIDIA cuDNN 是主要的深度学习GPU加速库，它提供深度学习中常用的操作。比如：卷积、池化、softmax、神经元激活（包括sigmoid、矫正线性器和Hyperbolic tangent ），当然这些函数都支持前向、后向传播，cuDNN主要是在以矩阵相乘的方面擅长，cuDNN特性定数据输出，支持可伸缩的维度定制， dimension ordering, striding and subregions for 对于四维张量。这些可伸缩性允许神经元积分，并且能够避免输入输出转换。

cuDNN是线程安全的，提供基于内容的APi，从而能够支持多线程和共用CUDA线程。使得开发者能够精确地使用多主线程和多GPU，控制库。确保GPU设备总是在一个特殊的主线程中使用。

cuDNN允许深度学习开发者能够掌握当前的状态并且能够把精力集中在应用和机器学习问题上，而不需要写额外的定制代码。cuDNN能够 在Windows和LInux OSes平台上。支持所有的NVIDIA GPU,从低端的GPU，比如Tegra k1 到高端的 Tesla K40. 当开发者使用cuDNN，能够确保得到目前的和未来的高性能，并且得益于GPU特性。

cuDNN是为了深度学习开发者，它非常易用，使得开发者无需知道CUDA。在CAFFE，深度学习使用基于本文的配置文件。使用CAFFE，使用定义神经网络的每一层，制定层的类型（包括数据、卷积和全连接）并且层提供他的输入。并且有类似的配置文件，定义怎样初始化神经网络参数以及训练的重复次数等等。

