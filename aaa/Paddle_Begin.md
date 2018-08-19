  前段日子准备给客服系统换架构，不过打算等sharding-sphere 3.0的release出来再动手。在等ss发版的这段时间，就想给客服系统做个推荐，然后平时在学堂在线听课，偶然发现了百度的paddlepaddle，因为机器学习没什么经验，就打算先用它入门了。
  
-----

  机器学习算法最主流的两大类：监督学习(supervised learning)、非监督学习      
  
    监督学习中主要有两类问题：分类问题、回归问题      
      分类中主要：决策树 decision tree、svm 支持向量机Support Vector Machine     
      回归：线性回归LinearRegression、非线性回归：GBDT（Gradient Boosting Decision Tree，渐进梯度回归树、迭代决策树）     

    非监督学习最常见的算法是聚类以及词嵌入

  有监督学习中数据本身是有类别信息的，算法需要进行区分，数据本身包含标签的数据       
  无监督学习中数据没有预先标记好类别，但是数据中有一些潜在的数据结构，这些潜在的数据结构中会有一些部分各自靠的比较近，无监督学习算法就要找到这些数据结构进行分类     

-----
  
  监督学习是要根据已知数据集X和Y，寻找映射关系。对于手写字体，有两个数据集，一个是手写字图片，一个是图片对应的数字，然后用算法找出图片和数字之间的关系。一个x:feature与对应的y:label数据(x,y)构成一个样本，样本的合集构成数据集。     

  例如，有一组数据(1,3),(2,6),(3,a)，求a     
  需要根据数据集求出：y=f(x)=θx     
  假设样本是线性关系，1×θ = 3，2×θ = 6      
  可以求出θ = 3     
  于是模型就是 y = 2x，模型θ描述了y和x之间关系     
  模型的主要能力：     
  1 拟合(fit)：模型必须很好的描述已有数据间的映射关系     
  2 预测(generalization泛化)：对未知数据（没有的标记label）有预测能力 a = 9      
  
-----

  机器学习就是从数据中产生模型的过程     
  
  **假设函数** hypothesis function:用数学的方法描述自变量x和因变量y之间的关系，比如上面假设是线性关系y = θx + 0，一个描述x，y之间关系的假设的方程。单变量线性假设函数（并不都过原点）：h<sub>θ</sub>(x) = θ<sub>0</sub> + θ<sub>1</sub>x<sub>1</sub>是其中最简单的，还可以有多元多次的情况。神经网路也是一种假设函数，后面再说。     
  
  **损失函数** cost function:用数学方法衡量假设函数预测结果与真实值之间的“误差”。通常数据可能不会完全契合假设函数，类似上面的例子中可能不会刚好所有样本都是3的倍数。对于假设函数，数据是已知量，另外模型中还有一些未知量，如θ<sub>0</sub>、 θ<sub>1</sub>，要衡量模型的好坏就可以用损失函数。比如均方差(MSE,Mean Squared Error，就是标准差，方差的算术平方根,能反映一个数据集的离散程度): 
  <iframe src="https://saaavsaaa.github.io/jax/t.html?a=%24%24%20J%28%5Ctheta%29%3D%5Csqrt%7B%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bi%3D1%7D%5E%7BN%7D%7B%28h_%5Ctheta%28x_i%29-y_i%29%5E2%7D%7D%20%24%24" height="100px" width="700px" frameborder="0" scrolling="no"> </iframe>
  
  因为假设的是一次函数，所以结果都在一条线上，h<sub>θ</sub>(x<sub>i</sub>)就是预测的线上的值，而实际值在坐标系中可能是离散的点y<sub>i</sub>，数据集样本之间的差值的代数平均数的平方根就是均方差，可以用来做损失函数衡量模型。由于计算机并不会解方程，所以要借助损失函数不断的迭代找到J(θ)中θ的最小值，即导数为0的点的θ值，用以找到误差最小的模型。如果有两个未知数θ<sub>0</sub>和θ<sub>1</sub>图像就是一个凹面，同样是找最低点。如果是复杂的损失函数有多个极值点的，现在还没学到，之后学了再说，交叉熵也是。

  **优化算法** gradient descent(梯度下降)算法是最常见的寻找最优模型的方法。θ<sub>i</sub>定义为:   
 <iframe src="https://saaavsaaa.github.io/jax/t.html?a=%24%24%20%5Ctheta_i%3A%3D%5Ctheta_i-%5Calpha%5Cfrac%7B%5Cdelta%7D%7B%5Cdelta%5Ctheta_i%7DJ%28%5Ctheta_i%29%20%24%24" height="100px" width="700px" frameborder="0" scrolling="no"> </iframe>    
    
  这个方程就是梯度下降算法中的方程，里面的J(θ<sub>i</sub>)就是假设函数，α代表学习率，后面的分数乘以假设函数是这个θ对损失函数的偏导。公式就是用前一个θ减去后一个θ对损失函数偏导的结果来更新θ。（类似抛物线）如果当前θ大于最优值，它的偏导，也就是所在点的斜率是正值，新得到的θ等于原θ减去正的学习率×正的斜率，所以θ会逐渐变小;如果θ小于最优值，它的斜率就是负值，那么新θ会逐渐变大，不断接近最优值。初始值据说有一些处理方法，不过还没学到，所以先用的是随机选的一个θ值。     
  另外，这个学习率α属于超参数（需要人为设定的参数），如果设的太小，就会需要更多次的迭代，用的时间就更长；设定的太大，会导致函数无法收敛，甚至发散cost值越来越大（前一个在极值点左边，后一个就跳到右边，左右来回不断的跳，学习率越大，跳的范围越大，可能会离极值点越来越远）。选择的方法也先挖个坑。     

  三种常见主要的梯度下降的优化框架(参与损失函数运算的是我们收集来的样本（x，y）的集合):        
  **批量梯度下降**（BGD，batch gradient descent）:每次计算损失函数所有样本都参与计算，运算就是理想的按梯度下降的方向迭代，方向稳定，但是如果样本集很大，每次计算所有样本都参与运算，每次迭代幅度不能太大，但是每次迭代都会用全部的样本，运算会非常慢（迭代次数×样本数）。     
  **随机梯度下降**（SGD）:每次只用一个样本，用这个样本来改善cost，用θ逼近cost。样本集全算一遍可以迭代很多次，收敛速度快，选取样本的随机性对避开局部极值点也有好处。缺点是每次迭代不一定是向着梯度下降的方向，迭代次数也过多。可能不会收敛到最佳点，而是在极值点左右来回跳动，不过一般够用了。     
  **小批量梯度下降**（MBGD）:综合了上面两种思路，一次使用一个batch的数据，一个batch的数据一般是10到200条的样子。如果数据向量化（用矩阵方式计算替代循环）比较好，一个batch的计算速度不比一条慢，易于并行计算。
  还有一些对这三的优化算法什么的，以后再说了。

