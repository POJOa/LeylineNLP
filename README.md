# 实验背景
阅读以 上海师范大学 林涛 为第一作者发表的《面向软件缺陷报告的提取方法》后，本人意识到，需要丰富机器学习和数据挖掘方面的知识，并形成用它们解决问题的思维方式。    
林涛指出：使用高维特征向量的支持向量机**“很容易在训练集中取得很好的效果，但在测试集外效果不理想”**。通过查阅资料，可以发现近年在**这一方向有新的进展**。Google提出的基于词向量的特征提取方式（Word2Vec）由于高效、准确在英语环境中已经成为文本分类的学术热点，在中文的情感分析等领域也有不俗的效果。   

本次实验的方式是对上文提出的**朴素贝叶斯（Naive Bayes）分类器**及其用于对照的**支持向量机（SVM）分类器**进行**文本二元分类**，并探索**支持向量机和词向量特征提取方式结合**的改进方案。


# 数据获取    
本次实验的系统环境为 **Python 3.6.1 , Mac OS X 10.12** , 2.3 GHz Intel Core i7 , 16 GB 1600 MHz DDR3，并采用 PyCharm 作为集成开发环境。    

本次实验用爬虫从上海师范大学的校园网（http://www.shnu.edu.cn/) 及其公开子域采集了36万篇文章，经过初步排重，共计有效文章**约24万篇**，**作为Word2Vec词向量模型的训练数据**。    

# 实验设计  

## 数据分类
为了保证数据分类的客观性，本次实验未采用手工标注数据的方式，而是将**人文社科、艺术类学院**对应子域下的文章分为大文科类，将**理工商科学院**对应子域下的文章分为大理科类。

符合以上标准的有效数据共计43483篇，其中大文科类30908篇，大理科类12575篇。在此基础上，**随机抽取80%的数据用作训练，剩余20%的数据用作测试**。

为了分析和验证林涛在《面向软件缺陷报告的提取方法》论文中提到的**支持向量机的缺陷**，本次实验另从**外校的对应学院**抓取了额外的测试数据，其中大文科类1810篇、大理科类3283篇。

数据分类的详情在附录中有另外说明。    

## Word2Vec相关参数
本次实验中，Word2Vec的词向量均采用5的窗口(window)值，该值的选用需要参考语料的句子长度，因为Word2Vec所用Skip-Gram算法的特性，窗口过大会影响准确率。    

难以明确每一维的实际含义是词向量为人诟病的问题之一。方案A中，Word2Vec选用了300维度。方案B中，Word2Vec选用了400维度。维数的选定缺乏权威资料，官方文档推荐选择数十到数百，需要参考语料的数量。搜狐新闻的语料数量更大，于是采用了更大的维度。    

支持向量机需要面向文章的特征向量进行分类，若要将Word2Vec与其结合，就需要基于词向量计算文章的特征向量。本次实现采用的方式是**对每篇文章进行分词，在词向量模型中搜索词语对应的特征向量，并对搜索结果进行累加平均**。对于中文，可能存在更合理的、基于语义和语法的加权平均方式，但未找到相关论文。    

## 方案介绍

在以上文本分类的实验中获得较好成绩的有以下四种方案

方案 | 特征提取工具 | 提取特征所用语料 | 分类器 | 备注
----|----|------|----|----
A | Word2Vec(300维) | 上师大全站文本  | 支持向量机 | 主要方案
B | Word2Vec(400维) | 搜狐新闻单月数据  | 支持向量机 | 测试词向量模型的普适性
C | TF/IDF | 校内数据训练集  | 支持向量机 | 文本分类的主流方案
D | TF/IDF | 校内数据训练集  | 朴素贝叶斯 | 林涛论文的方案

注：鉴于TF/IDF的特点，不宜采用训练集以外的语料。




# 实验结果

基于**支持向量机+TF/IDF**的**方案C**在校内数据中取得了近乎精确的成绩，基于**支持向量机+校内Word2Vec模型**的**方案A**在校外数据中取得了最好的成绩。    


校内数据测试结果：

方案 | 测试数据 | 准确率 | 召回率 | F-Score | 备注
----|----|----|------|----|----
A | 校内数据测试集 | 0.96  | 0.95 | 0.95 | 主要方案
B  | 校内数据测试集 | 0.89  | 0.88 | 0.87 | 词向量模型对比　
C | 校内数据测试集 | 0.99  | 0.99 | 0.99 | 特征提取工具对比　
D | 校内数据测试集 | 0.89 | 0.88 | 0.87 | 分类器对比　


