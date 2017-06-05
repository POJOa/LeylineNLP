# 1 实验背景
阅读以 上海师范大学 林涛 为第一作者发表的《面向软件缺陷报告的提取方法》后，本人意识到，需要丰富机器学习和数据挖掘方面的知识，并形成用它们解决问题的思维方式。    
林涛指出：使用高维特征向量的支持向量机“**很容易在训练集中取得很好的效果，但在测试集外效果不理想**”。通过查阅资料，可以发现**这一方向有新的进展**。

Google提出了基于词向量的特征提取方式（Word2Vec），并发布了基于海量新闻数据训练的词向量模型。由于高效、准确，Word2Vec在英语环境中已经成为文本分类的学术热点。目前，面向英语的Word2Vec的分支已经出现了用于消歧义的、可标注词性的词向量工具Sense2Vec。在内容分类领域，出现了使用类似模型实现降维的Image2Vec和Video2Vec等学术成果，前景广阔。     
Word2Vec在中文的应用也有不俗的效果，尤其是在特定领域的文本分类、词典扩充等方面（见附录7.3），然而词向量在中文方面的基础数据和工具仍然在建设阶段。

# 2 实验目标
1. 尝试论文中推荐的**朴素贝叶斯（Naive Bayes）分类器**
2. 尝试论文中用于对照的**支持向量机（SVM）分类器**
3. 从**改变特征向量提取方式**的角度出发探索改进方案
4. 利用以上方案进行**文本二元分类**的实验


# 3 过程简述
## 3.1 前期准备

### 3.1.1 运行环境
本次实验的系统环境为 **Python 3.6.1 , Mac OS X 10.12** , 2.3 GHz Intel Core i7 , 16 GB 1600 MHz DDR3，并采用 PyCharm 作为集成开发环境。

### 3.1.2 文本采集和分词：校园网全站
本次实验用爬虫从上海师范大学的校园网（http://www.shnu.edu.cn/) 及其公开子域采集了36万篇文章，经过初步排重，共计有效文章**约24万篇**，**作为Word2Vec词向量模型的训练数据**。

经过排重后，需要对文本进行分词，为输入特征提取工具做准备。

### 3.1.3 确定分类依据：大文科类、大理科类
为了保证数据分类的客观性，本次实验未采用手工标注数据的方式，而是将**人文社科、艺术类学院**对应子域下的文章分为大文科类，将**理工商科学院**对应子域下的文章分为大理科类。

符合以上标准的有效数据共计43483篇，其中大文科类30908篇，大理科类12575篇。在此基础上，**随机抽取80%的数据用作训练，剩余20%的数据用作测试**。

为了分析和验证林涛在《面向软件缺陷报告的提取方法》论文中提到的**支持向量机的缺陷**，本次实验另从**外校的对应学院**抓取了额外的测试数据，其中大文科类1810篇、大理科类3283篇。

数据分类的详情在附录7.2.3中有另外说明。

# 3.2 实验简述

## 3.2.1 改进方案的思路：文章所有词向量的累加平均
支持向量机需要面向文章的特征向量进行分类。若要将Word2Vec与其结合，就需要基于词向量计算文章的特征向量。
本次实验采用的方式是**对每篇文章进行分词，在词向量模型中搜索词语对应的特征向量，并对搜索结果进行累加平均**。
对于中文，可能存在更合理的、基于语义和语法的加权平均方式，但未找到相关论文。

## 3.2.2 Word2Vec的相关参数
本次实验中，Word2Vec的词向量均采用5的窗口(window)值，该值的选用需要参考语料的句子长度，因为Word2Vec所用Skip-Gram算法的特性，窗口过大会影响准确率。详情可见附录7.1.2。

**难以明确每一维的实际含义是词向量为人诟病的问题之一**。方案A中，Word2Vec选用了300维度。方案B中，Word2Vec选用了400维度。维数的选定缺乏权威资料，官方文档推荐选择数十到数百，维度大小要参考语料的数量。搜狐新闻的语料数量更大，于是采用了更大的维度。

