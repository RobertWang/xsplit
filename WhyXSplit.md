# 关于双数组trie树和计算文本相似度的一点说明 #

something about double array trie tree with xsplit and text similarity calculation

之前发布了一个PHP的分词扩展xsplit( http://code.google.com/p/xsplit )，只是在主页简单说了一下用法，下面再做点实用解释，不讲理论。

xsplit的分词实际上主要是基于双数组trie树的结构，算是trie树的一个变种，常用做搜索树，也是一种确定有限自动机（deterministic finite automaton）。顾名思义，双数组trie树利用数组保存状态，效率也更高一些（数组的操作效率比复杂的数据结构高）。trie树一般在NLP（natural language processing）用的较多一些。

xsplit作为一个PHP扩展，最初主要是考虑到当时的应用场景以web居多，并且采用PHP作为上层逻辑控制语言，使用上比较方便，开发效率更高一些。当然也可以作成Python，Ruby等的扩展。xsplit同时也提供了基于双数组trie树的操作接口，common prefix search和exact match search。

common prefix search的理解很简单，比如有一句话叫“中华人民共和国”，在我们的词典里有“中华”，“中华人民”，“中华人民共和国”这三个词，如果用以下函数：

```
$result=xs_search('中华人民共和国', XS_SEARCH_CP);
```

这时就会得到“中华”，“中华人民”，“中华人民共和国”这个三个词，因为他们有共同的前缀（prefix），这就是为什么叫common prefix search。

如果是exact match search

```
$result=xs_search('中华人民共和国', XS_SEARCH_EM);
```

则只会返回“中华人民共和国”这一个词，也就是所谓的精确（exact match）查找（完全匹配）。

这两个功能，在统计词频或者做一些基于双数组trie树的应用时非常有用。

另外，xsplit还加入了simhash和hamming distance的计算，这个是计算文本相似度有用到的。印象当中这个还算是比较棘手的问题，无论是聚类还是自动分类，似乎都是机器学习（machine learning）像什么SVM用到的比较多。而基于simhash的计算，原理比较简单，效率要高很多，处理大规模数据的时候比较有优势。

这些都可以参考Google的一篇论文： http://research.google.com/pubs/archive/33026.pdf

传统的hash算法，同样的数据或者同样的文本，可以得到相同的一个hash value，这对于检查文本是否完全相同比较方便。但是对于相似的文本，比如两段文字，他们只是少量的个别词语不同，如何计算相似度就比较麻烦了。而simhash的作用就是，相似的文本会有一个相似的hash value，这里的相似不是说他们的大小差不多，而是hamming distance很小。关于hamming distance，汉明距离，可以看一下百度百科的解释 http://baike.baidu.com/view/725269.html ，不难理解。

实际上，simhash在计算的时候，需要将文本tokenize（呃我现在也不知道tokenize中文里到底怎么理解好，分词？分解？）。英文一般就是按照空格分开，中文的话还是需要分词，如果按照单个汉字来计算，效果很不理想，这个我都有实验过。中文除了分词，还需要将无用的标点符号，特殊符号等等都去掉，这些对simhash的计算都有影响。另外，simhash在计算的时候，还支持每个token设定权重，比如在中文里就是给某些关键词设定较高的权重，我试了之后发现效果似乎不升反降，所以在xsplit里每一个token的权重都是相同的。这也是我为什么把simhash的相关计算函数放在一个分词扩展里的。

这个东西有什么用？举个很简单的例子，我有一个site可以让每个用户提交新闻（比如digg之类的），但是我不想用户提交重复的新闻。于是在数据库里保存的时候，每一条新闻有一个simhash的字段，当有用户提交新的news的时候就去数据库里利用simhash检查一下，如果相似度过高，就禁止提交或者是待审核状态。这样就大大减少了重复信息。

看到这里也许有的朋友会问，每次提交都去检查相似度，还需要计算hamming distance，当数量很大的时候岂不是会有效率问题。是会遇到这样的问题，如果hamming distance的阈值为3（这是实际检验有效的阈值），一个simhash是64 bit，所有小于等于3的组合大概有40000多，这样就把每次计算的probe控制在40000多次。但是如果数据规模仍然很大的话，这还不是一个理想的方法，具体怎么解决，请读一下刚才贴的那个Google的paper里的第三章吧，内容很详细，全文最后还有个比较。