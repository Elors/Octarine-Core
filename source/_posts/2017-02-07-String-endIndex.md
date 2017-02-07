---
layout: post
title: Swift 中的 String.endIndex
date: 2017-02-07 16:30:36
tags: Swift
---

偶然在知乎发现了[一个很有意思的问题](https://www.zhihu.com/question/36963886)，在这里记录一下分析过程。

> 以下是问题（已更新至Swift3.1）：
>
> ```Swift
> let greeting = "zhang❤️\u{1f40c}"    // 这是一个包含 unicode 的 String。
>
> greeting.endIndex    // 9
> greeting.index(before: greeting.endIndex)    // 7
>
> for index in string.characters.indices {
>     print(string[index] ,index)
> }    //这个循环共执行7次
>
> // 以上9和7是怎么计算的？是什么样的关系？
> ```

<!-- more -->



在这个问题中 `"zhang❤️\u{1f40c}"` 等价于`"zhang❤️🐌"` ( "\u{1f40c}" 是 🐌 在Swift字符串中的字符串字面量: *Unicode 标量* )。
循环共执行7次也很好理解，因为 `greeting` 中包含了z、h、a、n、g、❤️、🐌，七个 `Character` 。




问题在于 endIndex 为 9。
按照我之前对 `String.endIndex` 的理解，`endIndex` 是字符串中最后一个有效下标向后一个位置的索引。即此处应为7。
> A string’s “past the end” position—that is, the position one greater than the last valid subscript argument.
>
> —— Swift Standard Library Documentation.
> 这时很容易就会联想到❤️和🐌都是由两个 Unicode标量 组合成的特殊字符，所以这里我遍历了 `greeting` 的所有 Unicode 标量。
```Swift
for s in greeting.unicodeScalars {
    print(s, s.value, s.escaped(asASCII: true))
}
/*
z 122 z
h 104 h
a 97 a
n 110 n
g 103 g
❤ 10084 \u{2764}
️ 65039 \u{FE0F}
🐌 128012 \u{0001F40C}
*/
```
从输出结果可以看到❤️是由两个Unicode标量组合成的特殊字符，而 🐌 则是一个单独字符。




既然关于 Unicode 标量 的猜想是错的，我只好老老实实的查看 `String.Character` 对应的 `Index` 了。
```Swift
for index in greeting.characters.indices {
    print(string[index] ,index)
}
/*
z Index(_base: Swift.String.UnicodeScalarView.Index(_position: 0), _countUTF16: 1)
h Index(_base: Swift.String.UnicodeScalarView.Index(_position: 1), _countUTF16: 1)
a Index(_base: Swift.String.UnicodeScalarView.Index(_position: 2), _countUTF16: 1)
n Index(_base: Swift.String.UnicodeScalarView.Index(_position: 3), _countUTF16: 1)
g Index(_base: Swift.String.UnicodeScalarView.Index(_position: 4), _countUTF16: 1)
❤️ Index(_base: Swift.String.UnicodeScalarView.Index(_position: 5), _countUTF16: 2)
🐌 Index(_base: Swift.String.UnicodeScalarView.Index(_position: 7), _countUTF16: 2)
*/
```




这下结果就非常明了了，由此可以猜想到 Swift 中的字符串在默认情况下可能是使用 UTF-16 进行编码存储的，结构大致如下图：
![Swift_Character](../../../../../gallery/Swift_Character.png)
通过这幅图很容易就能理解 `greeting.endIndex` 为什么会是 9，以及向前一位的 Index 为什么会是7。

