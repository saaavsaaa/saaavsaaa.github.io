### 单因子模型的训练
本章介绍的声学模型特指经典的基于隐马尔科夫模型(Hidden Markov Model,HMM) 语音识别框架中的声学模型。本节将关注一种基本的模型结构：使用高斯混合模型(Gaussian Mixture Model,GMM) 描述单音子(Monophone) 发音状态的概率分布函数(Probability Distribution Function,PDF) 的 HMM 模型。这种结构的声学模型在原理和训练流程上，是训练其他声学模型的基础。

#### 声学模型的基本概念
在经典的语音识别框架中，一个声学模型就是一组 HMM。一个 HMM 的参数由初始概率、转移概率、观察概率三部构成。对于语音识别框架中的声学模型里的每个 HMM。应当定义该 HMM 中有多少个状态，以及各个状态起始的马尔科夫的初始概率，各状态间的转移概率及每个状态的概率分布函数。

实践中，一般设初始概率恒为1，因此不必在模型中记录。转移概率对识别结果的影响很小，甚至有时可以忽略。所以实践中经常把状态的转移概率预设为固定值，不在训练中更新转移概率。声学模型包含的信息，主要是状态定义和各状态的观察概率分布。如果用高斯混合模型对观察概率分布建模，那么就是 GMM-HMM 模型；如果用神经网络模型对观察概率分布建模，那么就是 NN-HMM 模型。

HMM 状态的物理意义在语音识别中可以认为是音素的发音状态。习惯上把一个音素的发音状态分为三个部分，分别是：”初始态“、”稳定态“、”结束态“。对应的，一个音素的发音用三个 HMM 状态建模。

需要说明的是，音素建模的状态不一定是三个，只是在传统语音识别方法中用三个状态对音素建模更加常见。目前 Kaldi 主推的 chain model 使用两个状态的建模方法来建模音素的起始帧和其他帧。而在传统的基于 HMM 的语音合成(TTS) 中更常用5个状态对音素建模。     

根据声学模型,可以计算某一帧声学特征在某一状态上的声学分（AM score）。这里所说的声学分，指的是该帧声学特征对于该状态的对数观察概率，或者称为对数似然值（Log-likelihood）:     
　　　　　　　　　　　　AMScore(t,i) = logP(o<sub>t</sub>|s<sub>i</sub>)       
~<sub>t</sub>  <sup>t</sup>~      
在上式中，AMScore(t,i) 是第 t 帧语音特征 o<sub>t</sub> 在第 i 个状态 s<sub>i</sub> 上的声学分。   
观察概率的经典建模方法是高斯混合模型（Gaussian Mixture Model,GMM）。GMM 的思路是使用多个高斯分量加权叠加，拟合出任意分布的概率密度函数。p109图   
GMM 建模观察概率可以用公式表示：

https://saaavsaaa.github.io/jax/escape.html     
$$
logP(o_t|s_i) = log\sum_{m=1}^{M}\frac{c_{i,m}exp(-\frac{1}{2}(o_t-μ_{i,m})^T(∑_{i,m}^{-1})(o_t-μ_{i,m}))}{(2π)^{\frac{D}{2}}|∑_{i,m}|^{\frac{1}{2}}}
$$    