## 3.2.3 主要方案介绍

实验中获得较好成绩的有以下四种方案

方案 | 特征提取工具 | 提取特征所用语料 | 分类器 | 备注
----|----|------|----|----
A | Word2Vec(300维) | 上师大全站文本  | 支持向量机 | 3.2.1提出的改进方案
B | Word2Vec(400维) | 搜狐新闻单月数据  | 支持向量机 | 测试词向量模型的普适性
C | Word2Vec(300维) | 校内数据训练集  | 支持向量机 | 主要与D、E对照
D | TF-IDF | 校内数据训练集  | 支持向量机 | 文本分类的主流方案
E | TF-IDF | 校内数据训练集  | 朴素贝叶斯 | 林涛论文的方案

注：因TF-IDF需要标注类别，所以不宜采用训练集以外的语料。

# 4 实验结果

校内数据测试结果：

方案 | 测试数据 | 准确率 | 召回率 | F-Score | 备注
----|----|----|------|----|----
A | 校内数据测试集 | 0.96  | 0.95 | 0.95 | 改进方案
B | 校内数据测试集 | 0.89  | 0.88 | 0.87 | 词向量模型对比　
C | 校内数据测试集 | 0.95  | 0.95 | 0.95 | 训练数据为A的子集　
D | 校内数据测试集 | 0.99  | 0.99 | 0.99 | 特征提取工具对比　
E | 校内数据测试集 | 0.89 | 0.88 | 0.87 | 分类器对比　


校外数据测试结果：

方案 | 测试数据 | 准确率 | 召回率 | F-Score | 备注
----|----|----|------|----|----
A | 外校数据 | 0.90  | 0.90 | 0.90 | 改进方案
B | 外校数据 | 0.79  | 0.71 | 0.72 | 词向量模型对比　
C | 外校数据 | 0.86  | 0.86 | 0.86 | 训练数据为A的子集　
D | 外校数据 | 0.81  | 0.75 | 0.76 | 特征提取工具对比　
E | 外校数据 | 0.80  | 0.78 | 0.76 | 分类器对比　


基于**支持向量机+TF-IDF**的**方案C**在校内数据的测试中取得了近乎精确的成绩，但在校外数据的测试中表现较差。

基于**支持向量机+校内Word2Vec模型**的**方案A**在校外数据中取得了最好的成绩，总体成绩稳定，表现更优秀。


下文列出方案A和D的详细测试结果数据，其中support是样本在分类中出现的次数。

方案A-校内数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|-------|-------|---------|-----------
大文科    | 0.94|   0.99   |   0.97   |   6123
大理科    |0.98 |   0.86  |    0.92    |  2574
平均/总量 | 0.96 |   0.95   |   0.95    |  8697

方案A-校外数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|----|----|----|-----
大文科   |   0.85   |   0.86   |   0.85   |   1810
大理科  |     0.92   |   0.91  |    0.92    |  3283
平均/总量  |     0.90   |   0.90   |   0.90    |  5093

方案D-校内数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|----|----|----|-----
大文科   |   0.99   |   1.00   |   0.99   |   6123
大理科  |     0.99   |   0.98  |    0.98    |  2574
平均/总量  |     0.99   |   0.99   |   0.99    |  8697

方案D-校外数据-结果详情：

分类 | 准确率 | 召回率 | F-Score |  Support
----|----|----|----|-----
大文科   |   0.60   |   0.91   |   0.72   |   1810
大理科  |     0.93   |   0.66  |    0.77    |  3283
平均/总量  |     0.81   |   0.75   |   0.76    |  5093


# 5 结果分析
TBA

# 6 不足和展望
和林涛的邮件讨论中，他肯定了我的实验过程，也提出了改进的建议。结合我自身的认识，在此予以列出。

