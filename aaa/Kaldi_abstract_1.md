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

识别的过程就是用语音的特征序列去匹配一个状态图，搜索最优路径。状态图中有无数路径，每条路径代表一种可能的识别结果，且都有一个分数，该分数表征语音和该识别结果的匹配程度。判断两条路径的优劣，就是比较这两条路径的分数，分数高的路径更忧，即高分路径上的识别结果和声音更匹配。    

状态图由若干节点和若干条跳转构成。有的跳转对应一个 HMM 状态，并在识别过程中对当前帧计算一个分数，分数由两部分构成，声学分和图固有分(Graphscore)，两者之和构成了该跳转在当前帧上的分数。   

图固有分主要来源于语言模型概率，同时也来源于发音词典的多音词选择概率和 HMM 模型的转移概率。这些概率在状态图的构建过程中就固定在了状态图中，和待识别的语音无关。声学分则是在识别过程中根据声学模型和待识别语音的匹配关系动态计算的，声学模型在语音识别过程中的最主要作用就是计算声学分。   

把声学特征的帧，从状态图的起始节点，依次对应到各个跳转上的每种方法，都对应图上的一条路径。由于路径是由路径上若干条跳转构成的，因此路径的分数是路径上各条跳转的分数的和，每条跳转和声学特征的一帧相对应。最优路径代表的状态序列就是状态机级识别结果。如果状态图的构建中包含了单词级信息，那么就可以反推出单词级别的识别结果，就完成了识别。      

实际场景中的状态图可能会有一些虚拟跳转。Kaldi 的状态图是基于 WFST 构建的。   

#### 4.2.3模型初始化
下面看一下 Librispeech 示例中第一个声学模型的训练。run.sh中的相关代码: $stage -le 8。   

在上面的代码中，只有 steps/train_mono.sh 这一行是用于模型训练的，后边的代码都是用于测试的。脚本 steps/train_mono.sh 用于训练一个单音子的 GMM-HMM 模型，使用方法是：steps/train_mono.sh [options] <data-dir> <lang-dir> <exp-dir>   

其中 <data-dir>、<lang-dir> 是输入路径，分别是训练数据目录的路径和第三章生成的语言目录的路径。<exp-dir> 是输出模型目录的路径，习惯上设为 exp/mono，用于存储训练完成后的声学模型相关文件，模型默认命名为 final.mdl。   

下面分析下训练脚本 steps/train_mono.sh 的内容，脚本的前半部分是比较琐碎的配置参数、处理声学特征等。当 stage -3 时，开始运行该脚本的第一个核心模块，使用 gmm-init-mono 工具创建初始模型。gmm-init-mono <topology-in> <dim><model-out> <tree-out> : gmm-init-mono topo 39 mono.mdl mono.tree   

将3.5.2节中介绍的 topo 文件和声学特征维数作为输入，该工具就会生成一个初始声学模型，本例存储在 exp/mono/0.mdl。可使用 gmm-copy 查看。这是一个完整的声学模型了，当然识别率非常低。gmm-init-mono 并不要求输入任何训练数据,仅仅初始化了一个基础模型,后续需要使用训练数据来更新这个模型参数。另外这个基础模型的每一个状态只有一个高斯分量，在后续的训练过程中，会进行单高斯分量到混合多高斯分量的分裂。   

在脚本steps/train_mono.sh中，实际调用gmm-init-mono工具时的参数稍复杂一些：   
```
$cmd JOB=1 $dir/log/init.log \
    gmm-init-mono $shared_phones_opt "--train-feats=$feats subset-feats --n=10 ark:- ark:-|" $lang/topo $feat_dim \
    $dir/0.mdl $dir/tree || exit 1;
```
这行脚本在调用 gmm-init-mono 工具时额外设置了两个选项。一个是 $shared_phones_opt，可以让某些音素共享相同的 pdf,默认为空；另一个使用 --train-feats 指定训练数据目录。虽然 gmm-init-mono 工具不要求提供训练数据，但如果提供了训练数据，gmm-init-mono 就会通过统计这些数据的均值方差来使初始模型的参数有一个更好的训练起点，有利于模型的后续训练。   

