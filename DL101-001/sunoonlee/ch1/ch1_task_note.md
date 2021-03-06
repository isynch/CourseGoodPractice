## ch1 任务笔记

### 任务1：参数个数的计算

使用 $P(w_1, w_2) = P(w_2|w_1) * P(w_1)$ 分解句子并没有减少参数个数，为什么？

[上一个版本](https://github.com/sunoonlee/DeepLearning101/commit/da9a519de2ae74d1293fcdeac40039df32c25d04#commitcomment-21200341) 经不起推敲。试重新解释如下。

#### 怎样理解「参数」

一开始不理解问题中「参数」的含义, 读了一些资料才想明白。参数([parameter](https://en.wikipedia.org/wiki/Parameter#Probability_theory)) 是概率论和统计学里的一个基本概念. 包含参数的概率分布是一个概率分布族，当参数取值确定时才是一个具体的概率分布，比如正态分布 $N(x;\mu,\sigma^2)$ 包含两个参数 $\mu$ 和 $\sigma^2$ 。

题目涉及的是离散随机变量的概率分布, 且没有像多项分布那样简洁的数学表达。为了确定这个分布，我们需要随机变量各种取值的概率，这些概率值就是参数。比如，给定词表 $V = \{x_1,x_2,...,x_N\}$，一元概率 P(w) 的参数就包括 $p(x_i), i=1,...,N$ (其实就是 [Multinoulli 分布](https://en.wikipedia.org/wiki/Categorical_distribution)). 但因为存在约束条件 $\sum_i p(x_i) = 1$ ，因此独立参数个数是 N-1 个。

#### 参数个数计算

式左端：$p(w_1 = a_i, w_2 = a_j)$ ，共 $N^2$ 种情况，扣除约束条件，有 $N^2 - 1$ 个独立参数。

式右端：
* 前面已经提到，$P(w_1)$ 的参数有  $N - 1$ 个
* 条件概率分布 $P(w_2|w_1)$ ：对于 $w_1$ 的每种取值，都存在 $N-1$ 个参数。因此，$P(w_2|w_1)$ 的总参数个数为 $(N-1) * N$

综上，式右端的参数个数 $= N-1 + (N-1) * N = N^2 - 1 =$ 式左端的参数个数`

更一般地，m 维联合分布 $P(w_1,...,w_m)$ 的参数个数为 $N^m - 1$ ，而条件分布 $P(w_m|w_1, ..., w_{m-1})$ 的参数个数为 $N^{m-1} * (N-1)$ 。为什么我们会看到有些资料里会说，这两种模型的参数个数是 $N^m$ 呢？应该是忽略低阶项的一种简化。


### 任务2：实现 N-gram 语言模型

确切地说，是 `Count based N-gram language model`

#### 读取语料并进行预处理

选择《张爱玲作品集》作为语料输入，约 250 万字。经过初步考虑，我觉得可以对语料进行这样的预处理：
* 以 `。！？` 作为断句符号。为了方便生成句首词，在分词序列中这些断句符号之后，加入自定义的句首 token。
* 忽略部分类型的符号，包括`\n`、空格、引号、书名号等。
  * 之所以忽略引号、书名号、括号等成对符号，是担心生成语句时容易出现不配对的符号，看起来会很奇怪（后来试验的确如此）。
  * 忽略引号的另一个原因是，右引号常常放在上一条提到的断句符号之后，这种情况下，右引号实际上成为句末，处理这种情况会增加复杂性。

#### 统计 ngram 出现次数及概率

统计次数的任务可看做是上周任务的扩展。一开始我想的是用 `Counter`，以 (context, word) 元组为 key；后来看到视频演示中童牧老师用了 `defaultdict(Counter)`，想想确实更好用呀。

统计完出现次数后，可以将次数除以该 context 的总次数，得到估计的条件概率 P(word|context)。这样就得到了一个 `count based N-gram language model` 啦。

#### 试着生成几个句子

生成单个词的方法是参考了 demo 代码，具体是用一个累计的概率去跟随机数 r 比较，直到这个概率大于 r 为止。

生成句子的时候，每个句子需要预先设好最初的 context，好让程序能生成句首词，然后逐步平移 context 即可。

#### 生成效果随 N 的变化

N 取 2~5 时生成的句子样例分别见 `generated_Ngram.txt` (N = 2,3,4,5)

可以看出，N=2 时句子基本是混乱的， N 变大时句子更为通顺，但 N>3 时，直接照搬来的原句也较多。这种趋势大致可概括为：从「火星文」到「鹦鹉学舌」。应该是语料规模太小，不足以支撑起一个具有泛化能力的语言模型。
