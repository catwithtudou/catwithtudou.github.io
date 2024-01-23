---
title: "基于 Go 实现的推文自动分割脚本"
date: 2024-01-24T02:11:00+08:00
draft: false
tags: [ "project","Go","脚本" ]
categories: [ "project" ]
slug: /project/common/0
---

> 此文档内容为飞书文档复制过来作为搜索，存在内容格式不兼容情况，建议看原[飞书文档](https://jih9axn4gg.feishu.cn/wiki/QjmDwJ87QiGgNDkWoxzcaYTZnAb?from=from_copylink)


## 背景

作为时常在推特发布推文的免费用户，经常会遇到在想要发布稍微长一点的内容的时候，就会碰到推特推文的限制。

此时你就需要手动对内容进行切割，并且为了让切割后的推文内容能更加清晰，通常会在前面增加类似“(x/x)”的前缀来说明推文切割内容的进度。如果在编辑的过程中，你又想要增加一些新的内容，就容易遇到重新调整推文分割的情况。所以在遇到这个场景越来越多的时候，我就想着做一个针对该场景的脚本来帮助我自动做这件事情。

> 这篇文章会简单描述下在写该脚本的一些过程和设计，虽然脚本实现难度并不大，但也主要是想作为输出者，对后续自己所开发的项目，通过输出博客，来进行项目的反思和沉淀，同时也希望能帮助到有类似需求的人。

**[repo地址：https://github.com/catwithtudou/x_tool](https://github.com/catwithtudou/x_tool)**

## 需求功能

一句话描述功能：输入一段排版后的文字，若超过推特推文的长度限制则自动进行切割，并增加切割进度前缀信息。

这里需要注意的是：

- 在切割的过程中需要满足，不能调整原文的排版内容，且尽可能多地填充一次推文的内容
- 需要支持中文相关字符的处理，切割后避免出现因中文字符长度特殊而导致的乱码

## 关键设计

### 推文长度限制

首先就需要了解目前推特推文的长度限制，从而方便后续的字符长度计算：

- 长度限制为280个字符，不包括链接和图片
- 对于中文字符是按两个字符来计算，其余字符（包含标点符号、空格、换行等）是按一个字符来计算

### Go 中的字符长度计算

这里我们需要了解一些 Go 中的前置信息：

- 字符串默认都是以 UTF-8 编码保存的，其中每个中文占用 3 个字节
- 可通过 rune 数据类型数组来保存字符串，其中 rune 类型即 Unicode 编码，而每个中文则占用一个长度

所以根据以上信息我们可以通过以下代码计算出字符串在推特推文中的字符长度：

```Go
func calculateTwitterContentLen(content string) int {
    // utf8.RuneCountInString 方法可获得字符串转成 rune 数组的长度
    runeCount := utf8.RuneCountInString(content)    
    return runeCount + (len(content)-runeCount)/2
}
```

### 二分查找字符截断位置

若推文内容超过限制长度，根据其限制长度找到目前字符串中第一个长度为限制长度的子字符串。需要注意：

- 这里所指的长度是“推文中的字符长度”
- 在查询字符截断位置之前，需要算上前缀进度（如"(1/2)"）字符的长度
- 在进行截断的时候，避免出现因中文字符字节不完全导致中文乱码的情况

为了提高查找的性能，这里采用二分查询去找到字符串中第一个满足推文长度限制的位置，所以代码如下：

```Go
// splitFirstMaxTwitterContent 查找到对应第一个截断位置后输出切割后的左右字符串
func splitFirstMaxTwitterContent(content string) (left string, right string) {
    contentLen := len(content)
    if contentLen <= twitterMaxLength {
       return content, ""
    }

    // 可以看到这里计算长度都是按 Unicode 的方式去查找，避免出现中文乱码
    runeContent := []rune(content)
    runeIdx := sort.Search(len(runeContent), func(i int) bool {
       _, isExceed := calculateTwitterRuneContentLen(runeContent[:i])
       return isExceed
    })
    
    return string(runeContent[:runeIdx]), string(runeContent[runeIdx:])
}

func calculateTwitterRuneContentLen(runeContent []rune) (int, bool) {
    tweetLen := len(runeContent) + (len(string(runeContent))-len(runeContent))/2
    return tweetLen, tweetLen >= twitterMaxLength
}
```

## 使用说明

1. 执行编译好的 main 程序
2. 根据终端的提示输入想要发布的推文内容，输入完毕后换行输入"exit"标识结束
3. 最后若推文内容没有超过限制长度则返回原文，若有超过，程序会输出自动截断后的推文内容并加上前缀进度

```Shell
./main
===================================================
Please enter the content of the tweet (finally enter "exit" to end the input):
这会是一条测试文案：以下语句摘自《正见》
潜意识中，我们期待自己会到达不再需要修理任何东西的境界。总有一天，我们会「从此过着快乐的生活」。我们深信「解决」的概念。好像我们所有经历的一切，到这一刻为止的生命，都只是在彩排。盛大的演出还没开始。对大多数的人来说，这种永无休止的处理和重新安排以及更新版本，就是生活的定义。事实上，我们是在等待生命开始。
我们往往也会这么想：当我们死后，世界依然存在。同样的太阳会继续照亮大地，同样的星球会继续转动，因为我们认为从开天辟地以来，它们一直都是如此。我们的孩子会继承这个地球。这都显示出我们对于不断流转的世间和一切现象是多么无知。
exit
===================================================
Current Tweet Length: 566
Exceed the Twitter length limit: true
===================================================
Now back to the cut tweet content
>>>>>>>「the 1th tweet content」
(1/3)这会是一条测试文案：以下语句摘自《正见》
潜意识中，我们期待自己会到达不再需要修理任何东西的境界。总有一天，我们会「从此过着快乐的生活」。我们深信「解决」的概念。好像我们所有经历的一切，到这一刻为止的生命，都只是在彩排。盛大的演出还没开始。对大多数的人来说，这种永无休止的处理
>>>>>>>「the 2th tweet content」
(2/3)和重新安排以及更新版本，就是生活的定义。事实上，我们是在等待生命开始。
我们往往也会这么想：当我们死后，世界依然存在。同样的太阳会继续照亮大地，同样的星球会继续转动，因为我们认为从开天辟地以来，它们一直都是如此。我们的孩子会继承这个地球。这都显示出我们对于不断流转的世间和一切
>>>>>>>「the 3th tweet content」
(3/3)现象是多么无知。
>>>>>>>「End of tweet cutting」
```

## 后续拓展

- 终端交互优化
- 自动发布推文
- 排版优化
- .....