-----

  下面是paddlepaddle的例子，由于公司被封(目前还没违法，P2P清盘，大家都懂的)，目前也只跑通了例子，之后我想办法弄环境吧，家里机器太少了，数据我设置了只有公司外网IP能访问...。     
  房价预测的例子，需要以下5个步骤：     
  1.数据预处理：拿到的数据可能范围不一致，或有缺失等     
  2.设计假设函数     
  3.设计损失函数     
  4.进行训练，观察训练过程：根据梯度下降不断优化损失函数，找到对应的θ，使θ对应的误差达到最小     
  5.用模型做预测：用训练得到的参数结合假设函数构造模型，用模型进行预测      

-----

  例子是从UCI(加利福尼亚大学欧文分校University of California, Irvine，简称UC Irvine或UCI)获得的波士顿房价数据集，506行，每行14个维度，前13个是包含犯罪率等的各种比率，第14个是房价的中位数，也就是标注数据y。     
  数据里有些范围特别大，有些又特别小，所以要先进行第一步预处理。范围特别大可能会造成运算时浮点上溢或下溢，范围明显不合理的值也会对模型造成不合理的影响，范围越大影响越大，范围不同也会造成不同属性对模型的重要性不同。另外，有一些机器学习的技巧，如正则化，是基于一些假设的，例如假设所有属性的取值都差不多是以0附近的值（如1、2）为均值且取值范围相近，如果不在这个范围会影响整个训练过程和结果。可使用常用的归一化方法，把数据统一到一个范围中，比如都化为[1,-1]之间的小数（另外，归一化还有一种是把有量纲表达式变为无量纲表达式）。     
  Paddle中提供了一个dataset来放一个batch的数据集，paddle的python api中提供了reader、reader-creator、reader-decorator三种模式的组合，是数据可以复用。     

