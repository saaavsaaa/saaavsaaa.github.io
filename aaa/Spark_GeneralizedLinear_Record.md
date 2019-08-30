这是本来打算看一下有没有办法在Spark的支持向量机上搞核函数时看代码的记录。看上去，实在想实现的话，Spark现有的支持向量机代码基本也是用不上的。准备用TensorFlow搞一下看看。

有没有核函数看上去区别只是在一个直接代入向量，一个先计算了相似度代入的是相似度的向量。由点和已有数据点的相似度作为特征，也就是有多少点就有多少特征组成一个样本，交叉，Spark无法支持如此大的单行。

训练模型调用的是SVMWithSGD的run，这个run方法就是GeneralizedLinearAlgorithm的run:

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

Spark支持向量机用的是HingeGradient做梯度下降：

-----

      private val gradient = new HingeGradient()
      private val updater = new SquaredL2Updater() // R(w) = 1/2 ||w||^2  w' = w - thisIterStepSize * (gradient + regParam * w) or w' = (1 - thisIterStepSize * regParam) * w - thisIterStepSize * gradient
      @Since("0.8.0")
      override val optimizer = new GradientDescent(gradient, updater)
        .setStepSize(stepSize)
        .setNumIterations(numIterations)
        .setRegParam(regParam)
        .setMiniBatchFraction(miniBatchFraction)
      override protected val validators = List(DataValidators.binaryLabelValidator)
  
-----
  
  HingeGradient，就是一些计算，实际上Spark支持向量机的部分看上去就这点，其他的大多是线性算法通用的
  
-----
  
          /**
         * 在支持向量机二分类场景中，计算Hinge损失函数的梯度和损失。详细公式也可以看文档
         * 注意：假设标签只有{0,1}
         */
        @DeveloperApi
        class HingeGradient extends Gradient {
          override def compute(data: Vector, label: Double, weights: Vector): (Vector, Double) = {
            val dotProduct = dot(data, weights) //点积
            // Our loss function with {0, 1} labels is max(0, 1 - (2y - 1) (f_w(x)))
            // Therefore the gradient is -(2y - 1)*x
            val labelScaled = 2 * label - 1.0
            if (1.0 > labelScaled * dotProduct) {
              val gradient = data.copy
              scal(-labelScaled, gradient) //x (gradient) = a(-labelScaled) * x 缩放，增量为1:dscal(x.values.length, a, x.values, 1)
              (gradient, 1.0 - labelScaled * dotProduct)
            } else {
              (Vectors.sparse(weights.size, Array.empty, Array.empty), 0.0)
            }
          }

          override def compute(
              data: Vector,
              label: Double,
              weights: Vector,
              cumGradient: Vector): Double = {
            val dotProduct = dot(data, weights)
            // Our loss function with {0, 1} labels is max(0, 1 - (2y - 1) (f_w(x)))
            // Therefore the gradient is -(2y - 1)*x
            val labelScaled = 2 * label - 1.0
            if (1.0 > labelScaled * dotProduct) {
              axpy(-labelScaled, data, cumGradient) //y(cumGradient) += a(-labelScaled) * x(cumGradient)
              1.0 - labelScaled * dotProduct
            } else {
              0.0
            }
          }
        }

-----

预测就是向量内积加截距，结果和设定的置信度比较，默认置信度是0.0

-----

      override protected def predictPoint(
          dataMatrix: Vector,
          weightMatrix: Vector,
          intercept: Double) = {
        val margin = weightMatrix.asBreeze.dot(dataMatrix.asBreeze) + intercept
        threshold match {
          case Some(t) => if (margin > t) 1.0 else 0.0
          case None => margin
        }
      }

-----

微信公众号：

![Image](/ppp/0.png)
