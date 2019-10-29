---
layout:     post
title:      "回归程序的本质-语言无关"
subtitle:   "Thoughts on programming languages"
date:       2019-10-29 01:23:00
author:     "Jiupeng Zhang"
header-img: "img/in-post/coding.jpg"
tags:
    - Misc
---

这学期很荣幸能成为一门专业课的助教，我在老师建议下，将一个用C编写的网络模拟程序翻译成了自己“心爱”的Java版本，并布置给同学们作为一项编程作业，希望能够减轻大多数同学对编写C类代码的不适感。虽然出发点是好的，但课程论坛里的一位同学的强烈吐槽使我陷入了思考：

“Java缺少对无符号整数的支持，在这里做无符号数值运算以及类型转换简直是噩梦！我必须每次都显示声明类型转换，这让我的代码看起来无比丑陋，而且实现起来非常痛苦！”

他给出了以下这段程序，目的仅仅是为了将一个byte数组转换成int类型：

```java
Short.toUnsignedInt((short) ((short) Byte.toUnsignedInt(data[SEQ_LOW]) | ((short) (Byte.toUnsignedInt(data[SEQ_HIGH]) << 8))));
```

上面的代码确实丑陋，除了一大堆显示类型转换外，又套用了包装类里的转换方法，然而它想做的事仅仅是`a | b << 8`，于是我迅速在大脑中回想Java中不具有符号含义的基本类型，希望借此找到一种优雅的解决办法。然而我失败了，java中唯一不维护符号的基本类型只有`char`，它无法支持更大的数值，这反人类的设计简直是Java的软肋？因为这使得我们在Java中解释原始字节变得非常困难，而且代码中充斥了类型转换的陷阱（相反的是，C中类型仅仅是操作数据的句柄，甚至支持union这种反直觉但更加紧凑灵活的抽象方案，不得不说这使它数据操纵性极佳；而Java中的数据和类型却看似紧紧绑定在了一起）。于是我开始考虑Java在对应的包装类型中引入的无符号运算的支持，于是看到了诸如`parseUnsignedXX`，`toUnsignedXX`，`compareUnsigned`，`divideUnsigned`一类的workaround，更令人惊讶的是，这是在1.8中才引入的特性，不敢想象之前的代码处理无符号运算会是多么啰嗦和麻烦。Java，作为一个全面的，平台无关，　适用于规模化工程应用（讽刺的是，它自诞生起就尤其注重网络开发）的通用语言难道会有如此明显的疏漏？

说到这里，你可能会想到，Java程序员在位运算时往往离不开`&0xFF`这样的数据截断，因为这样可以解决隐性类型提升时，符号位扩充的问题，使我们能在原有的代码逻辑里，把一个有符号整数当成无符号数进行处理。但是这种将问题放大并复杂化的设计，是否有违背编程语言的本质？为什么Java没有在最初提供一组无符号的基本数据类型？这让我不由得动摇，甚至开始质疑Java的设计理念，因为这违背了我所理解和喜爱的Java哲学：“去繁就简，统一表达，让更多人写出更优质的代码”。于是我尝试在网上查找相关的资料，试图找出这里的“隐情”。

于是我找到了一段Gosling参与的访谈中的对话（[查看原文](http://www.gotw.ca/publications/c_family_interview.htm)），里面表达了他，这个语言的缔造者对于“一个简洁编程语言”的理解：

> For me as a language designer, which I don't really count myself as these days, what "simple" really ended up meaning was could I expect J. Random Developer to hold the spec in his head. That definition says that, for instance, Java isn't -- and in fact a lot of these languages end up with a lot of corner cases, things that nobody really understands. Quiz any C developer about unsigned, and pretty soon you discover that almost no C developers actually understand what goes on with unsigned, what unsigned arithmetic is. Things like that made C complex. The language part of Java is, I think, pretty simple. The libraries you have to look up.”

大致意思是，他抛弃无符号运算等一些tricky的功能，目的是为了减少开发者在这方面犯愚蠢的错误，因为大多程序员甚至不了解什么是无符号运算。我恍然大悟，这终于印证了那句老话 “Java语言不会帮助天才开发者写出最优的代码，但可以让愚蠢的程序员犯最少的错误”。

我不会在这里争论Java语言的性能，因为我觉得这是这门语言本身高明的地方 — 在规模化的工程应用中巧妙的找到平衡点。我也认同Java语言本身确实有非常多槽点（lambda过于繁琐，异常无法传出，以及鸡肋的类型推断等等），但是这个unsigned类型缺失的问题，确实让我认识到，这可能是个更大的短腿和败笔，对数据操纵方式的局限，使开发者处理原始数据时感到乏力（当然你也可以理解成Java造就了一种和C截然不同的编程思维和方式）。能想象到Gosling绞尽脑汁想简化Java本身，但是与此同时他让这门语言失去了人们表达逻辑的简洁性和合理性（譬如用有符号数来表达数组长度既浪费又无意义）。

突然想起来实习时mentor的一个观点（因为我当时过度迷恋Java，即使我在工作中使用其它语言），大概意思是：语言只是逻辑思维的表达工具，任何一个语言都会有自己的优势和局限性。足够满足大多数开发需要的前提下，挑一门最合适，让你觉得最舒服的语言就好，但注意不要让语言本身限制到你表达思维的能力... 

是啊！编程语言本身不就是一种用来表达逻辑思维的一种抽象工具。其实我们都应该更关注于程序本身，尽自己所能，让我们写出更优质的代码，表达出来更加清晰、精简的逻辑。这恐怕是一名真正的开发者应该追求的一种境界吧。

原来，对我来说，最理想的语言，应该是一门不会限制我思维（表达能力强），不会让我分心（合理的约束，良好的性能），并且能帮助我高效解决问题（完整的生态）的得力助手，真希望可以早点见到它！

<br/><br/>

本文所有权归作者所有，禁止未经许可的转载，违者将追究相关责任。

原始链接：<https://jiup.github.io/2018/10/29/thoughts-on-java/>