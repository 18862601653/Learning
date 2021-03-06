# Membership Inference Attacks Against Machine Learning Models

## 论文概述
这篇文章提出了Membership Inference攻击，该攻击可以对于一条给定的记录，以黑盒的方式访问机器学习模型，判断这条记录是否在模型的训练集中，从而造成信息泄露。文章的实验基于Google和Amazon提供的“Machine Learning as a service”服务，在真实的数据集上利用影子训练技术来训练攻击模型，由于机器学习算法存在过拟合的问题，所以根据这个性质可以区分记录是否在模型的训练集中。

## 部分记录

攻击黑盒模型（由商业界建立的“machine learning as a service”）比攻击白盒模型复杂，因为白盒模型的结构和参数对于攻击者来说是已知的。

为了构建攻击模型，本文提出了shadow training技术：首先，创建多个“shadow models”用来模仿目标模型的行为，但是训练的数据集和ground truth是已知的，然后用标记的影子模型的输入和输出训练攻击模型。

为影子模型生成训练数据的有效方法：
  - 使用黑盒模型访问目标模型来合成数据（不假设任何关于目标模型训练数据分布的先验知识）
  - 统计来自目标训练数据集绘制的总体（在推断一个给定记录是否在训练数据集中之前，允许攻击者查询目标模型一次)
  - 假设攻击者可以访问有噪音的目标训练数据集版本（在推断一个给定记录是否在训练数据集中之前，允许攻击者查询目标模型一次)

在黑盒场景下，敌人仅仅可以供给模型输入并接收模型的输出，有些场景下，模型可以为敌人间接地使用。比如，一个app的开发者可以使用机器学习服务从app收集到的数据中构建模型，并且让app对结果模型调用API。这个案例中，敌人提供app输入（而不是直接给模型）并且从app接收输出（这个是基于模型的输出的）。

对于任何输入数据记录，模型输出记录属于某一类的概率的预测向量，每类一个。具有最高置信度的类被选做这个数据记录的预测标记。模型的正确性通过测量它怎样生成训练集之外的数据以及它怎样预测来自同一个总体其他数据记录的标记来评估。

假设：
- 机器学习算法用来训练分类模型，可以捕获数据记录和标签内容之间的关系
- 假设攻击者可以查询模型并获得任何数据记录上的模型预测向量，攻击者知道模型输入和输出的格式，包括他们可以采用的数字和值域。
- 假设攻击者：或者1）知道机器学习模型的类型和结构以及训练算法，或者2）黑盒访问用来训练模型的机器学习服务平台（不需要知道模型结构的先验知识和元参数）

inference攻击：
给攻击者一个数据记录和目标模型的黑盒查询。如果攻击者能正确地确定这个数据记录是否是模型训练数据集中的一部分，那么攻击成功。

度量方式：
- precision（查准率），认为是训练数据集中的数据确实在数据集中的比率
- recall（查全率），在训练数据集中的数据被攻击者认为是成员的比率

membership inference攻击利用的是观察到的机器学习模型通常在他们受过训练的数据以及第一次碰到的数据上表现不同，过拟合是其中一个常见的原因。攻击者的目标是构建一个攻击模型，这个攻击模型能够识别目标模型的行为差异，并使用这些差异根据目标模型的输出来唯一地区分目标模型数据集的成员。为了提高准确性，这个攻击模型是模型的集合，每个模型输出目标模型的类别，因为目标模型根据输入的真实类别在输出类别上产生不同的分布。

攻击模型使用多个影子模型，影子模型与目标模型表现相似，与目标模型相反的是，我们知道每个影子模型的ground truth，即一个给定记录是否在训练数据集中。因此，可以在影子模型的输入和相应的输出上使用有监督的训练，来教攻击模型怎样区分在训练集成员及非训练集成员的影子模型输出。

