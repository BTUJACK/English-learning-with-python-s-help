# English-learning-with-python-help
**超级详细备注的代码：Python帮助您高效通过各种英语考试**

标题：限时免费|领取大学英语六级考试葵花宝典


```
# -*- coding:utf-8 -*-

#作者：公众号：湾区人工智能
#功能：实现分析各种英语历年考试真题，比如，中考，高考，四级，六级，雅思，托福，GRE,GMAT
#（pdf，Word，TXT格式数据，都能解析，不论是1个文件还是多个文件），统计单词出现次数，词频逆排序，
#从出现次数最多的单词开始记忆，提高背诵效率，做成单词红宝书，快速高效通过各种英语考试。
#时间：#2019-03-02 463710 March Saturday the 08 week, the 061 day SZ
#环境：Python3.7，编译器，sublime text，系统：Unix
#使用方法：把此代码文件和英文文件放在一个新建文件夹里，然后运行，就会产生单词红宝书。
#我把翻译部分屏蔽了，如果想要翻译，自己去后面取消屏蔽
#注意：文件夹里面不要有其他多余文件夹或者文件，要不然统计数目会有出入。但可以保证得到的单词没有重复的

```

# 几个函数的设计


### 读取PDF文件所有单词


```python

import re #正则化库

#引入解析pdf文件的库，处理pdf文件
import pdfplumber #解析pdf文件
def pdf_file():
    pdf = pdfplumber.open(filename)
    raw_words = '' #保存所有的单词
    for page in pdf.pages: 
        raw_words = raw_words + str(page.extract_words()) #两页，两个列表
    raw_words = raw_words.lower()

    words = re.sub('decimal', '', raw_words) #删除字符串里面所有的decimal,
    words = re.sub('top', '', words)
    words = re.sub('bottom', '', words)
    #words = re.sub('x', '', words)
    words = re.sub('text', '', words)
    
    words = re.findall('[a-z]+', words) #得到所有的英文单词，排除了汉语单词，各种符号
    
    while 'x' in words:
        words.remove('x')
    return words 

```
### 读取Word文件所有单词
```python

#读取docx中的单词
import docx
def docx_file():

    file=docx.Document(filename)
    raw_words = '' #保存所有的单词
    #输出每一段的内容
    for para in file.paragraphs:
        raw_words += para.text #把每段的单词添加到字符串里面
    raw_words = raw_words.lower()
    words = re.findall('[a-z]+',raw_words) #得到所有的英文单词，排除了汉语单词，各种符号
    return words 
```
### 读取TXT文件所有单词
```python

#读取TXT中的单词
def txt_file():
    with open(filename)  as file:
        raw_words = file.read().lower()
        words = re.findall('[a-z]+',raw_words) #得到所有的英文单词，排除了汉语单词，各种符号
    return words 
```
### 统计单词数目

按照词频逆排序，然后从出现个数最多的单词开始记忆，迅速提高学习效率
```python

#统计单词数目，按照词频逆排序，然后从出现个数最多的单词开始记忆，迅速提高学习效率
def count(words):
    words_set = set(words)
    number_words = len(words) #六级题总共出现的单词数目，包括重复的单词数目
    number_words_set = len(words_set) #需要记忆的单词个数
    print('需要记忆的单词个数',number_words_set)
    print('总共出现的单词数目',number_words)

    '''
    #利用函数Counter统计次数
    count_words = Counter(words)
    print(count_words)
    number_per_words = count_words.items()
    #print(number_per_words)
    '''

    #用hash计算每个单词出现次数
    hash = {}
    for word in words:
        if word not in hash:
            hash[word] = 1
        else:
            hash[word] += 1
    sorted_hash = sorted(hash,  reverse = False) #按照ABC顺序排列,没有重复单词,只有英文单词
    #sorted_hash = sorted(hash.values(), reverse = True)
    #print(sorted_hash)
    data = sorted(hash.items(), key=lambda x: x[1], reverse=True) #按照出现次数排列，没有重复单词
    return data #返回列表，每个元素是一个元组，左边是单词，右边是单词出现次数

```
### 爬取进行翻译
```python

#爬取有道翻译网页进行翻译
import urllib.request
import urllib.parse
import json
import time
#进行单词翻译
#https://blog.csdn.net/qaz3171210/article/details/51247230
def translate(word):
    content = word 
    url = 'http://fanyi.youdao.com/translate?smartresult=dict&smartresult=rule'
    data = {}
    data['i'] = content
    data['from']= 'AUTO'
    data['to'] = 'AUTO'
    data['smartresult'] = 'dict'
    data['client'] = 'fanyideskweb'
    data['salt'] = '1527480457870'
    data['sign'] = 'abf2d2301303ac6d8ae48c532c631060'
    data['doctype'] = 'json'
    data['version'] = '2.1'
    data['keyfrom'] = 'fanyi.web'
    data['action'] = 'FY_BY_REALTIME'
    data['typoResult'] = 'false'
    data = urllib.parse.urlencode(data).encode('utf-8')
    time.sleep( 6 )  #间隔几秒调取网页翻译功能，防止被屏蔽
    response = urllib.request.urlopen(url,data)
    html = response.read().decode('utf-8')
    target = json.loads(html)
    #print(target['translateResult'][0][0]['tgt'])
    return target['translateResult'][0][0]['tgt']
    #print(word,end = '              ')
    #print("翻译结果：%s" % (target['translateResult'][0][0]['tgt']))

```
# 主函数开始

#### 获取文件夹里所有文件
```python

#主函数开始
import os
file_output = '/Users/apple/Documents/ST/python/英语项目/雅思红宝书.csv'
filenames = os.listdir(os.getcwd()) #all the filenames in the current folder
all_words = []
#words = []
for filename in filenames:
```
#### 删除多余文件
```python

    #排除多余的文件，防止重复计数
    if '.csv' in filename:
        continue
    if '.py' in filename:
        continue
    print(filename)
```
####分别读取每个文件里面的单词
```python

    #统计每个pdf文件里面的单词，并且放到一个列表words里面
    if '.pdf' in filename: 
        words = pdf_file()
        print('len_words is :', len(words))

    #统计每个Word文件里面的单词，并且放到一个列表words里面
    if '.docx' in filename:
        words = docx_file()
        print('len_words is :', len(words))

    #统计每个TXT文件里面的单词，并且放到一个列表words里面
    if '.txt' in filename:
        words = txt_file()
        print('len_words is :', len(words))

```
#### 合并所有单词
```python

    #统计所有文件中的英文单词，并且放进列表里
    #all_words = all_words.extend(words)
    all_words = all_words + words 
    print('len_all_words is :', len(all_words))

```
#### 调用计数函数统计单词
```python

data = count(all_words) ##得到列表，每个元素是一个元组，左边是单词，右边是单词出现次数
word_text = ''
for w in data:
    word = w[0] #获取每个元组里的单词
    #print(chinese)
    word_text = word_text + word + '\n'#所有单词按照词频降序排列写入一个字符串里
```
#### 进行单词翻译
```python
    这里需要爬取网页翻译，耗费大量时间，可以自己选择运行
    chinese = translate(word)
    word_text = word_text + chinese + '\n'#所有单词写入一个
```
#### 生词单词红宝书
用Excel打开
```python
open(file_output,'a').write(word_text)

```