-----

  友情提示，下面的和例子代码不太一样，例子在github上paddle/book:01

  **预处理**：     
  创建一个uci housing data的reader，它首先会对数据做归一化处理统一范围，它本身是iterator一次返回一条数据:     
  reader = paddle.dataset.uci_housing.train()     
  shuffle，会先按buf_size大小从reader中读取数据，并且自动做shuffle是数据随机化，打乱数据:     
  shuffle_reader = paddle.reader.shuffle(reader, buf_size=500)     
  batch，从shuffle_reader读取batch_size大小的数据集，之后会送入训练器进行进一步的迭代训练，这里是两条数据：
  batch_reader = paddle.batch(shuffle_reader,batch_size=2)     

  **假设函数**:     
  假设13个属性和房价之间可以被线性关系描述：   
 <iframe src="https://saaavsaaa.github.io/jax/t.html?a=%24%24%20h_%5CTheta%28x%29%3D%5Csum_%7Bi%3D0%7D%5E%7Bd%7D%7B%5CTheta_iX_i%7D%3D%5CTheta_0+%5CTheta_1X_1+%5CTheta_2X_2+...+%5CTheta_dX_d%20%24%24" height="100px" width="700px" frameborder="0" scrolling="no"> </iframe>       
 
  模型的要学习的参数：θ<sub>0</sub>、θ<sub>1</sub>、θ<sub>2</sub>、...、θ<sub>d</sub> (d=13) + dias
  线性回归本质上是一个采用线性激活函数（linear activation）的全连接层(fully-connected layer):     
  输入数据，生成一个类型是dense_vector的data来描述数据，宽度13对应13个属性：     
  x = paddle.layer.data(name='x',type=paddle.data_type.dense_vector(13))     
  模型输出，模型的结果，对应了一个全连接网络(fc_layer)，激活函数使用的是线性激活函数：     
  y_predict = paddle.layer.fc(input=x,size=1,act=paddle.activation.linear())     
  
  **损失函数** 
  评估模型，这里使用均方差，线性回归模型最常见的损失函数就是使用均方误差     
  标注数据：     
  y_lable = paddle.layer.data(name='y_lable', paddle.data_type.dense_vector(1))     
  损失函数：     
  cost = paddle.layer.mse_cost(input=y_predict,lable=y_lable)    
  之后不断优化cost，找到使之最小的参数值     
  
  **线性回归之梯度下降**     
  方程momentum=0就是上面优化算法处公式的方法，这里学习率α选择0.0001，这个值不一定合理可以大一点     
  optimizer = paddle.optimizer.Momentum(momentum=0, learning_rate=0.0001)     
  构造SGD trainer，需要调整的参数parameters是需要提前构造出来的，优化参数θ的公式就用optimizer:     
  trainer = paddle.trainer.SGD(cost=cost,parameters=parameters,update_equation=optimizer)     
  开始训练，30轮，event_handler用于查看训练过程中的信息：
  trainer.train(reader=batch_reader,feeding=feeding,event_handler=event_handler,num_passes=30)     
  
  **应用模型**     
  训练完后就得到了一个模型，应用这个模型做预测     
  1.生成一些测试数据，做预测要使用一些数据做，从测试集里取5条数据放到需要预测的数据集     
&#160; test_data_creator = paddle.dataset.uci_housing.**test**()     
&#160; test_data = []     
&#160; for item in test_data_creator():     
&#160;&#160;&#160; test_data.append((item[0],))    
&#160;&#160;&#160; if len(test_data) == 5:     
&#160;&#160;&#160;&#160;&#160; break     

  这5条数据，每条有13个维度     
  2.预测(inference)
  probs = paddle.infer(ouput_layer=y_predict,parameters=paramters,input=test_data)
  for data in probs:
&#160;&#160; print data

  效果的话，可以通过做一个直角坐标系，横轴用实际价格，纵轴用对应的预测价格，如果完全贴合点会落在x=y的直线上或者在附近。

-----

  线性回归是最简单的一种机器学习算法，在日常中应用非常广泛，它有许多优点：     
    形式简单，易于建模，可以解决许多实际问题，实现也很简单，同时蕴含了机器学习的各种重要基本思想，如假设函数、损失函数...；      
    可以演变成其他复杂模型，加上一些非线性激活函数使它具备一些非线性的描述能力，可以以它为基础逐渐演变出许多非常强大的非线性模型（unlinear mode）       可解释性好，可以根据模型清楚的了解它的一些属性的含义和重要性，比如某个属性的参数是0,就可以认为这个属性对模型是没用的

  术语对应：数据集操作 paddle.dataset/paddle.reader;全连接层 paddle.layer.fc();线性激活函数 paddle.activation.linear();均方差损失函数 paddle.layer.mse_cost();参数优化器 paddle.optimizer.Momentum;梯度下降训练器 paddle.trainer.SGD。

-----

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Paddle_Begin.md)

