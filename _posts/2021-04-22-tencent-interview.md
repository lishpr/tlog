---
layout: post
title:  "Tencent Internship Interview Experience"
author: "Shori"
comments: false
tags: Career
---

### Notice: Chinese Content

Chinese characters go not very well with the word counting systems that did not take foreign languages into account when developing. Chinese is written without spaces, or is a *Scriptio continua*. Therefore, the word count designed for English tends to recognize entire sentences into words. This messes up a lot with my somewhat tidy homepage. This paragraph is a simple workaround to reach the word limit for the snippet that is showcased on my homepage.

<br>

本文最初发布于[牛客网](https://www.nowcoder.com/discuss/646742?type=post&order=create&pos=&page=1&channel=-1&source_id=search_post_nctrack)。

目前已收到入职指引，回馈一下地里。八股文问了啥忘了，那就主要介绍format和timeline吧。我个人情况就，转码硕，之前有两段二线厂实习这样。

一开始投的是后台开发岗位，最后录的是安全开发类。

<br>


## 3月16日官网投递提前批

我之前以为提前批只有大佬敢投呢，结果后来知道腾讯的提前批就没有这个区别。

<br>

## 3月19日CDG一面

电话面试。问了很多简历相关，之前的实习经历之类的。没有八股文。

反问得知CDG。一共聊了40分钟。

结束之后20分钟又来一个电话问我写过多少行代码，这我从来没算过，随口报了个5k行，面试官好像有点失望的语气的样子。

然后和同学一交流他们都说我傻了怎么可能只有5k，然后我仔细想想大呼上当。。。

<br>

## 3月21日笔试

菜狗1.2/5。不得不说腾讯笔试题和leetcode的手感差别很大。。。而且好像我记得有题我应该至少是次优的做法却过不了多少test case= =

不说了就是菜。

<br>

#### 3月22日变灰，心情很糟糕。3月24日被捞，很开心。（单纯

<br>

## 3月26日IEG一面

视频面试。先八股文，略。

问了一个智力题（[题面和解法](https://mp.weixin.qq.com/s/y_f55KgTs8W4oiSLyTMxeA)），面试官报完题面之后我笑了一下，心里知道惨了，不会，但是面试官以为是因为我做过才笑的哈哈哈。

然后不知所云了20分钟之后面试官手把手教着做完了。搞完我说，你看我是没做过吧（

接着算法题，lc14最长公共前缀。俺承认最优解是二分查找，但有点忘了，想着先写一个扫描，然后和面试官一起优化对吧（计划通）

结果写完扫描，俺觉得ok了，面试官看了一眼直接说“你有做过算法训练么？”（否定语气，感觉不是很满意这个解）。我就说大概200道吧。回说“那还可以啊”，就又给了一道lc322换硬币，秒之。

反问得知是Next Studios。

<br>

## 3月31日IEG二面

视频面试。没有八股文。先问了喜欢打啥游戏么？我：十字军之王，脑叶公司，足球经理，以撒的结合。面试官：哦。。。（然后就没接我的话茬草

然后开始扯我的简历。氛围非常轻松。

接着给了一道，让我设计一个洗牌算法。我内心：不走常规可还行。问了一下用什么语言，不限，那么就写Python节省时间。。。

我想了五分钟大概写成这样
```python
def shuffle(deck):
    for i in range(len(deck)):
        p = random.randint(0, len(deck) - 1)
        deck[p], deck[i] = deck[i], deck[p]
```
最后面试官让我写个测试，跑完测试说我不对。但是我是真的不知道为啥不对= =（面完网上搜了一下标答点点点

<br>

## 4月6日

我早上去打疫苗了，突然打疫苗的时候收到短信问我为啥之前我想的那个算法不对。。。

讲道理我面完去看过一下标答对吧，差不多意思是不能随机数取0到n，得是0到i+1，否则概率会越来越小。

我简单说了一下面试官就没回我了。下午变灰，心情很糟糕。

不过，晚上10点突然有一个0755区号的电话打我，突然很激动，问我明天能面试吗，在深圳行吗？我说好啊好啊（卑微

于是又被捞了。

<br>

## 4月7日CSIG一面

视频面试。没有八股文，全程问目前的实习内容。。。

算法题，给了我一串乱码字符串 `TWFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCB` ，让我翻译成明文。我：懵，直接和他说不会。

面试官和我说是base64解码，俺没答出来于是又给了我一道lc1204 Remove All Adjacent Duplicates in String II。stack秒。

反问得知是腾讯云。

<br>

## 4月8日CSIG二面

视频面试。先大篇幅进行八股文快速问答，略。

算法题，给一个`m*n`矩阵，你在左上格子，每次只能向右或向下走一格，求到右下格子有多少种走法。回溯法秒。

<br>

## 4月14日CSIG HR面

视频面试。

其实咱一直不知道4月14日之后没面完的都会回池子。。。是面试的HR告诉我的。。。（倍感后怕

没问behavioral，就只是问了一些基本信息和情况。

反问得知是移动安全实验室。

<br>

好了，其实到这里就算是稳了吧？但是没有踏入腾讯大厦之前永远是慌的，大概这就是人生吧。

4月14日云证，4月18日电话确认（offer call），4月20日pdf offer发到邮箱，4月21日填入职指引。

嗯，求入职指引审核早日顺利通过。谢谢大噶。