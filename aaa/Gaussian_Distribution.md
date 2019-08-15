简单介绍下目前在项目中的使用情况。这里仅介绍算法训练的过程，数据清洗编码等以前应该已有介绍过，如果没有，那就当没看见这句就好了。高斯分布最早由A.棣莫弗在求二项分布的渐近公式中得到。C.F.高斯在研究测量误差时从另一个角度导出了它。P.S.拉普拉斯和高斯研究了它的性质。是一个在数学、物理及工程等领域都非常重要的概率分布，在统计学的许多方面有着重大的影响力（这句来自百度）。     

使用多元高斯主要针对未知情况的数据，未来的客户与产品数据与现有数据不能保证相似的情况。其实主要是有个别产品实在是找不到正样本，也就类似于冷启动的情形。而没有直接用高斯分布，由于高斯分布不能挖掘出特征之间的关系，多元高斯可以自动捕捉到特征之间的关系，不用自己试特征交叉，当然成本也会高不少，一不小心就会内存溢出，需要各种调优。多元高斯使用时需要注意，样本数量需要至少是维度数量的10倍以上，原本的高斯分布是没有限制的，因为多元高斯的协方差矩阵需要可逆，矩阵可逆的条件中，需要注意的一点是如果有个别特征之间强相关，就相当于有多行或多列相等，这是要避免的。另外，协方差矩阵如果除对角线以外都是0，那么就退化成高斯分布了，这时候是挖掘不出特征间关系的。

1.对可选的特征组合参照分布情况及业务意义进行主观选择。   

2.此算法在特征数据呈正态分布时，有更好的效果。如果所选特征的数据分布不符合正态分布，可通过取对数并调整底数或类似的方法，将数据分布调整为正态分布。类似如下图例子：     
工具用得是octave,此处模拟了与我们某项数据非常相似的数据分布(x = randn(1000,1); a = x . * x)：
  
  ![Image](/ppp/gaussion/order_distribution.png)     
  
然后就是各种log，开方，加常数的常规量子力学式调参:     

  ![Image](/ppp/gaussion/log_order_distribution.png)      

  ![Image](/ppp/gaussion/01od.png)     

  ![Image](/ppp/gaussion/005od.png)    

  ![Image](/ppp/gaussion/035c65od.png)  

3.对维度进行交叉试验，找出预测错误的数据，找找看什么特征可以用于对这些预测错误的数据进行正确划分，去掉某个维度或加上某个维度，看训练结果在测试集或交叉验证集上是否有提升。某块区域样本分布的概率明显与另一块不同，但是算法并没有区分出不同时，就可以考虑再引入什么特征进行区分了。   

-----

      val gaussian = new GaussianMixture()
        .setK(2)
        .setInitialModel(initial)
      val gmm = gaussian.fit(data)

-----

4.在确定特征以及后续的训练中，有两个主要参数需要不断调整，最后以得到合适的模型。这两个参数是数据的平均值和协方差，平均值主要调整数据分布的中心点，通常离中心点越近，可能性越大；协方差主要调整数据分布区间在空间中的形态，比如某个概率分布在某个维度上的范围有多大，数据点落在某个分布区间预测为正的可能性更大或者更小。协方差变化的效果示例如下图（这图应该来自吴恩达的课程，实际使用中维度较高，难以画图）：     

![Image](/ppp/gaussion/tzt.png) 

代码类似于：

-----

     val initial = new GaussianMixtureModel(
        Array(0.5, 0.5),
        Array(
          new MultivariateGaussian(Vectors.dense(-1.0), Matrices.dense(1, 1, Array(1.0))),
          new MultivariateGaussian(Vectors.dense(1.0), Matrices.dense(1, 1, Array(1.0)))
        )
      )

-----

如果不传initialModel，spark就生成一个：

-----

      val (weights, gaussians) = initialModel match {
        case Some(gmm) => (gmm.weights, gmm.gaussians)

        case None =>
          val samples = breezeData.takeSample(withReplacement = true, k * nSamples, seed)
          (Array.fill(k)(1.0 / k), Array.tabulate(k) { i =>
            val slice = samples.view(i * nSamples, (i + 1) * nSamples)
            new MultivariateGaussian(vectorMean(slice), initCovariance(slice))
          })
      }
      
    /** Average of dense breeze vectors */
    private def vectorMean(x: IndexedSeq[BV[Double]]): BDV[Double] = {
      val v = BDV.zeros[Double](x(0).length)
      x.foreach(xi => v += xi)
      v / x.length.toDouble
    }

    /**
     * Construct matrix where diagonal entries are element-wise
     * variance of input vectors (computes biased variance)
     */
    private def initCovariance(x: IndexedSeq[BV[Double]]): BreezeMatrix[Double] = {
      val mu = vectorMean(x)
      val ss = BDV.zeros[Double](x(0).length)
      x.foreach(xi => ss += (xi - mu) ^:^ 2.0)
      diag(ss / x.length.toDouble)
    }

-----

实际使用中的部分代码：

-----

    var parsedData = sparkSession.sql(positiveSql)
    implicit val encoder: Encoder[Vector] = org.apache.spark.sql.Encoders.kryo[Vector]
    var vectorData = parsedData.map((row: Row) => {
      VectorBuilder.buildSingleVector(row)
    })

    val broadValue = 5.0
    val broadcastValue = sparkSession.sparkContext.broadcast(broadValue)
    val scaledRDD = vectorData.rdd.map(row => {
      row.asInstanceOf[DenseVector[Double]] :* broadcastValue.value
    })
    val scaledRDDByPartition = vectorData.rdd.glom().map((value) => {
      val arrayValues = value.map(denseVector => denseVector.toArray).flatten
      val denseMatrix = new DenseMatrix[Double](value.length,value(0).size,arrayValues)
      denseMatrix :*= broadcastValue.value
      denseMatrix.toDenseVector
    })

-----
 
5.模型持久化 ： gmm.write.overwrite().save(savedPath)

另：由于业务系统的特殊要求，原本spark我这个版本中混合高斯模型中的predict方法是不允许访问的，我放开了，直接调用，用业务数据凑成向量就可以直接调用了。

-----

![Image](/ppp/0.png)


