# Metagenome 笔记

主要是分为五个步骤

![avatar](1.png)

## 1. Experimental pipeline

实验设计以及测序实验的进行，最后得到测序的结果

1. 对照组的样本可能很难获得，特别是对于来自复杂环境的样品。因为微生物含量可以在同一环境下的不同样品之间变化，这使得在小组样品之间检测统计意义和生物学意义的差异变得复杂。一般来说建议进行纵向研究，将来自同一生境的样本随时间推移纳入研究，而不是简单的横向研究，比较不同生境的样本
2. 如果样本来源于动物模型，特别是同居的啮齿动物，则应考虑动物年龄、居住环境甚至接触动物的人的性别对微生物群落的潜在影响。通过单独饲养动物来防止微生物在笼中不同动物之间传播，将来自不同实验群体的动物安置在同一个笼子里，或用来自不同供应商或具有不同遗传背景的老鼠进行重复实验，这些方法通常是可行的(尽管这可能会导致动物的行为变化)

## 2. Pre-processing

这一步中需要使用FastQC、Trimmomatic、Picard tools等工具进行测序结果的筛选工作，以提高样本的准确性，来使后续的实验结果更加的可信

下面对FastQC的结果进行示例讲解

- **FastQC总报告信息**  
  
![avatar](https://pic4.zhimg.com/b19f28a107a90f7c4f7b02510c7884ab_r.jpg)

 1. Sequence length 代表测序的长度
 2. %GC 是需要重点关注的一个指标，这个值表示的是整体序列中的GC含量，不同物种的值不一样，比如人类细胞就是42%左右。
 3. Total Sequences记录了输入文本的reads的数量

- **Per base sequence quality**
  
![avatar](https://pic4.zhimg.com/38670ee6d5f373e326e3fe3e23ba4f9b_r.jpg)

 1. 此图中的横轴是测序序列第1个碱基到第101个碱基 
 2. 纵轴是质量得分，Q = -10*log10（error P）即20表示1%的错误率，30表示0.1%
 3. 图中每1个boxplot，都是该位置的所有序列的测序质量的一个统计，上面的bar是90%分位数，下面的bar是10%分位数，箱子的中间的横线是50%分位数，箱子的上边是75%分位数，下边是25%分位数
 4. 图中蓝色的细线是各个位置的平均值的连线
 5. 一般要求此图中，所有位置的10%分位数大于20,也就是我们常说的Q20过滤(bar的下边不低于20,图中的结果87bp之后的要去掉)

- **Per tile sequence quality**
   
![avatar](https://pic3.zhimg.com/ccf9bb69e20e2a3561581d08f57ef63e_r.jpg)

 1. 横轴同样是碱基不同的位置（1-101）
 2. 纵轴是tail的编号
 3. 蓝色代表质量好，如果出现暖色则代表该tail的质量不好，在后续的实验中要把它去掉
   
- **Per sequence quality scores**
  
![avatar](https://pic4.zhimg.com/63086aca8162f21cb685d45c20e88b6f_r.jpg)

 1. 横轴是0-40，表示Q值,即质量
 2. 纵轴是每个值对应的reads数目
 3. 观察测序结果所集中的Q值，越高越好

- **Per base sequence content** 
  
  ![avatar](https://pic1.zhimg.com/80/1b78609b8c4690eddf1eb1408c26f10c_720w.png)

 1. 横轴是1 - 101 bp；纵轴是百分比
 2. 图中四条线代表A T C G在每个位置平均含量
 3. 理论上来说，A和T应该相等，G和C应该相等，但是一般测序的时候，刚开始测序仪状态不稳定，就会出现上图的情况。像这种情况，即使测序的得分很高，也需要cut开始部分的序列信息。一般像碰到这种情况，会cut前面5bp

- **Per sequence GC content**
  
  ![avatar](https://pic3.zhimg.com/5fead4df4b5c980b7519ddd536a7b196_r.jpg)

 1.  横轴是0 - 100%, 纵轴是每条序列GC含量对应的数量
 2.  蓝色的线是程序根据经验分布给出的理论值，红色是真实值，两个应该比较接近才比较好
 3.  当红色的线出现双峰，基本肯定是混入了其他物种的DNA序列(图中的结果比较好)

- **Sequence length distribution**
  
  ![avatar](https://pic1.zhimg.com/97d24fbdf9e42fc057072f7582e1532c_r.jpg)

 1. 每次测序仪测出来的长度在理论上应该是完全相等的，但是总会有一些偏差
 2. 比如此图中，101bp是主要的，但是还是有少量的100和102bp的长度，不过数量比较少，不影响后续分析
 3. 当测序的长度不同时，如果很严重，则表明测序仪在此次测序过程中产生的数据不可信 

- **Adapter content**
  
  
  ![avatar](https://pic2.zhimg.com/7563b88d741b0f97e8546325f91e9969_r.jpg)

  关于adapter、barcode、insert序列的简介以及如何去adapter[**见此**](https://blog.csdn.net/sinat_38163598/article/details/72857172)

 1. 此图衡量的是序列中两端adapter的情况
 2. 如果在当时fastqc分析的时候-a选项没有内容，则默认使用图例中的四种通用adapter序列进行统计
 3. 本例中adapter都已经去除，如果有adapter序列没有去除干净的情况，在后续分析的时候需要先使用软件进行去接头

- **Kmer content** 
   
  ![avatar](https://pic4.zhimg.com/d36385723bfbe857c1d0516c622fc05f_r.jpg)

 1. 这个图统计的是，在序列中某些特征的短序列重复出现的次数
 2. 可以看到1-8bp的时候图例中的几种短序列都出现了非常多的次数，一般来说，出现这种情况，要么是adapter没有去除干净，而又没有使用-a参数；要么就是序列本身可能重复度比较高，如建库PCR的时候出现了bias
 3. 对于这种情况，办法是可以cut掉前面的一些长度，可以试着cut 5~8bp

## 3. Sequence analysis

这部分需要依据实验目标选定“read-based”和“assenmbly-based”两方法的结合。

关于宏基因组分析方法[**见2.1-2.5**](https://zhuanlan.zhihu.com/p/106405153)，

- **Read-based (mapping)和assembly-based两种分析策略**
  
 1. assembly-based approach 受到覆盖度的制约，因为组装时低覆盖度的区域是不会进行组装的，而是被丢弃，这样低丰度的细菌的信息就被丢弃了，反映在reads利用率上，就是往往reads利用率极低，往往低于50%。关于kmer和de Bruijn的定义[**见此**](https://zhuanlan.zhihu.com/p/57177938) 
      1. assembly-based approach 特别适用于微生物组研究尤其是包含大量以前未观测到（未测序）微生物，被部分参考序列覆盖的宏基因数据。assembly-based approach 的优势在于，他们不依赖于参考基因组的使用，而其他分析方法则会缺失群落中部分新的微生物信息
      2. 宏基因组的拼接很困难，因为每个基因组的覆盖范围取决于群落中每个基因组的丰度。低丰度的基因组可以通过调低kmer恢复，但这样重复kmer的频率会增加，使得基因的正确重建有更多的困难
 2. read-based (mapping) approach 则受到reference databases的制约，因为细菌的遗传多样性很高，即便是同一个菌种，它的不同菌株，其基因组的组成也是有相对比较大的差异的，那么在mapping的时候就会出现mapping不上的问题，使得mapping效率不够高；而且只能分析reference databases中有的物种，对于reference databases未收录的新物种，是无法进行分析的。

 |         | Assembly-based analysis | Read-based analysis （'mapping') |
 | :----: | :-----: | :-----: |
 | 全面性 | 可以构建多个完整的基因组，但只适用于具有足够覆盖范围的生物体，以便组装和归类| 是否可以提供群落功能或结构的聚合图，但仅基于有效映射到引用数据库的读取部分 |
 |群落复杂性|在复杂的群落中，只有一小部分基因组可以通过组装来分解|在足够的测序深度和满意的参考数据库覆盖范围下，能否处理任意复杂的群落|
 |新颖性|能够解析没有已知序列的亲缘关系的全新生物体基因组|无法分辨近亲基因组未知的生物体|
 |计算负担|需要长时间的计算assembly，mapping和binning|能够高效地执行，从而实现大型meta分析|
 |基因组相关的代谢|即使对于新的多样性，可以通过完全组装的基因组将代谢与系统发育联系起来|通常只能解决群落的聚集代谢，并且与系统发育的联系仅能在已知的参考基因组的背景下分析|
 |人工监督|为了准确的binning和scaffolding以及错误assembly的检测，需要人工管理|通常不需要人工管理，但是参考基因组的选择可能需要人工完成|
 |与微生物基因组学的整合|组件可以输入微生物基因组管道用于分析纯培养分离物的基因组|获得的图谱不能直接放入从纯培养的分离株中提取的基因组|
 
 不过可用的微生物参考基因组正在迅速地增加，包括那些原先难以培养的细菌由于培养方法的改进，使得对其进行测序成为可能，再加上单细胞测序的途径和 metagenomic assembly的途径得到的基因组序列。现在一些类型的环境样品（如人肠道）的参考基因组的多样性已经可以满足 assembly-free taxonomic profiling 的要求。


   






## 4. Post-processing

## 5. Validation