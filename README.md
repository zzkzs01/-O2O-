赛题链接：[天池新人实战赛o2o优惠券使用预测-天池大赛-阿里云天池 (aliyun.com)](https://tianchi.aliyun.com/competition/entrance/231593/introduction?spm=5176.12281973.1005.2.3dd51f546lcpM5)

代码借鉴：[xy2333/O2O: 天池大赛：O2O优惠券使用预测（排名：前1%，AUC：0.7948）（Top1：0.8116） (github.com)](https://github.com/xy2333/O2O)

推荐：推荐使用天池实验室来跑代码，在本地自己跑比较慢。



## 数据预处理

数据处理是针对已获取的数据进行清洗，使其能更好地提取出有用特征并提高模型预测准确度。常见的数据预处理包括去除重复，缺失值代替，异常值代替，将数据清理合法化。从整体数据的描述来看，赛题中的数据比较干净，几乎没有需要处理的部分。

消费是具有季节性的，七月的消费情况很难用一二月的消费数据来预测，所以需要进行数据划分，即滑窗法划分数据。

![image-20220628133633903](C:\Users\TTTime\AppData\Roaming\Typora\typora-user-images\image-20220628133633903.png)

 

 

   分为feature数据集和dataset数据集，feature有三个，分别是4.14到5.14、5.14到6.14和7.01到7.31的消费数据。而dataset也有三个，即1.01到4.13、2.01到5.14和3.15到6.30的消费数据。Feature代表的是用来限定提取特征的user_id、merchant_id、coupon_id等，用其对应的dataset即前三月的dataset来提取特征（下述），即用dataset的消费特征来对feature中的某字段（根据要提取的特征类型来）来表连接。由于七月是需要预测的消费数据，最终用feature_1和feature_2去预测，这样用5、6月份的特征去预测七月份的消费较为准确。

## 特征值提取

特征值的提取是项目中最为重要的部分之一，也是最耗时的部分。特征提取的好坏直接决定模型预测效果的上限。

题中给出的特征都是比较基础的信息，而且数量不多。为了模型能够更准确地预测，我们需要提取更多的特征。通过对基础字段的意义和生活经验分析各个字段的交互性，并在交互间提取新的特征。在本项目，主要分为五大类特征：用户特征、商家特征、用户商家交互特征、优惠券特征以及其他特征。

**User
用户购买商家类数（类数不重复）
用户距离已用消费券消费店铺的最大、最小、平均、中位距离
用户使用优惠券消费次数
用户消费次数
用户接收优惠券数目
用户接收并使用消费券间隔天数
用户接收并使用优惠券的最大、最小、平均间隔天数
用户消费总量中使用优惠券占比
用户接收总量中使用优惠券占比
new：用户浏览商品次数

**Merchant
商家卖出数目
商家核销数目
商家发放优惠券的总数量
商家已核销优惠券中距离的最小\最大\平均\中值
商家卖出总量中优惠券的核销占比
商家发放总量中优惠券的核销占比

**Coupon
消费券发放的周号\月份
消费券是否是满减类型
消费券满减的满\减
优惠券打折类型力度
每种优惠券的数目

User_Merchant
一个客户在一个商家一共买的次数
一个客户在一个商家一共收到的优惠券
一个客户在一个商家使用优惠券购买的次数
一个客户在一个商家浏览的次数
一个客户在一个商家没有使用优惠券购买的次数（暂留）
用户在某商家接收优惠券中使用的占比
用户在某商家消费中使用优惠券支付的占比
用户在某商家浏览次数中消费的占比
用户在某商家浏览中使用优惠券消费占比

Other
某用户所有领取优惠券数量
某用户领取特定优惠券数量
用户特定的优惠券领取是否是第一次\最后一次领取
一个用户某天所接收到的所有优惠券的数量
一个用户某天接收到特定优惠券的数量
用户领取某优惠券日期与上次\下次领取相同优惠券的日期间最小天数间隔，没有则 - 1

#### 特征提取的方法：

本项目中以上特征提取的方法主要使用的是加和和表连接的方法。比如提取一个用户领取的优惠券数，先将coupon_id不为空的数据封装起来，再将user_id字段复制到一个新数据框并给新增的数据框增加count列，该列全部置1。通过加和函数将对重复的user_id，在count字段上加和，即可获得到用户领取的优惠券数count。利用此方法继续提取其他需要特征，最后对多个新数据框进行一个表连接，比如对优惠券：先将需要整合到的数据集feature_x（x=1、2、3）的coupon_id复制出来，然后将按照上述加和方法提取特征的多个数据框根据coupon_id字段进行左表连接到复制的coupon_id，于是针对feature_x的优惠券特征就提取完成。使用类似方法去提取其他类型的特征。最后同是用表连接将提取各个特征类型获得的数据框进行表连接到feature中。

于是最终，三个feature数据集的特征全部提取完成，针对feature_1和feature_2计算是否核销，增加label（即核销标签）字段并值1或0。feature_3是七月要预测的数据，其label是需要模型预测出来的。

## 模型预测

   模型预测最重要的是模型的选择，二分类问题可选的模型有很多，几乎每一种模型都能支持二分类问题。这里根据网上建议，选取竞赛中比较常用其效果比较好的xgboost（也有使用gbdt的）。



**以下两个部分我做出的效果不是很好，就不呈现在代码里，大家可以借鉴其他代码学习一下：**

## 模型调参

  模型的参数的初始化一般都根据经验取值，不一定最符合当前模型的预测。手动地调节参数能够增强模型。本项目中采用的是网格搜索交叉验证法来获取最优参数。在交叉验证的基础上，根据网格搜索来进行参数的寻找，类似在所给参数范围根据步长遍历找最佳。

## 模型融合

  模型融合即使用多个模型，给不同的模型加上不同的权重，最后融合出来一个预测更加准确的复合模型。由于集多模型优点与一身，常常能取到比较好的效果。

## 得分截图：

​	这是我学习别人的代码后，按照自己理解做的，AUC=0.7530是我最高的分（不是本项目中参数跑出来的，是在天池实验室的某一次的结果。由于调参不断地修改覆盖代码，无法还原此分数对应的代码，本仓库的代码auc应该在0.73左右）

![image-20220628122203730](C:\Users\TTTime\AppData\Roaming\Typora\typora-user-images\image-20220628122203730.png)