简单介绍下目前在项目中的使用情况。这里仅介绍算法训练的过程，数据清洗编码等以前应该已有介绍过，如果没有，那就当没看见这句就好了。高斯分布最早由A.棣莫弗在求二项分布的渐近公式中得到。C.F.高斯在研究测量误差时从另一个角度导出了它。P.S.拉普拉斯和高斯研究了它的性质。是一个在数学、物理及工程等领域都非常重要的概率分布，在统计学的许多方面有着重大的影响力（这句来自百度）。     

使用多元高斯主要针对未知情况的数据，未来的客户与产品数据与现有数据不能保证相似的情况。其实主要是有个别产品实在是找不到正样本，也就类似于冷启动的情形。

1.对可选的特征组合参照分布情况及业务意义进行主观选择。   

2.此算法在特征数据呈正态分布时，有更好的效果。如果所选特征的数据分布不符合正态分布，可通过取对数并调整底数或类似的方法，将数据分布调整为正态分布。类似如下图例子：【图片】
 
3.对维度进行交叉试验，去掉某个维度或加上某个维度，看训练结果在测试集或交叉验证集上是否有提升。     

4.在上一步的训练以及后续的训练中，有两个主要参数需要不断调整，最后以得到合适的模型。这两个参数是数据的平均值和协方差，平均值主要调整数据分布的中心点，通常离中心点越近，被预测为正的可能性越大；协方差主要调整数据分布区间在空间中的形态，比如某个概率分布在某个维度上的范围有多大，数据点落在某个分布区间预测为正的可能性更大或者更小。协方差变化的效果示例如下图（实际使用中维度较高，无法画图）：【图片】


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
 
5.上线   