$$
logP(第t帧语音特征|第i个状态) = log\sum_{m=1}^{M}\frac{c_{i,m}exp(-\frac{1}{2}(第t帧语音特征-状态s_i的第m个高斯分量的D维均值向量)^T(状态s_i的第m个高斯分量的协方差矩阵^{-1})(第t帧语音特征-状态s_i的第m个高斯分量的D维均值向量)}{(2π)^{\frac{D}{2}}|状态s_i的第m个高斯分量的协方差矩阵|^{\frac{1}{2}}}
$$     

-----

<iframe src="https://saaavsaaa.github.io/jax/t.html?a=%24%24%0AlogP%28o_t%7Cs_i%29%20%3D%20log%5Csum_%7Bm%3D1%7D%5E%7BM%7D%5Cfrac%7Bc_%7Bi%2Cm%7Dexp%28-%5Cfrac%7B1%7D%7B2%7D%28o_t-%u03BC_%7Bi%2Cm%7D%29%5ET%28%u2211_%7Bi%2Cm%7D%5E%7B-1%7D%29%28o_t-%u03BC_%7Bi%2Cm%7D%29%29%7D%7B%282%u03C0%29%5E%7B%5Cfrac%7BD%7D%7B2%7D%7D%7C%u2211_%7Bi%2Cm%7D%7C%5E%7B%5Cfrac%7B1%7D%7B2%7D%7D%7D%0A%24%24%0A%20%20" height="100px" width="700px" frameborder="0" scrolling="no"> </iframe>

-----

注：   
套用高斯分布exp；协方差度量两个变量的相关性，两个相同时退化成方差   
混合高斯模型（GMM）指对样本的概率密度分布进行估计，而估计采用的模型（训练模型）是几个高斯模型的加权和（具体是几个要在模型训练前建立好）。每个高斯模型就代表了一个类（一个Cluster）。选取样本中的数据分别在几个高斯模型上得到的概率，例如可以选取概率最大的类作为分类结果。   

<iframe src="https://saaavsaaa.github.io/jax/t.html?a=%24%24%0AlogP%28%u7B2Ct%u5E27%u8BED%u97F3%u7279%u5F81%7C%u7B2Ci%u4E2A%u72B6%u6001%29%20%3D%20log%5Csum_%7Bm%3D1%7D%5E%7BM%7D%5Cfrac%7Bc_%7Bi%2Cm%7Dexp%28-%5Cfrac%7B1%7D%7B2%7D%28%u7B2Ct%u5E27%u8BED%u97F3%u7279%u5F81-%u72B6%u6001s_i%u7684%u7B2Cm%u4E2A%u9AD8%u65AF%u5206%u91CF%u7684D%u7EF4%u5747%u503C%u5411%u91CF%29%5ET%28%u72B6%u6001s_i%u7684%u7B2Cm%u4E2A%u9AD8%u65AF%u5206%u91CF%u7684%u534F%u65B9%u5DEE%u77E9%u9635%5E%7B-1%7D%29%28%u7B2Ct%u5E27%u8BED%u97F3%u7279%u5F81-%u72B6%u6001s_i%u7684%u7B2Cm%u4E2A%u9AD8%u65AF%u5206%u91CF%u7684D%u7EF4%u5747%u503C%u5411%u91CF%29%7D%7B%282%u03C0%29%5E%7B%5Cfrac%7BD%7D%7B2%7D%7D%7C%u72B6%u6001s_i%u7684%u7B2Cm%u4E2A%u9AD8%u65AF%u5206%u91CF%u7684%u534F%u65B9%u5DEE%u77E9%u9635%7C%5E%7B%5Cfrac%7B1%7D%7B2%7D%7D%7D%0A%24%24%0A%20%20" height="100px" width="700px" frameborder="0" scrolling="no"> </iframe>

-----

在上式中，μ<sub>i,m</sub> 为状态 s<sub>i</sub> 的第 m 个高斯分量的 D 维均值向量，∑<sub>i,m</sub> 为状态 s<sub>i</sub> 的第 m 个高斯分量的协方差矩阵。在声学模型训练中，为了降低模型参数量，通常令协方差矩阵为对角阵：∑<sub>i,m</sub> = diag(δ<sub>i,m</sub>) ，其中δ<sub>i,m</sub>为D维向量。   

一个 GMM-HMM 声学模型存储的主要参数为各状态和高斯分量的 μ<sub>i,m</sub>、δ<sub>i,m</sub>、c<sub>i,m</sub>。使用 Kaldi 提供的模型复制工具可以方便地查看声学模型的内容。2.4小节提到过的一个 GMM-HMM 声学模型，存储为 final.mdl。 使用命令可以把这个模型转换成文本格式查看：gmm-copy -- binary=false final.mdl final.mdl.txt。内容提供在p110     

可以看到模型文件有一个 <TransitionModel> 和多个 <DiagGMM> 构成。<TransitionModel> 存储 Transition 模型，它定义了每个因素由多少个状态构成等信息。<DiagGMM> 用于描述状态概率分布，每个的 DiagGMM 即为一个状态的高斯分量的概率分布函数，也经常被称作一个 PDF，内容由 <MEANS_INVVARS>、<INV_VARS>、<WEIGHTS>、<GCONSTS> 四部分构成，其实这四部分存储的信息仍然是上文讲到的的 μ<sub>i,m</sub>、δ<sub>i,m</sub>、c<sub>i,m</sub>。为了减少实时计算量，	Kaldi 并不是直接存储这些参数的，而是用这些参数做了一个概率密度的预计算，如矩阵求逆等，把计算结果存储模型中。     

#### 4.2.2将声学模型用于语音识别    

在深入声学模型训练的细节前，我们先看一下如何在语音识别的过程中使用一个已经训练好的声学模型。简单地说，识别的过程就是用语音的特征序列去匹配一个状态图，搜索最优路径。状态图中有无数路径，每条路径代表一种可能的识别结果，且都有一个分数，该分数表征语音和该识别结果的匹配程度。判断两条路径的优劣，就是比较这两条路径的分数，分数高的路径更忧，即高分路径上的识别结果和声音更匹配。    

状态图由若干节点和若干条跳转构成。有的跳转对应一个 HMM 状态，并在识别过程中对当前帧计算一个分数，分数由两部分构成，声学分和图固有分(Graphscore)，两者之和构成了该跳转在当前帧上的分数。   

图固有分主要来源于语言模型概率，同时也来源于发音词典的多音词选择概率和 HMM 模型的转移概率。这些概率在状态图的构建过程中就固定在了状态图中，和待识别的语音无关。声学分则是在识别过程中根据声学模型和待识别语音的匹配关系动态计算的，声学模型在语音识别过程中的最主要作用就是计算声学分。   

有无数种方法声学特征的帧从状态图的起始节点依次对应到各个跳转上，每种方法都对应图上的一条路径，由于路径是由路径上若干条跳转构成的，因此路径的分数是路径上各条跳转的分数的和，每条跳转和声学特征的一帧相对应。最优路径代表的状态序列就是状态机级识别结果。如果状态图的构建中包含了单词级信息，那么就可以反推出单词级别的识别结果，就完成了识别。      

实际场景中的状态图可能会有一些虚拟跳转。Kaldi 的状态图是基于 WFST 构建的。   

#### 4.2.3模型初始化
下面看一下 Librispeech 示例中第一个声学模型的训练。run.sh中的相关代码: $stage -le 8。   

在上面的代码中，只有 steps/trainmono.sh 这一行是用于模型训练的，后边的代码都是用于测试的。脚本 steps/trainmono.sh 用于训练一个单音子的 GMM-HMM 模型，使用方法是：steps/trainmono.sh [options] <data-dir> <lang-dir> <exp-dir>   

其中 <data-dir>、<lang-dir> 是输入路径，分别是训练数据目录的路径和第三章生成的语言目录的路径。<exp-dir> 是输出模型目录的路径，习惯上设为 exp/mono，用于存储训练完成后的声学模型相关文件，模型默认命名为 final.mdl。

-----

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Kaldi_abstract_1.md)