## 6.1 不足的部分
### 6.1.1 知识欠缺
1. Word2Vec的Skip-Gram提出的模型是深度学习的范畴，但未将深度学习方面在分类任务中表现出色的卷积神经网络模型（CNN）作为分类器加入实验。
2. 缺乏对Word2Vec具体实现的细致认识，调参较为盲目。
3. 根据词向量计算文章的特征向量的方式有改进空间。
4. 未能就特定方向的任务（例如软件缺陷报告）对现有的开源实现进行横向对比。例如基于支持向量机的软件缺陷报告提取实现：https://www.cs.ubc.ca/cs-research/software-practices-lab/projects/summarizing-software-artifacts

### 6.1.2 资源不足
1. 限于设备算力、资源和精力，在本次实验中未能使用规模更大的、更具有现实意义的、更具有一般性和高质量的语料。
2. 处理中文语料需要分词，本次实验未采用自定义词库。

## 6.2 下一步的任务
1. 在二元分类的基础上，实现文本的多类别分类。
2. 限定某个领域，对该领域的语料进行精准的分词和挖掘，并扩充语料库、提高语料质量。
3. 将词向量输入更多的分类器，如CNN。
4. 试验用无监督的方式解决此类问题的准确性，例如K-Means聚类或LDA聚类。    

# 7 附录
## 7.1 Word2Vec衍生试验
### 7.1.1 语料的对模型的影响

对比三个词向量模型对于同一个词的不同近似词输出，相似度从左到右递减。    
这三个模型的语料分别来自上师大校园网、搜狐新闻和一万多篇程序员博客。  
  
可以**得出结论**：    
1. 如果需要分析的语料**具有领域的特性**，要尽量**在对应的语料上训练**。   
2. 如果要分析的语料**难以确定特性**，需要探索**消歧义**的方式才能提高精确度。

结果：

上海师大校园网 - 驱动：    
内生 促进改革 原动力 导向 战略     

搜狐新闻 - 驱动：    
牵引 驱动力 耦合 转变 结构 后驱      

程序员博客 - 驱动：    
显卡 强力 声卡 ATI 驱动程序     

--------------------------------- 
 
上海师大校园网 - 资源：    
教育资源 公共资源 人才资源 资源优势 网络资源 资源整合 自然资源 教学资源 客户资源 集约     

搜狐新闻 - 资源：    
资源优势 教育资源 人才资源 网络资源 矿产资源 自然资源 煤炭资源 海洋资源 水资源 地热资源     

程序员博客 - 资源：    
系统资源 缓存 CDN 存储空间 资金 外链 内存 延迟 策略 开销     



### 7.1.2 窗口的对模型的影响
在大多数情况下，5和15的window值只会导致结果的前后位置变动，但一般都是正确答案的范畴。    
然而，不合理的数值总会露出马脚。    
根据Skip-Gram模型的特性，它会以当前词为基点，**计算前后window*2个词语和该词语的关系**。    
训练时，本人将文本以篇为单位输入模型，所以并没有根据句子切分。window值太大理论上会**导致文本存在噪音**。    
通常来说，**中文的一句话有11个词是合理的，但31个词就是长句了**。    
实测下来，**窗口15的模型经常会把连续的词语当作相似度高的词输出**，而正解会被排到后面，例如：    

窗口5 - 夺得：    
摘得 勇夺 拿下 桂冠 斩获    
窗口15 - 夺得：    
摘得 桂冠 勇夺 一举 拿下    

窗口5 - 转折点：    
转折 里程碑 缩影 黄金时代 苦短   
窗口15 - 转折点：    
转折 此战 奇迹 里程碑 史上     

窗口5 - 振聋发聩：     
掷地有声 一句 刺耳 大公无私 哽咽     
窗口15 - 振聋发聩：    
流芳百世 无以 不掉 发人深省 然 不由 掷地有声    

**在维度不变的情况下，单个词需要考虑的特征越多，结果会越不精确，是说得通的。今后还需要进一步试验，在维度和window都提升的情况下，效果会如何**。     

### 7.1.3 验证Google的噱头    
#### 7.1.3.1 King + Women - Man = Queen
Google在推出新产品时往往会有一个抓眼球的噱头。    
这个噱头往往可以实现，但并没有传说的那么神奇和实用。    
虽然词向量更大的作用体现在预处理数据上，但是这次，Word2Vec也就有一个广为传播的噱头：King + Women - Man = Queen    

