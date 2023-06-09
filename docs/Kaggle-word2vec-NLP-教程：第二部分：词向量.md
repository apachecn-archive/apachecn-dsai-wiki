<!--yml
category: Kaggle
date: 2022-07-01 00:00:00
-->

# Kaggle word2vec NLP 教程：第二部分：词向量

### 代码

第二部分的教程代码[在这里](https://github.com/wendykan/DeepLearningMovies/blob/master/Word2Vec_AverageVectors.py)。

### 分布式词向量简介

本教程的这一部分将重点介绍使用 Word2Vec 算法创建分布式单词向量。 （深度学习的概述，以及其他一些教程的链接，请参阅“什么是深度学习？”页面）。

第 2 部分和第 3 部分比第 1 部分假设你更熟悉Python。我们在双核 Macbook Pro 上开发了以下代码，但是，我们还没有在 Windows 上成功运行代码。如果你是 Windows 用户并且使其正常运行，请在论坛中留言如何进行操作！更多详细信息，请参阅“配置系统”页面。

[Word2vec](https://code.google.com/p/word2vec/)，由 Google 于 2013 年发表，是一种神经网络实现，可以学习单词的[分布式表示](http://www.cs.toronto.edu/~bonner/courses/2014s/csc321/lectures/lec5.pdf)。在此之前已经提出了用于学习单词表示的其他深度或循环神经网络架构，但是这些的主要问题是训练模型所需时长间。 Word2vec 相对于其他模型学习得快。

Word2Vec 不需要标签来创建有意义的表示。这很有用，因为现实世界中的大多数数据都是未标记的。如果给网络足够的训练数据（数百亿个单词），它会产生特征极好的单词向量。具有相似含义的词出现在簇中，并且簇具有间隔，使得可以使用向量数学来再现诸如类比的一些词关系。着名的例子是，通过训练好的单词向量，“国王 - 男人 + 女人 = 女王”。

查看 [Google 的代码，文章和附带的论文](https://code.google.com/p/word2vec/)。 [此演示](https://docs.google.com/file/d/0B7XkCwpI5KDYRWRnd1RzWXQ2TWc/edit)也很有帮助。 原始代码是 C 写的，但它已被移植到其他语言，包括 Python。 我们鼓励你使用原始 C 工具，但如果你是初学程序员（我们必须手动编辑头文件来编译），请注意它不是用户友好的。

最近斯坦福大学的工作也将[深度学习应用于情感分析](http://nlp.stanford.edu/sentiment/)；他们的代码以 Java 提供。 但是，他们的方法依赖于句子解析，不能直接应用于任意长度的段落。

分布式词向量强大，可用于许多应用，尤其是单词预测和转换。 在这里，我们将尝试将它们应用于情感分析。

### 在 Python 中使用 word2vec

在 Python 中，我们将使用`gensim`包中的 word2vec 的优秀实现。 如果你还没有安装`gensim`，则需要[安装](http://radimrehurek.com/gensim/install.html)它。 [这里](http://radimrehurek.com/2014/02/word2vec-tutorial/)有一个包含 Python Word2Vec 实现的优秀教程。

虽然 Word2Vec 不像许多深度学习算法那样需要图形处理单元（GPU），但它是计算密集型的。 Google 的版本和 Python 版本都依赖于多线程（在你的计算机上并行运行多个进程以节省时间）。 为了在合理的时间内训练你的模型，你需要安装 cython（[这里是指南](http://docs.cython.org/src/quickstart/install.html)）。 Word2Vec 可在没有安装 cython 的情况下运行，但运行它需要几天而不是几分钟。

### 为训练模型做准备

现在到了细节！ 首先，我们使用`pandas`读取数据，就像我们在第 1 部分中所做的那样。与第 1 部分不同，我们现在使用`unlabeledTrain.tsv`，其中包含 50,000 个额外的评论，没有标签。 当我们在第 1 部分中构建词袋模型时，额外的未标记的训练评论没有用。 但是，由于 Word2Vec 可以从未标记的数据中学习，现在可以使用这些额外的 50,000 条评论。

```py
import pandas as pd

# 从文件读取数据
train = pd.read_csv( "labeledTrainData.tsv", header=0, 
 delimiter="\t", quoting=3 )
test = pd.read_csv( "testData.tsv", header=0, delimiter="\t", quoting=3 )
unlabeled_train = pd.read_csv( "unlabeledTrainData.tsv", header=0, 
 delimiter="\t", quoting=3 )

# 验证已读取的评论数量（总共 100,000 个）
print "Read %d labeled train reviews, %d labeled test reviews, " \
 "and %d unlabeled reviews\n" % (train["review"].size,  
 test["review"].size, unlabeled_train["review"].size )
```

我们为清理数据而编写的函数也与第 1 部分类似，尽管现在存在一些差异。 首先，为了训练 Word2Vec，最好不要删除停止词，因为算法依赖于句子的更广泛的上下文，以便产生高质量的词向量。 因此，我们将在下面的函数中，将停止词删除变成可选的。 最好不要删除数字，但我们将其留作读者的练习。

```py
# Import various modules for string cleaning
from bs4 import BeautifulSoup
import re
from nltk.corpus import stopwords

def review_to_wordlist( review, remove_stopwords=False ):
    # 将文档转换为单词序列的函数，可选地删除停止词。 返回单词列表。
    #
    # 1. 移除 HTML
    review_text = BeautifulSoup(review).get_text()
    #  
    # 2. 移除非字母
    review_text = re.sub("[^a-zA-Z]"," ", review_text)
    #
    # 3. 将单词转换为小写并将其拆分
    words = review_text.lower().split()
    #
    # 4. 可选地删除停止词（默认为 false）
    if remove_stopwords:
        stops = set(stopwords.words("english"))
        words = [w for w in words if not w in stops]
    #
    # 5. 返回单词列表
    return(words)
```

接下来，我们需要一种特定的输入格式。 Word2Vec 需要单个句子，每个句子都是一列单词。 换句话说，输入格式是列表的列表。

如何将一个段落分成句子并不简单。 自然语言中有各种各样的问题。 英语句子可能以“?”，“!”，“"”或“.”等结尾，并且间距和大写也不是可靠的标志。因此，我们将使用 NLTK 的`punkt`分词器进行句子分割。为了使用它，你需要安装 NLTK 并使用`nltk.download()`下载`punkt`的相关训练文件。

```py
# 为句子拆分下载 punkt 分词器
import nltk.data
nltk.download()   

# 加载 punkt 分词器
tokenizer = nltk.data.load('tokenizers/punkt/english.pickle')

# 定义一个函数将评论拆分为已解析的句子
def review_to_sentences( review, tokenizer, remove_stopwords=False ):
    # 将评论拆分为已解析句子的函数。
    # 返回句子列表，其中每个句子都是单词列表
    # 1. 使用 NLTK 分词器将段落拆分为句子
    raw_sentences = tokenizer.tokenize(review.strip())
    #
    # 2. 遍历每个句子
    sentences = []
    for raw_sentence in raw_sentences:
        # 如果句子为空，则跳过
        if len(raw_sentence) > 0:
            # 否则，调用 review_to_wordlist 来获取单词列表
            sentences.append( review_to_wordlist( raw_sentence, \
              remove_stopwords ))
    # 返回句子列表（每个句子都是单词列表，
    # 因此返回列表的列表）
    return sentences
```

现在我们可以应用此函数，来准备 Word2Vec 的输入数据（这将需要几分钟）：

```py
sentences = []  # 初始化空的句子列表

print "Parsing sentences from training set"
for review in train["review"]:
    sentences += review_to_sentences(review, tokenizer)

print "Parsing sentences from unlabeled set"
for review in unlabeled_train["review"]:
    sentences += review_to_sentences(review, tokenizer)
```

你可能会从`BeautifulSoup`那里得到一些关于句子中 URL 的警告。 这些都不用担心（尽管你可能需要考虑在清理文本时删除 URL）。

我们可以看一下输出，看看它与第 1 部分的不同之处：

```py
>>> # 检查我们总共有多少句子 - 应该是 850,000+ 左右
... print len(sentences)
857234

>>> print sentences[0]
[u'with', u'all', u'this', u'stuff', u'going', u'down', u'at', u'the', u'moment', u'with', u'mj', u'i', u've', u'started', u'listening', u'to', u'his', u'music', u'watching', u'the', u'odd', u'documentary', u'here', u'and', u'there', u'watched', u'the', u'wiz', u'and', u'watched', u'moonwalker', u'again']

>>> print sentences[1]
[u'maybe', u'i', u'just', u'want', u'to', u'get', u'a', u'certain', u'insight', u'into', u'this', u'guy', u'who', u'i', u'thought', u'was', u'really', u'cool', u'in', u'the', u'eighties', u'just', u'to', u'maybe', u'make', u'up', u'my', u'mind', u'whether', u'he', u'is', u'guilty', u'or', u'innocent']
```

需要注意的一个小细节是 Python 列表中`+=`和`append`之间的区别。 在许多应用中，这两者是可以互换的，但在这里它们不是。 如果要将列表列表附加到另一个列表列表，`append`仅仅附加外层列表; 你需要使用`+=`才能连接所有内层列表。

> 译者注：原文中这里的解释有误，已修改。

### 训练并保存你的模型

使用精心解析的句子列表，我们已准备好训练模型。 有许多参数选项会影响运行时间和生成的最终模型的质量。 以下算法的详细信息，请参阅 [word2vec API 文档](http://radimrehurek.com/gensim/models/word2vec.html)以及 [Google 文档](https://code.google.com/p/word2vec/)。

+   架构：架构选项是 skip-gram（默认）或 CBOW。 我们发现 skip-gram 非常慢，但产生了更好的结果。
+   训练算法：分层 softmax（默认）或负采样。 对我们来说，默认效果很好。
+   对频繁词汇进行下采样：Google 文档建议值介于`.00001`和`.001`之间。 对我们来说，接近0.001的值似乎可以提高最终模型的准确性。
+   单词向量维度：更多特征会产生更长的运行时间，并且通常（但并非总是）会产生更好的模型。 合理的值可能介于几十到几百；我们用了 300。
+   上下文/窗口大小：训练算法应考虑多少个上下文单词？ 10 似乎适用于分层 softmax（越多越好，达到一定程度）。
+   工作线程：要运行的并行进程数。 这是特定于计算机的，但 4 到 6 之间应该适用于大多数系统。
+   最小词数：这有助于将词汇量的大小限制为有意义的单词。 在所有文档中，至少没有出现这个次数的任何单词都将被忽略。 合理的值可以在 10 到 100 之间。在这种情况下，由于每个电影出现 30 次，我们将最小字数设置为 40，来避免过分重视单个电影标题。 这导致了整体词汇量大约为 15,000 个单词。 较高的值也有助于限制运行时间。

选择参数并不容易，但是一旦我们选择了参数，创建 Word2Vec 模型就很简单：

```py
# 导入内置日志记录模块并配置它，以便 Word2Vec 创建良好的输出消息
import logging
logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s',\
    level=logging.INFO)

# 设置各种参数的值
num_features = 300    # 词向量维度
min_word_count = 40   # 最小单词数
num_workers = 4       # 并行运行的线程数
context = 10          # 上下文窗口大小
downsampling = 1e-3   # 为频繁词设置下采样

# 初始化并训练模型（这需要一些时间）
from gensim.models import word2vec
print "Training model..."
model = word2vec.Word2Vec(sentences, workers=num_workers, \
            size=num_features, min_count = min_word_count, \
            window = context, sample = downsampling)

# 如果你不打算再进一步训练模型，
# 则调用 init_sims 将使模型更具内存效率。
model.init_sims(replace=True)

# 创建有意义的模型名称并保存模型以供以后使用会很有帮助。 
# 你可以稍后使用 Word2Vec.load() 加载它
model_name = "300features_40minwords_10context"
model.save(model_name)
```

在双核 Macbook Pro 上，使用 4 个工作线程来运行，花费不到 15 分钟。 但是，它会因你的计算机而异。 幸运的是，日志记录功能可以打印带有信息的消息。

如果你使用的是 Mac 或 Linux 系统，则可以使用终端内（而不是来自 Python 内部）的`top`命令，来查看你的系统是否在模型训练时成功并行化。 键入：

```
> top -o cpu 
```

在模型训练时进入终端窗口。 对于 4 个 worker，列表中的第一个进程应该是 Python，它应该显示 300-400% 的 CPU 使用率。

![](https://kaggle2.blob.core.windows.net/competitions/kaggle/3971/media/Screen%20Shot%202014-08-04%20at%202.01.40%20PM.png)

如果你的 CPU 使用率较低，则可能是你的计算机上的 cython 无法正常运行。

### 探索模型结果

恭喜你到目前为止成功通过了一切！ 让我们来看看我们在 75,000 个训练评论中创建的模型。

`doesnt_match`函数将尝试推断集合中哪个单词与其他单词最不相似：

```py
>>> model.doesnt_match("man woman child kitchen".split())
'kitchen'
```

我们的模型能够区分意义上的差异！ 它知道男人，女人和孩子彼此更相似，而不是厨房。 更多的探索表明，该模型对意义上更微妙的差异敏感，例如国家和城市之间的差异：

```py
>>> model.doesnt_match("france england germany berlin".split())
'berlin'
```

...虽然我们使用的训练集相对较小，但肯定不完美：

```py
>>> model.doesnt_match("paris berlin london austria".split())
'paris'
```

我们还可以使用`most_similar`函数来深入了解模型的单词簇：

```py
>>> model.most_similar("man")
[(u'woman', 0.6056041121482849), (u'guy', 0.4935004413127899), (u'boy', 0.48933547735214233), (u'men', 0.4632953703403473), (u'person', 0.45742249488830566), (u'lady', 0.4487500488758087), (u'himself', 0.4288588762283325), (u'girl', 0.4166809320449829), (u'his', 0.3853422999382019), (u'he', 0.38293731212615967)]

>>> model.most_similar("queen")
[(u'princess', 0.519856333732605), (u'latifah', 0.47644317150115967), (u'prince', 0.45914226770401), (u'king', 0.4466976821422577), (u'elizabeth', 0.4134873151779175), (u'antoinette', 0.41033703088760376), (u'marie', 0.4061327874660492), (u'stepmother', 0.4040161967277527), (u'belle', 0.38827288150787354), (u'lovely', 0.38668593764305115)]
```

鉴于我们特定的训练集，“Latifah”与“女王”的相似性最高，也就不足为奇了。

或者，与情感分析更相关：

```py
>>> model.most_similar("awful")
[(u'terrible', 0.6812670230865479), (u'horrible', 0.62867271900177), (u'dreadful', 0.5879652500152588), (u'laughable', 0.5469599962234497), (u'horrendous', 0.5167273283004761), (u'atrocious', 0.5115568041801453), (u'ridiculous', 0.5104714632034302), (u'abysmal', 0.5015234351158142), (u'pathetic', 0.4880446791648865), (u'embarrassing', 0.48272213339805603)]
```

因此，似乎我们有相当好的语义意义模型 - 至少和词袋一样好。 但是，我们如何才能将这些花哨的分布式单词向量用于监督学习呢？ 下一节将对此进行一次尝试。