形式定义：
- 目标模型Ftarget()
- 私有训练集D（train，target），其中包含标记的数据记录（Xi，Yi）target
- 数据记录Xitarget是模型的输入
- Yitarget是真实标签，它可取的值来自大小为Ctarget的类集合
- 目标模型的输出是大小为Ctarget的概率向量，向量中的元素值为0或者1，元素总和为1
- 攻击模型Fattack()
- 输入Xattack由正确的标签记录和大小为Ctarget的预测向量组成
- 由于攻击的目标是成员推断决策，所以攻击模型是二分类，输出类型是“in”和“out”

对于一个打了标签的记录（X，y），使用目标模型来计算预测向量Y=Ftarget（X）。Y的分布（分类置信度）高度依赖于X的真实分类。由于Y的概率分布在y的周围，攻击模型成员概率Pr{（X，y）∈D（train，target）}，即（（X，y），Y）属于“in”的概率，或者说X在Ftarget（）的训练数据集中的概率。

主要挑战是：在攻击者对目标模型的内部参数不知情并只能通过公共API查询的情况下，如何训练攻击模型来区分目标模型的训练数据集成员和非成员。为解决这个难题，开发了影子训练技术，这样可以在代理目标上训练攻击模型，而在代理目标上是知道训练数据集的，因此可以执行有监督的训练。

影子模型：
攻击者创建k个影子模型Fshadow_i(),每个影子模型i在数据集D（train，shadow）i上训练，数据集D（train，shadow）i与目标模型的训练数据集有相同的格式和相似的分布。假设用于训练影子模型的数据集与用于训练目标模型的私有数据集不相交，这对攻击者来说是最坏的情况，如果训练集有重叠攻击效果好些。

影子模型必须与目标模型以相似的方法训练，如果目标训练算法（神经网络、SVM、逻辑回归）和模型结构（神经网络的连接）都已知的话就很容易。机器学习服务中，目标模型的类型和结构都是未知的，但是攻击者可以准确地使用与用来训练目标模型相同的服务来训练影子模型。并且影子模型越多，攻击模型的准确率就越高。因为攻击模型被训练来识别影子模型在不同数据集（训练数据集的成员和非成员）上操作的行为差异，所以更多的影子模型提供给攻击模型更多的训练素材。

为影子模型合成训练数据：
为了训练影子模型，攻击者需要训练数据，这些训练数据需要与目标模型的训练数据具有相似的分布。以下是几个生成这种数据的方法：
- 基于模型合成：

如果攻击者既没有真实的训练数据，也没有任何关于训练数据的分布的统计，他可以使用目标模型自己生成训练数据。这么做的直觉是，由目标模型分类的具有高置信度的记录在统计上与目标训练数据集相似，因此为影子模型提供了好的素材。

合成程序有两个阶段：1）搜索，使用爬山算法，搜索可能的数据记录空间以找出被目标模型用高置信度划分的输入；2）采样，从这些记录中采样生成数据。在合成一个记录后，攻击者可以重复此过程直到影子模型的训练数据集满了。

这个合成程序只有敌人可以有效探测可能输入的空间并且发现由目标模型高置信度划分的输入。例如，如果输入是高分辨率的图像，而目标模型执行复杂的图像分类任务，则可能不起作用。

- 基于统计合成：
攻击者可能有一些关于目标模型训练数据的统计信息，比如攻击者可能有不同属性边缘分布的先验知识。实验中通过从它自己的边缘分布中独立采样每个属性的值为影子模型生成合成训练记录。结果攻击模型很有效。

- 嘈杂的真实数据：
攻击者可以访问一些与目标模型训练数据相似的数据，并可以认为是一个有噪音的版本。实验中使用位置数据集，通过翻转10%或20%随机选择的属性值进行模拟，然后在产生的噪音数据集上训练影子模型。这个场景模拟一种情况：目标和阴影模型的训练数据没有从完全相同的总体中取样，或者以非均匀的方式取样的情况。

影子训练技术背后的主要思想是：相似的模型使用同样的服务在相对相似的数据记录上训练以相似的方式表现。