接下来就要使用训练数据迭代更新模型的参数了。在经典 HMM 理论中，训练 GMM-HMM 使用 Baum-Welch 算法，该算法并不需要预先得知训练样本每一帧具体对应哪个状态，只需给出训练样本的状态序列，就可基于**期望最大化算法(Expectation-Maximization Algorithm)** ，求取各参数在整个序列上的最大似然估计(Maximum Likelihood Estimation,MLE)。关于 EM 算法的详细推导可参考相关文献。   

Kaldi 的实现使用了一种更直接的训练方案。虽然也使用 EM 算法训练，但只把 EM 算法应用到 GMM 参数的更新上，要求显示地输入每一帧对应的状态，使用带标注的训练数据更新 GMM 的参数，这种训练方法比 **Baum-Welch** 算法速度更快，模型性能却没有明显损失，该方法被称为**维特比训练(Viterbi training)** 。   

#### 4.2.4 对齐   
要进行维特比训练,需要解决的一个问题是如何获取每一帧对应的状态号，作为训练的标签。

我们来看 Kaldi 的做法，在 stage -2 阶段构建了一个直线型的状态图，其内容只包含训练句子的标注文本所对应的状态：   

stage -le -2 阶段脚本的核心是调用 compile-train-graphs 工具，这个工具的原理属于 WFST 构图的范畴(第5章)。这里只看该工具的输出，该工具输出了一个状态图，给出了训练语音的状态序列。   

在实际使用时，由于存在多音词的情况及对图的优化算法，因此实际状态图并不与 图4-9 一样是直线形状的，但仍然只包含标注文本的状态序列。这个状态图虽然看起来形状是一条直线，但每个状态有指向自身的跳转，被称为自跳(self-loop)。由于自跳的存在,所以这个状态图实际上并非只有一条路径，而是有着无数条路径(自环可以无限次)。   

接下来,我们需要根据语音帧和已有的声学模型选取状态图中的一条最优路径，把各帧匹配到状态图上去,这样就得到每一帧所对应的状态了。这其实就是一个完整的语音识别过程，只不过把解码路径限制在直线形状状态图中，使得识别结果必定是参考文本。这个过程被称作对齐(Align)或强制对齐(Forced alignment)，目的是获取每一帧所对应的状态(搜索空间纵轴为状态横轴为时间)。   

需要说明的是，Kaldi 使用有限状态转录机I(Weighted Finite-State Transducer,WFST)来构建状态图，状态信息实际上在图的边上而不是在节点上。本节将状态记录在节点上，是为了遵循 HMM 的传统表示习惯，也是为了表述方便，和在 Kaldi 中的做法是相同的。

gmm-align 工具是一个已封装好的对齐工具，其内部通过调用 FasterDecoder 解码器来完成对齐。使用方法：   
gm-align [options] tree-inmodel-in lexicon-fst-in feature-rspecifier transcriptions-rspecifier alignments-wspecifier   
e.g.:gm-align tree1.mdl lex.fst scp:train.scp 'ark:sym2int.pl -f2 -words.txt text|' ark:l.ali   

工具根据输入的声学模型以及相应的 Tree 文件和 L.fst 文件。可以把声学特征和文本进行对齐。对齐后生成的 ALI 文件如果作为文本形式输出格式是:sample_1000093 4 1 1 1 16 15 15 .....   
上面的对齐结果文件由若干个句子的对齐信息构成。每个句子以句子 ID 开头，如示例中的 sample_1000093，ID 后面的整数序列是 transition-id，transition-id 概念将在4.2.5介绍。对齐结果文件按照 Kaldi 的 IntegerVector 类型存档格式保存。可以用 copy-int-vector 工具进行文本和二进制格式的互转。   

Kaldi 还提供了一个 gmm-align-compiled工 具，可以看作是 gmm-align 的简化版。gmm-align-compiled 和 gmm-align 的不同之处在于，gmm-align-compiled 使用预先构建好的对齐状态图，而 gmm-align 是在线构建对齐状态图，两者的原理是完全相同的。

在后面的模型训练中，将使用 gmm-align-compiled 对训练数据进行反复对齐。    

#### 4.2.5 Transition 模型   



-----

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Kaldi_abstract_1.md)