校外数据测试结果：

方案 | 测试数据 | 准确率 | 召回率 | F-Score | 备注
----|----|----|------|----|----
A | 外校数据 | 0.90  | 0.90 | 0.90 | 主要方案
B | 外校数据 | 0.79  | 0.71 | 0.72 | 词向量模型对比　
C | 外校数据 | 0.81  | 0.75 | 0.76 | 特征提取工具对比　
D | 外校数据 | 0.80  | 0.78 | 0.76 | 分类器对比　


注：下文support是样本在分类中出现的次数

方案A-校外数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|----|----|----|-----
大文科   |   0.85   |   0.86   |   0.85   |   1810 
大理科  |     0.92   |   0.91  |    0.92    |  3283
平均/总量  |     0.90   |   0.90   |   0.90    |  5093

方案A-校内数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|----|----|----|-----
大文科   |   0.94   |   0.99   |   0.97   |   6123 
大理科  |     0.98   |   0.86  |    0.92    |  2574
平均/总量  |     0.96   |   0.95   |   0.95    |  8697


方案C-校外数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|----|----|----|-----
大文科   |   0.56   |   0.90   |   0.69   |   1810 
大理科  |     0.92   |   0.61  |    0.73    |  3283
平均/总量  |     0.79   |   0.71   |   0.72    |  5093

方案C-校内数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|----|----|----|-----
大文科   |   0.99   |   1.00   |   0.99   |   6123 
大理科  |     0.99   |   0.98  |    0.98    |  2574
平均/总量  |     0.99   |   0.99   |   0.99    |  8697

# 结果分析


# 不足和展望
和林涛的邮件讨论中，他肯定了我的实验过程，也提出了需要改进的地方。结合我自身的认识，在此予以列出。

## 不足的部分
### 知识欠缺
1. Word2Vec的Skip-Gram提出的模型是深度学习的范畴，但未将深度学习方面在分类任务中表现出色的卷积神经网络模型（CNN）作为分类器加入实验。
2. 缺乏对Word2Vec具体实现的细致认识，调参较为盲目。
3. 根据词向量计算文章的特征向量的方式有改进空间。
4. 未能就特定方向的任务（例如软件缺陷报告）对现有的开源实现进行横向对比。例如基于支持向量机的软件缺陷报告提取实现：https://www.cs.ubc.ca/cs-research/software-practices-lab/projects/summarizing-software-artifacts

### 资源不足
1. 限于设备算力、资源和精力，在本次实验中未能使用规模更大的、更具有现实意义的、更具有一般性和高质量的语料。
2. 处理中文语料需要分词，本次实验未采用自定义词库。


# 附录

## 数据相关
### 文本排重
由于校园网URL的特性，例如网站样式皮肤的路径附带在链接里，所以存在大量一文多链的情况，且由于不同皮肤带来的DOM改变，爬虫往往会爬到极为相似但不完全相同的文本。    
在爬取数据后，本次实验使用了Python gensim库的docsim算法删除相似度大于99.99%的文章。

### 分类情况

#### 校内大理科类
商学院、生环学院、信机学院、数理学院、金融学院、建工学院、化学实验教学示范中心、资源化学教育部重点实验室

#### 校内大文科类
陶行知研究中心、人传学院、谢晋影视学院、马克思主义学院、非洲研究中心、对外汉语学院、国际与比较教育研究院、哲学学院、基础教育发展中心、美术学院、外语学院、法政学院、中国传统思想研究所

#### 外校大文科类
华东师范大学中文系、华东师范大学哲学系、华东师范大学传播学院

#### 外校大理科类
海事大学信工学院、华东理工大学化学与分子工程学院、上海大学生命科学学院

## 未列出的实验内容
另实验了Doc2Vec和支持向量机、Word2Vec和朴素贝叶斯结合等分类方式，但效果不理想，不予列出。    
数据中的朴素贝叶斯分类器均采用伯努利模型，多项式、高斯判别分析模型表现均不如伯努利模型，不予列出。    
数据中的支持向量机分类器均采用线性核，其他核（例如rbf核）的表现几乎都不如线性核，不予列出。 
## 所用模型和算法的实现   
Word2Vec采用Python gensim库的Skip-Gram实现。    
分类器、除Word2Vec外的特征提取工具、测试/训练集分类统计工具采用Python scikit-learn库的对应实现。    
分词工具是开启了新词发现（HMM）功能的jieba。