实验结果表明，学习怎样在影子模型训练数据集（知道ground truth，能够在有监督的训练中容易地计算代价函数）中推测成员生成的攻击模型同样能成功地在目标模型训练数据集上推测成员。

实验的三种目标模型：两个通过基于云的机器学习服务平台构建，一个本地实现。所有情况下，攻击都把这些模型看成黑盒，对于云服务，我们不知道他们创建模型的类型和结构，也不知道在训练过程期间使用的超参数的值。

机器学习服务：
- Google Prediction API：用户上传数据集并获取API查询结果模型，用户不能改变配置参数。
- Amazon ML：用户不能选择模型的类型，但是能控制一些元参数。在实验中，可以改变训练数据的最大传递次数和L2正则化值。前者确定训练期的次数并控制模型训练收敛，默认值是10；后者调节模型参数的正则化程度，防止过拟合。

攻击奏效的原因：
- 过拟合：在训练数据集上效果好，在测试数据集上效果不好，可以泄露训练数据。
- 不同的机器学习模型因为他们的不同结构记住他们训练数据集的不同的信息量。

缓解措施：
- 限制预测向量的top-k类，k越小，模型泄露的信息越少
- 预测向量的粗精度，精度越粗，模型泄露的信息越少
- 增加预测向量的熵
- 使用正则化，防止过拟合

其他机器学习相关的攻击：
- 在统计和机器学习模型上的攻击：
已知的SVM和HMM模型的参数被用来推测训练数据集的一般统计信息

利用协同推荐系统输出的改变推测引起这些改变的输入，这些攻击利用基于协同过滤的推荐系统特有的时间行为。

- 模型反演：
使用应用于隐藏输入的模型的输出来推断该输入的某些特征。模型反演产生的特征的平均值最多可以描述整个输出类，它既不构造训练数据集的特定成员，也不能对给定的输入和模型确定特定输入是否用于训练模型。
- 模型提取：
目的是提取训练私有数据的模型的参数，攻击者的目标是构建一个模型，它在验证数据上的预测性能与目标模型相似
- 隐私保护机器学习：
机器学习的隐私保护主要注重于不直接访问训练数据的学习。

## 知识点
- membership inference
  给定一个机器学习模型和一条记录，确定这条记录是否是模型的训练集中的一部分。
- 机器学习算法
  无监督的训练模型目标是从未标记的数据中提取有用的性质，并建立一个可以解释它的隐藏结构的模型；有监督的训练模型的训练集被打了标签，目的是学习数据和标签之间的关系，并构造一个模型使其适用于一般数据。
  
  模型训练算法的目的是最小化模型在训练数据集上的预测错误，但是这可能会导致过拟合，即对训练样本预测效果很好，但是对测试样本预测效果很差。所以正则化技术被提出来防止过拟合同时还最小化预测错误。
  
  有监督的训练常用于分类和其他预测任务。
  
- Machine learning as a service
  主流的互联网公司在他们的云服务平台上提供了机器学习服务，如Google Prediction API，Amazon Machine Learning、Microsoft Azure Machine Learning以及BigML。这些平台提供上传数据以及训练和查询模型的简单的API。
  
  模型和训练算法的细节对数据拥有者不可见，服务提供者基本不提供正则化，所以可能导致过拟合。模型不能下载，只能通过服务API访问，因而称之为黑盒。
  
- 模型反演中关于隐私的概念：如果一个敌人可以使用模型的输出来推断作为模型输入的非预期（敏感）属性的值，就会发生隐私泄露。

- CIFAR-10和CIFAR-100是用来评估图像识别算法的基准数据集。CIFAR-10由10类32 * 32色的图片组成，每类6000张图片。总的来说，一共有50000张训练图片和10000张测试突破。CIFAR-100和CIFAR-10有相同的格式，但是有100类，每类600张图片，其中500张用来训练，100张用来测试。