它是很容易复现的。    

> word_vectors.most_similar(positive=['国王','女性'],negative=['男性'])   

以上代码相当于计算"国王+女性-男性"

> [('王后', 0.582228422164917), ('王储', 0.5397378206253052), ('五世', 0.5332286357879639), ('二世', 0.5186830759048462), ('王室', 0.5071204900741577), ('叶卡捷琳娜', 0.5062432885169983)]

以上的返回值是一个list，它包含了若干(key,similarity)的tuple。    
可以看到**相似度最高的是王后**。


但值得注意的是，如果**把女性和男性换成女和男**，准度就会大大降低。    
推测有两个方面的原因。    
第一，中文的单字的精度表现往往会差一些。    
第二，女和女性的相似度由于数据量的问题所以并没有被训练到理想程度。      
   

---
#### 7.1.3.2 意义明确的例子：马云 + 企业家 - 互联网 = 王石

马云 + 企业家 - 互联网的结果是：     
**王石**、柳传志 ...

马云 + 企业家的结果是：    
创业者、**李彦宏** ...

可见这个negative项有效地排除了互联网行业的企业家，有不错的实际效果。    

意义明确的输入很可能会获得理想的结果，例如法国+伦敦-英国=巴黎，类似的例子不胜枚举。

---

#### 7.1.3.3 抽象的例子：神庙 + 工业 - 宗教 = 钢结构

一位**重视基建**的印度领导人说过："**大坝**是印度的**神庙**"，这是较高层次的抽象。        

神庙 + 工业 - 宗教的结果是：    
**钢结构、厂房、工业园**、模具、建材

虽然结果并没有出现大坝，但还是说得通的。    
  
贴出神庙和工业的相似词作为对比：    

神庙：
帕特农、宫殿、城堡、神话故事、洋房

工业：
轻工、制造业、精细化工、纺织、工程技术

神庙+工业：
古村、成吉思汗、泰姬陵、古村落、柴达木盆地

可见**每个参数都发挥了一定语义方面的作用**。    

---
#### 7.1.3.4 为什么会失败：火药桶 + 东亚 + 地区 - 欧洲 = 桥头堡


然而在抽象的例子中，**失败的占了大多数**，可能和文本的数量和预期结果判较为主观有关。

例如火药桶+东亚+地区-欧洲，预期的结果是朝鲜半岛，但给出的结果是桥头堡、信息港。再求火药桶和巴尔干的相似度，发现只有0.1。在语料内搜索，发现巴尔干火药桶这个词语组合出现频次很低。    
可见需要得到更好的结果，就需要更多高质量的语料。


### 7.1.4 扩充情感词典的试验
根据后文一些论文提到的方法，本人尝试了**用大连理工情感词库生成正面和负面情感词典**。    
实测时只输出对应词汇最相似的两个词，除了**名词方面的扩充相关词容易出现问题外，结果大多都是正确的**。但仍然需要**人工检查**生成的词汇。    
另外，也会有**身手不凡、坚持原则等词被划入负面**，但很难说它们不是负面的。在极性词典中，**坚持这类词语很容易被标注为带有“害怕”情感的词语**，在这方面，该库是更权威的。    

　

### 7.1.5 相似文本生成
最近有一篇AI方面的新闻，写“微软小冰作现代诗”。受此启发，我尝试生成相似的文本。    
生成的方式是，将模板分词的同时做好词性标注，替换特定词性的词语，再将它拼接起来。做法比较简单，效果也一般，如果不限定词性，甚至不限定相似词的字数，文章很容易变成前言不搭后语的崩坏作品。    
但可以日后进一步开发作为**写作辅助工具**。其实，已经有类似的论文发表，具体的方式是**将相似词推荐算法和输入法集成**。    

结果一例：

依据p -->  根据
坚持v -->  秉持
引领v -->  引导
抓手v -->  突破口
推进v -->  推动
完善v -->  完备
按照p -->  依照

