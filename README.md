# A_Simple_QASystem
一个基于检索式的简易的问答系统，基于最经典的方法也是最有效的方法。
检索式的问答系统
问答系统所需要的数据已经提供，对于每一个问题都可以找得到相应的答案，所以可以理解为每一个样本数据是 <问题、答案>。 那系统的核心是当用户输入一个问题的时候，首先要找到跟这个问题最相近的已经存储在库里的问题，然后直接返回相应的答案即可（但实际上也可以抽取其中的实体或者关键词)。 举一个简单的例子：

假设我们的库里面已有存在以下几个<问题,答案>：

<"阿里巴巴主要做什么方面的业务？”， “他们主要做电商方面”>
<"人工智能和机器学习的关系什么？", "其实机器学习是人工智能的一个范畴，很多人工智能的应用要基于机器学习的技术">
<"人工智能最核心的语言是什么？"， ”Python“>
.....
假设一个用户往系统中输入了问题 “alibaba是做什么的？”， 那这时候系统先去匹配最相近的“已经存在库里的”问题。 那在这里很显然是 “alibaba是做什么的”和“alibaba主要做什么方面的业务？”是最相近的。 所以当我们定位到这个问题之后，直接返回它的答案 “他们主要做人工智能方面的教育”就可以了。 所以这里的核心问题可以归结为计算两个问句（query）之间的相似度。

noise channel model详解:
拼写错误类型
Non-word errors: 单词不存在于字典之中，即完全写错了。例如 teh → the
Real-word errors: 单词在字典中，但写成其他单词了，与想表达的不符。具体来说，又分为两种错误：
typographical errors: three → there
cognitive errors： too → two， your → you're
Noisy channel model for non-word spelling correction
假设 a = {a-z, A-Z, ...} 是英语所有可能构成单词的字母集合， a* 为由这个字母表所构成的任意有限长度的字符串集合。 其中所有有效的单词构成的集合D是a*的一个子集。而noise channel 是指从目的词（即字典）与实际接收到的字符串x所构成的矩阵。
对于所捕获到的，存在拼写错误的字符串x, 目标是在字典中找到一个词w，使这一情况出现的概率最大。 即：



由于用户实际想拼写的单词是不确定的，因此需要生成一个修正候选列表（candidate corrections），这个列表基于两个规则：

shortest weighted edit distance
highest noisy channel probability
因此， noisy channel 实际上可以理解为，用户所输入的一个错误的字符串，经过怎样的变换过程可以得到若干个正确的单词。变换的过程越多，相当于channel越长， 而找候选列表的过程也就是找channel最短的过程。

最小编辑距离
最小编辑距离（minimum edit distance）是指从一个string到另一个string所需的最小编辑步骤，包括：插入、删除、替换。而采用这三种编辑手段计算所得的距离又称为Levenshtein distance。这一距离将所有操作的cost都记为1.

但严格来说，替换这一操作等于先删除再插入。因此这一操作的cost可当成是2（更接近实际操作的cost）。 此外，对于Damerau–Levenshtein distance，这一距离还新增了transposition of two adjacent characters 这一操作。

如何计算两个单词的edit distance
单词长度对齐→例如在单词头部＋#作为占位符

以对齐后的单词长度n建立一个n×n的矩阵。其中(n, 0)为坐标原点，横纵坐标分别对应两个单词各个字母的位置。

按照以下方法遍历矩阵n，计算edit distance


computing distance

举个例子，计算单词intention → execution的 edit 距离如下：


example
从矩阵右上角沿着对角线上数值最小的路径回退，走过的路径即为应执行的edit操作。

若路径上当前位置与后一步的数值相同→说明一致不需修改
若数值不同：
如果是落在对角线上→当前位置的字符执行替换操作
如果是落在对角线左侧→ 执行插入操作
如果是落在对角线右侧 → 执行删除操作


editing path
候选词列表的产生
80%的错误可以在1个edit 操作内纠正
几乎所有的错误都可以在2个 edit 操作内纠正
由于这是个NP问题，计算量很大，因此只需要计算考虑edit 操作数目 为 2 的单词即可。(intention → execution需要5步操作，因此没必要考虑这种情况。)
此外，非字母符号也应该在纠正过程中考虑在内， 例如goodmorning → good morning， inlaw → in-law。
同时，也应该计算不同错误出现的概率问题。 例如， P(the | teh) → high， P(the | tze) → low
而这一点的实现也是基于统计的，也即需要从样本中统计不同错误出现的概率。
channel model probability
统计概率的计算方法如下：
首先对错误统计的方式：

del[x, y] : count(xy typed as x)
ins[x,y]: count(x typed as xy)
sub[x,y]: count(x typed as y)
trans[x,y]: count(xy typed as yx)
然后，基于一个很大的语料库，统计不同单词出现的概率以及用这个单词时出现不同拼写错误的概率。

其中，w是正确的单词， x 表示捕捉到的错误字符串。 wi 是指w中的第 i 个字符。
以一个错误的输入 acress 为例， 可能的单词与其对应的概率如下：

显然，用户想输入across的概率最大，这样候选词列表就有了排序和过滤的依据（大概率的排在前面，概率过低的可以不显示）。另一方面，P(word) 也可以使用bigram，这样就与上下文取得了联系，能更好的预测用户想要输入的单词。

Noisy channel model for real-word spelling correction
有25%-40%的错误属于 real-word error
这一部分是language model与noisy channel model的结合。假设用户输入的所有单词都没有non-word error

对于输入的每个单词，无论拼写是否正确，依然生成若干个候选列表
当输入多个单词后， 利用LM， 例如trigram等，判断哪种组合的概率更高
只考虑有一个词需要变更的情况。
举个例子，用户输入 "two of thew"：

example
仅考虑 two off thew, two of the, too of thew 的概率，取最大值。

其他可用于spelling correction 的因素
根据单词发音， 优先考虑同音异义词的错误情况
例如：bigger then → bigger than， a peace of paper → a piece of paper
考虑不同键盘布局下最容易错误输入的字母概率。
例如，对于 QWERT 布局的键盘，e 和 w 这两个字母输入时更容易出错 threw → three
