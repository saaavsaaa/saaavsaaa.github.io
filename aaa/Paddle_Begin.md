
  机器学习算法最主流的两大类：监督学习、无监督学习      
  
    监督学习中主要有两类问题：分类问题、回归问题      
      分类中主要：决策树 decision tree、svm 支持向量机Support Vector Machine     
      回归：线性回归LinearRegression、非线性回归：GBDT（Gradient Boosting Decision Tree，渐进梯度回归树、迭代决策树）     

    无监督学习最常见的算法是聚类以及词嵌入

-----

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
  
  **假设函数** hypothesis function:用数学的方法描述自变量x和因变量y之间的关系，比如上面假设是线性关系y = θx + 0，一个描述x，y之间关系的假设的方程。单变量线性假设函数（并不都过原点）：h<sub>θ</sub>(x) = θ<sub>0</sub> + θ<sub>1</sub>x<sub>1</sub>是其中最简单的，还可以有多元多次的情况。     
  
  **损失函数** cost function:用数学方法衡量假设函数预测结果与真实值之间的“误差”。通常数据可能不会完全契合假设函数，类似上面的例子中可能不会刚好所有样本都是3的倍数。对于假设函数，数据是已知量，另外模型中还有一些未知量，如θ<sub>0</sub>、 θ<sub>1</sub>，要衡量模型的好坏就可以用损失函数。比如均方差(MSE,Mean Squared Error，就是标准差，方差的算术平方根,能反映一个数据集的离散程度): 
  <iframe src="https://saaavsaaa.github.io/jax/t.html?a=%24%24%20J%28%5Ctheta%29%3D%5Csqrt%7B%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bi%3D1%7D%5E%7BN%7D%7B%28h_%5Ctheta%28x_i%29-y_i%29%5E2%7D%7D%20%24%24" height="100px" width="700px" frameborder="0" scrolling="no"> </iframe>
  
  因为假设的是一次函数，所以结果都在一条线上，h<sub>θ</sub>(x<sub>i</sub>)就是预测的线上的值，而实际值在坐标系中可能是离散的点y<sub>i</sub>，数据集样本之间的差值的代数平均数的平方根就是均方差，可以用来做损失函数衡量模型。由于计算机并不会解方程，所以要借助损失函数不断的迭代找到J(θ)中θ的最小值，即导数为0的点的θ值，用以找到误差最小的模型。如果有两个未知数θ<sub>0</sub>和θ<sub>1</sub>图像就是一个凹面，同样是找最低点。如果是复杂的损失函数有多个极值点的，现在还没学到，之后学了再说。

  **优化算法** gradient descent(梯度下降)算法是最常见的寻找最优模型的方法。θ<sub>i</sub>定义为:   
    <iframe src="https://saaavsaaa.github.io/jax/t.html?a=%24%24%20%5Ctheta_i%3A%3D%5Ctheta_i-%5Calpha%5Cfrac%7B%5Cdelta%7D%7B%5Cdelta%5Ctheta_i%7DJ%28%5Ctheta_i%29%20%24%24" height="100px" width="700px" frameborder="0" scrolling="no"> </iframe>    
    这个方程就是梯度下降算法中的方程，里面的J(θ<sub>i</sub>)就是假设函数，


-----

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Paddle_Begin.md)

