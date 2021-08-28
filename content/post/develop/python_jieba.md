---
title: "Python-jieba库的使用"
description: 读书时的碎碎念
date: 2018-02-01T21:14:23+08:00
tags:
    - 开发
    - Python
    - jieba
Categories:
    - 开发
---

jieba库是一个python的中文分词库

当我们需要进行中文语义分析的时候可以用到

> import jieba

## intro.

三种模式:

```
精确(默认)模式: 将句子最精确地切开，适合文本分析;

全模式: 将句子中所有的可以成词的词语都扫描出来，但不能解决歧义;

搜索引擎模式: 在精确模式基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词。
```

它还支持繁体分词，也可以载入自定义词典。

1. 分词

   jieba.cut(‘需要分词的字符串’,cut_all=False,HMM=True)

   输入的三个参数为: 需要分词的字符串(utf8或unicode编码), cut_all参数选择是否开启全模式，HMM参数选择是否使用HMM模型。

   ```
    text = "小明硕士毕业于中国科学院计算所，后在日本京都大学深造"
   
    word_list = jieba.cut(text) #默认cut_all=False,HMM=True
    word_list2 = jieba.cut(text,cut_all=True)
    word_list3 = jieba.cut_for_search(text)
   
    print('精确模式:','/'.join(word_list))
    print('全模式:','/'.join(word_list2))
    print('搜索引擎模式:','/'.join(word_list3))
    
   ```

   需要注意的是，jieba.cut返回的是一个迭代器(生成器)generator

   这里说明一下，generator和list，tuple相比，是一种比较不消耗内存的存储方式，所以取值的方式也有点特殊。

   ```
    while True:
        try:
            print(next(generator))
        except:
            break
    
   ```

   当然，我们不需要用(‘/‘).join(word_list).split(‘/‘)这种方式来获得一个词语的list。不够pythonic。

   jieba库直接提供了返回list的cut方法，参数和上面例子中的完全相同。

   ```
    word_list = jieba.lcut(text)
    type(word_list) #
    
   ```

   其它方法和此例相同。

   

2. 载入自定义词典

   开启HMM模式以后，jieba库就有了自动识别新词的能力，但是如果有自定义词典的话，识别的准确性就能提高的更多。

   jieba.load_userdict(‘dict_path’) #载入自定义词典

   dict_path是词典的路径

   词典的格式:

   和jieba库的dict.txt一样，一个词占一行。每一行分三部分:词语，词频(可省略)，词性(可省略)，用空格隔开，顺序不可颠倒。文件需为utf8或unicode编码。

   ```
    #coding:utf8
   
    import jieba
   
    text = '穿过县界长长的隧道，便是雪国。夜空下一片白茫茫。火车在信号所前停了下来。
    一位姑娘从对面座位上站起身子，把岛村座位前的玻璃窗打开。一股冷空气卷袭进来。姑娘将身子探出窗外，仿佛向远方呼唤似地喊道：
    “站长先生，站长先生！”'
   
    word_list = jieba.lcut(text)
    print('/'.join(word_list))
   
    print('\n——————————————\n')
   
    jieba.load_userdict('mydict.txt') #载入自定义词典mydict.txt
   
    word_list2 = jieba.lcut(text)
    print('/'.join(word_list2))
   
    '''mydict.txt
    站长先生
    临时宿舍
    '''
    
   ```

3. 动态修改词典

   add_word(word,freq=None,tag=None)和del_word(word)，可以动态修改词典

   suggest_freq(segment,tune=True) 可以调节单个词语的词频，使其能/不能被分出来

   ```
    jieba.add_word('你好')
   
    jieba.suggest_freq('女神',tune=True)
    
   ```

4. 关键词抽取

   > import jieba.analyse

   a.基于TF-IDF算法
   jieba.analyse.extract_tags(sentence,topK=20,withWeight=False,allowPOS=())

   参数:
   sentence为待提取的文本，topK为返回几个TF/IDF权重最大的关键词，默认为20个，withWeight为是否一并返回关键词权重值，默认False, allow POS仅包括指定词性的词，默认值为空，即不筛选。

   b.TextRank算法
   jieba.analyse.textranke(sentence,topK=20,withWeight=False,allowPOS=(‘ns’,’n’,’vn’,’v’))

   参数同上

   TF-IDF算法:

   > http://blog.csdn.net/sangyongjia/article/details/52440063

   TextRank算法:

   > http://blog.csdn.net/kamendula/article/details/51756552