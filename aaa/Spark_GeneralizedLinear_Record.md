

-----

    // 权重，各维度系数，也就是要求的模型变量的系数
    def run(input: RDD[LabeledPoint], initialWeights: Vector): M = {
    //向量的维度
      if (numFeatures < 0) {
        numFeatures = input.map(_.features.size).first()
      }
    //优化警告，.cache()就可以
      if (input.getStorageLevel == StorageLevel.NONE) {
        logWarning("The input data is not directly cached, which may hurt performance if its"
          + " parent RDDs are also uncached.")
      }

      // 在执行优化器验证数据，SVMWithSGD中override:validators = List(DataValidators.binaryLabelValidator)
      if (validateData && !validators.forall(func => func(input))) {
        throw new SparkException("Input validation failed.")
      }

      //数据标准化后每个特征的平均值变为0(每个特征值都减掉该特征原平均值)、标准差变为1
    //withMean和withStd同时false什么都不做;withMean=true:向量中的各元素均减去它相应的均值。withStd=true:除以它们相应的方差。
    //withMean=true，只能处理稠密的向量，不能处理稀疏向量。稀疏向量由顺序和值两个并列数组组成
    //如向量(1.0,0.0,1.0,3.0)用稀疏格式表示为(4,[0,2,3],[1.0,1.0,3.0])，[0,2,3]表示[1.0,1.0,3.0]各值的索引
      val scaler = if (useFeatureScaling) { // 在模型训练之前是否执行特征缩放以减少条件数，这将显著帮助优化器更快地收敛
        new StandardScaler(withStd = true, withMean = false).fit(input.map(_.features))
      } else {
        null
      }

      val data =
        if (addIntercept) { // 是否设置了截距，也就是添加常数项，它参数就是那个θ0
        // 向量增加一列1
          if (useFeatureScaling) {
            input.map(lp => (lp.label, appendBias(scaler.transform(lp.features)))).cache()
          } else {
            input.map(lp => (lp.label, appendBias(lp.features))).cache()
          }
        } else {
          if (useFeatureScaling) {
            input.map(lp => (lp.label, scaler.transform(lp.features))).cache()
          } else {
            input.map(lp => (lp.label, lp.features))
          }
        }

      val initialWeightsWithIntercept = if (addIntercept && numOfLinearPredictor == 1) { //默认1，逻辑回归模型中更新为分类数-1
        appendBias(initialWeights)
      } else {
        /** If `分类数 > 1`, initialWeights already contains 截距项. */
        initialWeights
      }

    //SVMWithSGD小批量随机梯度下降GradientDescent.runMiniBatchSGD,override val optimizer = new GradientDescent(gradient, updater)
      val weightsWithIntercept = optimizer.optimize(data, initialWeightsWithIntercept)//凸优化

    //执行梯度下降
      val intercept = if (addIntercept && numOfLinearPredictor == 1) {
        weightsWithIntercept(weightsWithIntercept.size - 1)
      } else {
        0.0
      }
    //执行梯度下降
      var weights = if (addIntercept && numOfLinearPredictor == 1) {
        Vectors.dense(weightsWithIntercept.toArray.slice(0, weightsWithIntercept.size - 1))
      } else {
        weightsWithIntercept
      }

    // 如果是经过标准化的向量，需要把结果转换回来。如果只标准化，没有减均值，截距不变。
    //Math shows that if we only perform standardization without subtracting means, the intercept will not be changed. 
    // w_i = w_i' / v_i where w_i' is the coefficient in the scaled space, w_i is the coefficient in the original space, and v_i is the variance of the column i.
      if (useFeatureScaling) {
        if (numOfLinearPredictor == 1) {
          weights = scaler.transform(weights)
        } else {
          /**
           当“numOfLinearPredictor>1”，必须将weights转换回原始比例。注意，当'addintercept==true'时必须显式排除截距，因为它现在包含在weights中
           */
          var i = 0
          val n = weights.size / numOfLinearPredictor
          val weightsArray = weights.toArray
          while (i < numOfLinearPredictor) {
            val start = i * n
            val end = (i + 1) * n - { if (addIntercept) 1 else 0 }

            val partialWeightsArray = scaler.transform(
              Vectors.dense(weightsArray.slice(start, end))).toArray

            System.arraycopy(partialWeightsArray, 0, weightsArray, start, partialWeightsArray.length)
            i += 1
          }
          weights = Vectors.dense(weightsArray)
        }
      }

      // 性能警告
      if (input.getStorageLevel == StorageLevel.NONE) {
        logWarning("The input data was not directly cached, which may hurt performance if its"
          + " parent RDDs are also uncached.")
      }

      // 清理
      if (data.getStorageLevel != StorageLevel.NONE) {
        data.unpersist(false)
      }

      createModel(weights, intercept)
    }

-----
