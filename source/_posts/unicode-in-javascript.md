---
title: JavaScript 中的 Unicode
date: 2018-7-21 22:05:58
categories:
  - FE
tags:
  - Javascript
  - Unicode
---

在以往的项目中经常会遇到多字节字符的处理，最近重新查阅了一些资料，记录一下关于 Unicode 的一些知识点，方便日后学习。

<!-- more -->

### Unicode

Unicode 是目前绝大多数程序使用的字符编码，定义也很简单，用一个码点（`Code Point`）映射一个字符。码点值的范围是从 `U+0000` 到 `U+10FFFF`，可以表示超过 110 万个符号。比如：

- A 的码点：`U+0041`
- a 的码点：`U+0061`
- 😐 的码点：`U+1F610`

每个码点的详细信息都可以在 [Codepoints](https://codepoints.net/) 查询到。

Unicode 最前面的 65536 个字符位，称为基本平面（`Basic Multilingual Plane`），它的码点范围是从 `U+0000` 到 `U+FFFF`。最常见的字符都放在这个平面，这是 Unicode 最先定义和公布的一个平面。

剩下的字符都放在补充平面（`Supplementary Plane`），码点范围从 `U+010000` 一直到 `U+10FFFF`，共 16 个。

### UTF 与 UCS

UTF（`Unicode Transformation Format`）Unicode 转换格式，是服务于 Unicode 的，用于将一个 Unicode 码点转换为特定的字节序列。常见的 UTF 有：

> UTF-8 可变字节序列，用 1 到 4 个字节表示一个码点 UTF-16 可变字节序列，用 2 或 4 个字节表示一个码点 UTF-32 固定字节序列，用 4 个字节表示一个码点

然而，由于 JS 问世的时候，`UTF-16` 还没有诞生，所以使用了 `UCS-2` 来处理字符，也因此留下了很多坑。

UCS（`Universal Character Set`）通用字符集，是一个 ISO 标准，目前与 Unicode 可以说是等价的。 相对于 UTF，UCS 也有自己的转换方法（编码）。如：

> UCS-2 用 2 个字节表示 BMP 的码点 UCS-4 用 4 个字节表示码点

`UCS-2` 是一个过时的编码方式，因为它只能编码基本平面（BMP）的码点，在 BMP 的编码上，与 `UTF-16` 是一致的，所以可以认为是 `UTF-16` 的一个子集。

`UCS-4` 则与 `UTF-32` 等价，都是用 4 个字节来编码 Unicode。

`UTF-16` 对于 BMP 的码点，采用 2 个字节进行编码，而 BMP 之外的码点，用 4 个字节组成代理对（`Surrogate Pair`）来表示。其中前两个字节范围是 `U+D800` 到 `U+DBFF`，后两个字节范围是 `U+DC00` 到 `U+DFFF`，通过以下公式完成映射：

```Javascript
// （H：高字节 L：低字节 c：码点）

H = Math.floor((c - 0x10000) / 0x400) + 0xD800

L = (c – 0x10000) % 0x400 + 0xDC00
```

### JS 字符处理

这里我们拿上面的 😐 的码点来解释，由于 JS 认为 2 个字节表示一个字符，而 😐 在 JS 中的编码是 `\ud83d\ude10`，所以这个表情是占两个字符的，那么这时用 length 判断长度的时候会是 2。

这时候，我们可以通过正则来识别这些占两个字符的文字或表情，然后将其替换为 BMP 内的字符就可以了。

```Javascript
var reg = /[\uD800-\uDBFF][\uDC00-\uDFFF]/g // 匹配 UTF-16 的代理对

function count(string) {
	return string
		.replace(reg, '-')
		.length
}

count('😐') // 1
```

当然，ES6 中已经增加了很多对于 Unicode 字符的新方法，比如 `String.codePointAt()` 方法，`\u{128528}` 通过加上花括号和标示来表示而不用去计算代理对。