结果：
**根据**学校学科发展规划，**秉持**学科**引导**，以教师教育学科和面向世界城市发展的特色学科专业群建设为**突破口**，**推动**学科专业布局调整，**完备**学科评价体系，**依照**“扶需、扶特、扶强”原则，大力加强重点学科建设。

原文：
依据学校学科发展规划，坚持学科引领，以教师教育学科和面向世界城市发展的特色学科专业群建设为抓手，推进学科专业布局调整，完善学科评价体系，按照“扶需、扶特、扶强”原则，大力加强重点学科建设。
 

## 7.2 数据相关
### 7.2.1 爬虫
本次实验采用了Python Scrapy作爬虫，并使用XPATH最大限度去除了Header、Footer等与正文无关的元素。但由于校园网CMS系统建设年代较远，DOM、ID和CLASS命名习惯都未遵守HTML5或Web 2.0相关的规范，实际执行起来效果一般，难以避免文本噪音。    
除了不规范DOM的问题，还遇到了严重的超链错误问题。在一些分站，每次点击分页器都会直接在当前URL后APPEND新的参数，产生一个新的URL，爬虫本质上是个递归过程，因为这类网站会导致无限递归，所以没法完全爬完。    
本次爬虫采用了两个初始入口：校园网首页和信息办的运行cms展示栏目，应该得以覆盖大部分页面。    
### 7.2.2 文本排重    
由于校园网URL的特性，例如网站样式皮肤的路径附带在链接里，所以存在大量一文多链的情况，且由于不同皮肤带来的DOM改变，爬虫往往会爬到极为相似但不完全相同的文本。    
在爬取数据后，本次实验使用了Python gensim库的docsim算法删除相似度大于99.99%的文章。

### 7.2.3 分类情况

#### 7.2.3.1 校内大理科类
商学院、生环学院、信机学院、数理学院、金融学院、建工学院、化学实验教学示范中心、资源化学教育部重点实验室

#### 7.2.3.2 校内大文科类
陶行知研究中心、人传学院、谢晋影视学院、马克思主义学院、非洲研究中心、对外汉语学院、国际与比较教育研究院、哲学学院、基础教育发展中心、美术学院、外语学院、法政学院、中国传统思想研究所

#### 7.2.3.3 外校大文科类
华东师范大学中文系、华东师范大学哲学系、华东师范大学传播学院

#### 7.2.3.4 外校大理科类
海事大学信工学院、华东理工大学化学与分子工程学院、上海大学生命科学学院

## 7.3 有关Word2Vec应用的中文论文
《基于Word Embedding语义相似度的字母缩略术语消歧》 于东 荀恩东 北京语言大学汉语国际教育技术研发中心北京语言大学信息科学学院    
《基于Word2Vec及大众健康信息源的疾病关联探测》 罗文馨 陈翀 邓思艺 北京师范大学政府管理学院    
《基于Word2Vec的微博情感新词识别与倾向判断研究》隋浩 广西大学    
《基于word2vec扩充情感词典的商品评论倾向分析》 陆峰 华南理工大学数学学院    
《基于LDA和Word2Vec的推荐算法研究》 北京邮电大学 董文   


## 7.4 未列出的实验内容
另实验了Doc2Vec和支持向量机、Word2Vec和朴素贝叶斯结合等分类方式，但效果不理想，不予列出。    
数据中的朴素贝叶斯分类器均采用伯努利模型，多项式、高斯判别分析模型表现均不如伯努利模型，不予列出。    
数据中的支持向量机分类器均采用线性核，其他核（例如rbf核）的表现几乎都不如线性核，不予列出。     
## 7.5 所用模型和算法的实现
Word2Vec采用Python gensim库的Skip-Gram实现。    
分类器、除Word2Vec外的特征提取工具、测试/训练集分类统计工具采用Python scikit-learn库的对应实现。    
分词工具是jieba，除了开启了新词发现（HMM）功能外，其余配置默认。    


