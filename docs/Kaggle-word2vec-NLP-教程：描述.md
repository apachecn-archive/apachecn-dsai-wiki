<!--yml
category: Kaggle
date: 2022-07-01 00:00:00
-->

# Kaggle word2vec NLP 教程：描述

> 原文：[Bag of Words Meets Bags of Popcorn](https://www.kaggle.com/c/word2vec-nlp-tutorial)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)
> 
> 自豪地采用[谷歌翻译](https://translate.google.cn/)


在本教程竞赛中，我们对情感分析进行了一些“深入”研究。谷歌的 Word2Vec 是一种受深度学习启发的方法，专注于单词的含义。 Word2Vec 试图理解单词之间的意义和语义关系。它的工作方式类似于深度方法，例如循环神经网络或深度神经网络，但计算效率更高。本教程重点介绍用于情感分析的 Word2Vec。

情感分析是机器学习中的一个挑战性课题。人们用语言来表达自己的情感，这种语言经常被讽刺，二义性和文字游戏所掩盖，所有这些都会对人类和计算机产生误导。还有另一个 Kaggle 电影评论情绪分析竞赛。在本教程中，我们将探讨如何将 Word2Vec 应用于类似的问题。

在过去的几年里，深度学习在新闻中大量出现，甚至进入纽约时报的头版。这些机器学习技术受到人类大脑架构的启发，并且由于计算能力的最新进展而实现，由于图像识别，语音处理和自然语言任务的突破性结果，已经成为浪潮。最近，深度学习方法赢得了几项 Kaggle 比赛，包括药物发现任务和猫狗图像识别。

### 教程概览

本教程将帮助你开始使用 Word2Vec 进行自然语言处理。 它有两个目标：

基本自然语言处理：本教程的第 1 部分适用于初学者，涵盖了本教程后续部分所需的基本自然语言处理技术。

文本理解的深度学习：在第 2 部分和第 3 部分中，我们深入研究如何使用 Word2Vec 训练模型以及如何使用生成的单词向量进行情感分析。

由于深度学习是一个快速发展的领域，大量的工作尚未发表，或仅作为学术论文存在。 本教程的第 3 部分比说明性更具探索性 - 我们尝试了几种使用 Word2Vec 的方法，而不是为你提供使用输出的方法。

为了实现这些目标，我们依靠 IMDB 情绪分析数据集，其中包含 100,000 个多段电影评论，包括正面和负面。

### 致谢

此数据集是与以下出版物一起收集的：

> [Andrew L. Maas, Raymond E. Daly, Peter T. Pham, Dan Huang, Andrew Y. Ng, and Christopher Potts. (2011). "Learning Word Vectors for Sentiment Analysis." The 49th Annual Meeting of the Association for Computational Linguistics (ACL 2011).](http://ai.stanford.edu/~ang/papers/acl11-WordVectorsSentimentAnalysis.pdf)

如果你将数据用于任何研究应用，请发送电子邮件给该论文的作者。 该教程由 [Angela Chapman](http://www.linkedin.com/pub/angela-chapman/5/330/b97) 在 2014 年夏天在 Kaggle 实习期间开发。

### 什么是深度学习

术语“深度学习”是在2006年创造的，指的是具有多个非线性层并且可以学习特征层次结构的机器学习算法[1]。

大多数现代机器学习依赖于特征工程或某种级别的领域知识来获得良好的结果。 在深度学习系统中，情况并非如此 - 相反，算法可以自动学习特征层次结构，这些层次结构表示抽象级别增加的对象。 虽然许多深度学习算法的基本要素已存在多年，但由于计算能力的提高，计算硬件成本的下降以及机器学习研究的进步，它们目前正日益受到欢迎。

深度学习算法可以按其架构（前馈，反馈或双向）和训练协议（监督，混合或无监督）进行分类[2]。

一些好的背景材料包括：

[1] ["Deep Learning for Signal and Information Processing", by Li Deng and Dong Yu (out of Microsoft)](http://cs.tju.edu.cn/web/docs/2013-Deep%20Learning%20for%20Signal%20and%20Information%20Processing.pdf)

[2] ["Deep Learning Tutorial" (2013 Presentation by Yann LeCun and Marc'Aurelio Ranzato)](http://www.cs.nyu.edu/~yann/talks/lecun-ranzato-icml2013.pdf)

### Word2Vec 适合哪里？

Word2Vec的工作方式类似于深度方法，如循环神经网络或深度神经网络，但它实现了某些算法，例如分层 softmax，使计算效率更高。

对于 Word2Vec 以及本文的更多信息，请参阅本教程的第 2 部分，以及这篇论文：[Efficient Estimation of Word Representations in Vector Space](http://arxiv.org/pdf/1301.3781v3.pdf)

在本教程中，我们使用混合方法进行训练 - 由无监督的片段（Word2Vec）和监督学习（随机森林）组成。

### 库和包

以下列表并不是详尽无遗的。

Python 中：

Theano 提供非常底层的基本功能，用于构建深度学习系统。 你还可以在他们的网站上找到一些很好的教程。
Caffe 是 Berkeley 视觉和学习中心的深度学习框架。
Pylearn2 包装了 Theano，似乎更加用户友好。
OverFeat 用于赢得 Kaggle 猫和狗的比赛。

Lua 中：

Torch 是一个受欢迎的包，并附带一个教程。

R 中：

截至 2014 年 8 月，有一些软件包刚刚开始开发，但没有可以在教程中使用的，非常成熟的包。

其他语言也可能有很好的包，但我们还没有对它们进行过研究。

### 更多教程

O'Reilly 博客有一系列深度学习文章和教程：

[什么是深度学习，为什么要关心？](http://radar.oreilly.com/2014/07/what-is-deep-learning-and-why-should-you-care.html)
[如何构建和运行你的第一个深度学习网络](http://radar.oreilly.com/2014/07/how-to-build-and-run-your-first-deep-learning-network.html)
[网络广播：如何起步深入学习计算机视觉](http://www.oreilly.com/pub/e/3121)

还有[几个使用 Theano 的教程](http://deeplearning.net/tutorial/)。

如果你想从零开始创建神经网络，请查看 Geoffrey Hinton 的 [Coursera 课程](https://www.coursera.org/course/neuralnets)。

对于 NLP，请查看斯坦福大学最近的这个讲座：[没有魔法的 NLP 的深度学习](http://techtalks.tv/talks/deep-learning-for-nlp-without-magic-part-1/58414/)。

这本免费的在线书籍还介绍了用于深度学习的神经网络：[神经网络和深度学习](http://neuralnetworksanddeeplearning.com/)。

### 配置你的系统

如果你之前没有安装过 Python 模块，请查看[此教程](http://programminghistorian.org/lessons/installing-python-modules-pip)，提供了从终端（在 Mac / Linux 中）或命令提示符（在 Windows 中）安装模块的指南。

运行本教程需要安装以下软件包。 在大多数（或所有）情况下，我们建议你使用[`pip`](https://pypi.python.org/pypi/pip)来安装软件包。

*   [pandas](http://pandas.pydata.org/pandas-docs/stable/install.html)
*   [numpy](http://docs.scipy.org/doc/numpy/user/install.html)
*   scipy
*   [scikit-learn ](http://scikit-learn.org/stable/install.html)
*   [Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/bs4/doc/)
*   [NLTK](http://www.nltk.org/install.html)
*   [Cython](http://docs.cython.org/src/quickstart/install.html)
*   [gensim](http://radimrehurek.com/gensim/install.html)

Word2Vec 可以在`gensim`包中找到。 请注意，到目前为止，我们只在 Mac OS X 上成功运行了本教程，而不是 Windows。

如果你在 Mac Mavericks（10.9）上安装软件包时遇到问题，[本教程](http://hackercodex.com/guide/python-development-environment-on-mac-osx/)包含正确配置系统的说明。

本教程中的代码是为 Python 2.7 开发的。
