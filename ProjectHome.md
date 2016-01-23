This is an implementation of MMSEG Chinese segmentation algorithm, wrapped as a PHP extension. It also provides some basic string pattern searching functions based on double-array trie tree.

### 一点历史 ###
这个project大约是09年初开始做的，很快就形成了现在版本的样子，后陆续做了一些小修改和修复了一些小bug，现整理了一下决定发布。把它open source的主要原因，就是希望这个小工具能对大家有用，并且能够参与进来一起把它做的更好

20101025


---


Change Log

20130509 0.0.9 released fixed an issue when compiler uses strict type checking


20101108 0.0.8 beta，加入xs\_simhash和xs\_hdist函数，分别计算simhash和汉明距离。


---



关于xsplit的交流，请到xsplit贴吧： http://tieba.baidu.com/f?kw=xsplit  参与讨论。

xsplit是一个PHP扩展，提供基于MMSEG算法的分词功能。目前只在linux下测试并部署过，希望有朋友可以帮忙编译提供windows下的dll。

**xsplit只处理UTF8编码格式，如果是其他编码格式，请在使用前自行转换**

xsplit主要有以下几个函数：

```
bool xs_build ( array $words, string $dict_file )

resource xs_open (string $dict_file [, bool $persistent])

array xs_split ( string $text [, int $split_method = 1 ［, resource $dictionary_identifier ］ ] )

mixed xs_search ( string $text ［, int $search_method [, resource $dictionary_identifer ] ］ )

string xs_simhash( string $text [, bool $isBinary] ) 

int xs_hdist( string $simhash1, string $simhash2 )
```

安装过程与一般的PHP扩展安装一样
```
$phpize
$./configure --with-php-config=/path/to/php-config
$make
$make install
```

php.ini中可以设置以下参数：

```
xsplit.allow_persisten = On
xsplit.max_dicts = 5
xsplit.max_persistent = 3
xsplit.default_dict_file = /home/xdict
```

xsplit.allow\_persistent 是否允许加载持久词典

xsplit.max\_dicts 允许同时打开的最大词典数目

xsplit.max\_persistent 允许同时打开的最大持久词典数目

xsplit.default\_dict\_file 默认的词典，没有指定词典时会调用此词典


源码中有一个utils目录，包含

```
make_dict.php 提供命令行方式创建词典
xsplit.php 一个简单的示例文件
xdict_example.txt 一个文本词库的格式示例
```

make\_dict.php的使用例子如下：

```
$php make_dict.php ./xdict_example.txt ./xdict.db
```

文本词库的格式请参考xdict\_example.txt

---


bool xs\_build (array $words, string $dict\_file)

从$words数组建立名称为$dict\_file的词典，若成功则返回true。$words数组的格式请参考示例，key为词语，value为词频。

例子如下：

```
<?php
$dict_file='dict.db';

$dwords['美丽']=100;
$dwords['蝴蝶']=100;
$dwords['永远']=100;
$dwords['心中']=100;
$dwords['翩翩']=100;
$dwords['飞舞']=100;
$dwords['翩翩飞舞']=10;

if(!xs_build($dwords, $dict_file)) {
    die('建立词典失败！');
}
```


resource xs\_open (string $dict\_file [, bool $persistent])

打开一个词典文件，并返回一个resource类型的identifier。$persistent可以指定是否是持久化词典，持久化词典在这里可以理解为词典资源生命周期的不同，一般情况下$persistent=true或者默认缺省即可。在进行分词的时候，可以指定不同的词典。

```
$dict_file_1 = 'xdcit.db';
$dict_file_2 = 'mydict.db';

$dict1 = xs_open($dict_file);

xs_open($dict_file); 
```


array xs\_split ( string $text [, int $split\_method = 1 ［, resource $dictionary\_identifier ］ ] )

对文本进行分词，可以指定分词方法和词典。分词方法目前有两种，一个是MMSEG算法（默认），一个是正向最大匹配，分别用常量XS\_SPLIT\_MMSEG和XS\_SPLIT\_MMFWD表示。返回值是一个数组，包含所有切分好的词语。如果不指定词典，最后一次打开的词典将被使用。

```
<?php
$text="那只美丽的蝴蝶永远在我心中翩翩飞舞着。";
$dict_file = 'xdict.db';
$dict_res = xs_open($dict_file);
$words = xs_split($text);  /* 此处没有指定词典资源，默认使用最后一次打开的词典 */

$words1 = xs_split($text, XS_SPLIT_MMSEG, $dict_res);
```


mixed xs\_search ( string $text ［, int $search\_method [, $dictionary\_identifer ] ］ )
基于双数组trie树提供的一些功能，$search\_method有四个常量表示：

XS\_SEARCH\_CP : darts的commonPrefixSearch封装，如果没有找到，返回false。

XS\_SEARCH\_EM : darts的exactMatchSearch封装，如果没有找到，返回false。

XS\_SEARCH\_ALL\_SIMPLE : 按照词典返回所有词语词频总和，一个INT型数值。

XS\_SEARCH\_ALL\_DETAIL : 按照词典返回所有词典的词频，并以数组形式返回每一个词语的详细统计。

如果不指定词典，最后一次打开的词典将被使用。

```
<?php
xs_open($dict_file);
$text="那只美丽的蝴蝶永远在我心中翩翩飞舞着。";
$word='翩翩飞舞';
$result=xs_search($word, XS_SEARCH_CP); /* common prefix search */
var_dump($result);
$result=xs_search($word, XS_SEARCH_EM); /* exact match search */
var_dump($result);
$result=xs_search($text, XS_SEARCH_ALL_SIMPLE);
var_dump($result);
$result=xs_search($text, XS_SEARCH_ALL_DETAIL);
var_dump($result);
```


string xs\_simhash( array $tokens [, bool $rawoutput] )

计算simhash。这里所有token权重都是1，$tokens的例子如array('在', '这个', '世界')。$rawoput默认为0，即返回simhash的hex string形式，如md5， sha1函数一样；如过$rawoput为真，返回一个8字节的字符串，这个字符串实际上是一个64 bits的整型数，uint64\_t，在一些特殊情况下可以用到。

int xs\_hdist（ string $simhash1, $string $simhash2)

计算汉明距离。

```
<?php
xs_open('xdict');
$text1="那只美丽的蝴蝶永远在我心中翩翩飞舞着。";
$text2="那只美丽的蝴蝶永远在我心中翩翩飞舞。";
$tokens1=xs_search($text1, XS_SEARCH_ALL_INDICT); /* 去掉标点等特殊符号，经过实验，计算simhash时，一些标点、换行、特殊符号等对效果影响较大 */
$tokens2=xs_search($text2, XS_SEARCH_ALL_INDICT);

$simhash1=xs_simhash($tokens1);
$simhash2=xs_simhash($tokens2);

echo "simhash1 is {$simhash1}\n";
echo "simhash2 is {$simhash2}\n";

$hamming_dist=xs_hdist($simhash1, $simhash2);

echo "bit-wise format:\n";
echo decbin(hexdec($simhash1)), "\n";
echo decbin(hexdec($simhash2)), "\n";

echo "hamming distance is {$hamming_dist}\n";
```