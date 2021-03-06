Iterative Self-organizing Data Analysis Techniques Algorithm

[1] J. Richards and X. Jia, Remote Sensing Digital Image Analysis. Berlin:
Springer, 1999.
[2] PCI Geomatics Corp., “ISOCLUS–Isodata clustering program,” http://
www.pcigeomatics.com/cgi-bin/pcihlp/ISOCLUS.
[3] G. H. Ball and D. J. Hall, “Some fundamental concepts and synthesis
procedures for pattern recognition preprocessors,” in Intl. Conf. on
Microwaves, Circuit Theory, and Inform. Theory, Tokyo, Japan, Sept.
1964.
[4] A. K. Jain and R. C. Dubes, Algorithms for Clustering Data. Englewood
Cliffs, NJ: Prentice Hall, 1988.
[5] J. T. Tou and R. C. Gonzalez, Pattern Recognition Principles. London:
Addison-Wesley, 1974.
[6] T. Kanungo, D. M. Mount, N. S. Netanyahu, C. Piatko, R. Silverman,
and A. Y. Wu, “An efficient k-means clustering algorithm: Analysis and
implementation,” IEEE Trans. Pattern Anal. Mach. Intell., vol. 24, pp.
881–892, 2002.
[7] J. Bentley, “Multidimensional binary search trees used for associative
searching,” Commun. ACM, vol. 18, pp. 509–517, 1975.
[8] S. P. Lloyd, “Least squares quantization in PCM,” IEEE Trans. Inform.
Theory, vol. 28, pp. 129–137, 1982.
[9] J. MacQueen, “Some methods for classification and analysis of multivariate observations,” in Proc. 5th Berkeley Symp. Math. Stat. Prob.,
vol. 1, Berkeley, CA, 1967, pp. 281–296.
[10] S. Z. Selim and M. A. Ismail, “K-means-type algorithms: A generalized
convergence theorem and characterization of local optimality,” IEEE
Trans. Patt. Anal. Mach. Intell., vol. 6, pp. 81–87, 1984.
[11] D. Pelleg and A. Moore, “Accelerating exact k-means algorithms with
geometric reasoning,” in Proc. ACM SIGKDD Intl. Conf. on Knowledge
Discovery and Data Mining, San Diego, CA, Aug. 1999, pp. 277–281.
[12] S. Phillips, “Reducing the computation time of the ISODATA and k-means unsupervised classification algorithms,” in Proc. 22nd IEEE Intl.
Geosci. and Remote Sensing Symp., Toronto, Canada, June 2002.
[13] G. H. Ball and D. J. Hall, “ISODATA, A novel method of data analysis and pattern classification,” Stanford Research Institute, Menlo Park, CA,
Tech. Rep. AD 699616, 1965

方差大未必距离中心点远，只是距平均值变化幅度大，填充聚类需要用实际距离。     

先在现有点中随便选几个点，以这几个点的位置为中心聚类     

然后按设定好的次数开始循环调整聚类中心的位置，循环次数还需要做一些实验？     

在每次循环中，调整中心点的方法是     
首先将各个数据点（以特征为维度，不限制维度数，如果上一轮分配过点，清了重分）添加到与自身的欧式距离最近的中心点所在的聚类中     
将所有只有中心的聚类取消掉，注意，此处与原本的ISOData不同，原本是按参数，小于参数的聚类取消，但我们的数据里有很稀有的离群点，虽然做数据分析可能价值不大，但是不显示出来，可能会有业务人员问，所以无所谓，不差这一两个类     
然后把现有的聚类中，各个点的每个维度分别求平均，以各个维度的平均值为坐标的位置作为当前聚类的新中心     
对各个聚类中所属的所有点与所在聚类的中心的欧式距离求平均     
再对所有聚类的该欧式距离平均值加和求一次总的平均     

此时，如果聚类的数量少于期望数量的一半，寻找所有符合分裂标准的聚类进行分裂   
分裂的标准是，每个聚类中所属所有的点的每个维度的值与平均值（应该？就是当前聚类的中心的当前分量值，再确认一下）标准差中最大的那个比初始参数设定分裂阈值大，此处还需要确认算法流程？   
分裂的方法是将待分裂的聚类的中心的上面标准差最大值的那个分量的值加上和减去该标准差值乘以一个系数，这个系数目前我用了0.5，得到的两个结果分别作为分裂后两个聚类的中心

然后（此处与原算法不同，原算法将所有待分裂的聚类分裂后，进入下一轮循环，在我的环境中目前看没有必要，测试也没受影响），判断当前执行次数是否是偶数次（我的情况，这也没必要判断了，因为都合到一起了），或者聚类的个数超过了期望参数的两倍，符合条件则对现有聚类进行合并操作     
两两判断中心间距离，将其中小于阈值的两个中心的聚类进行合并，合并的方法和之前更新中心的方法类似，取两个聚类内所属所有点的每个分量的平均值为新中心，把点都放一个里，（可以考虑判断是不是最后一轮循环，如果不是，可以不用合并所有点，因为下一轮循环会重新分配），合并完待合并的就可以进行下一轮循环了

当达到设定循环轮数的时候，或者多次循环没有变化的时候就可以结束了
