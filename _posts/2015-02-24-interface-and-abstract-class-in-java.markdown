---
layout:     post
title:      "浅谈Java中的接口和抽象类"
subtitle:   "Interface and abstract class in java"
date:       2015-02-24 17:02:00
author:     "Jiupeng Zhang"
header-img: "img/in-post/coding.jpg"
tags:
    - Review
---

Java中接口`interface`和抽象类`abstract class`的设计非常经典，二者在目的上很相似，而我们要编写出优秀的代码往往需要合理的抽象。虽然从语法上讲，接口可以简单看作是严格抽象的抽象类，但从设计的角度来讲，二者其实是缺一不可，相辅相成的。

<br/>

#### 让我们先感受一下接口和抽象类在语法上的异同：

##### **下面是接口的例子**

```java
interface Steerable {
    public static final String NAME_CH = "车";    // 只支持常量

    // 只支持抽象方法
    public abstract void run();

    // java8中新增的虚拟扩展方法（为了兼容旧版本）
    default public void aExtensionMethod(){
        //...
    }
}
```

##### **接下来是抽象类的例子**

```java
abstract class Vehicle {
    public static final String NAME_CH = "车";	// 可以有常量
    public static int count;					// 可以有类变量
    private long no;							// 可以有普通变量

    // 可以有抽象方法
    protected abstract void run();

    // 可以有构造方法
    protected Vehicle(long no) {
        this.no = no;
        this.count ++;
    }

    // 可以有类方法
    public static int getCount() {
        return count;
    }

    // 可以有普通方法
    public long getNo() {
        return no;
    }
}
```

<br/>

#### 从语法角度进行一下总结：

1. 抽象类实际上是一种特殊的类，和普通类不同的是，它允许你仅仅声明方法而不必编写方法体。
2. 接口和抽象类都是抽象概念，它们都可以用来实现多态，但你永远无法获得它们的实例。
3. 你其实不必纠结于抽象类中看似怪异的构造方法，显然，你是无法直接调用抽象类中的构造方法的，不过你可以在它子类的构造方法中间接构造这个抽象类（当然，这种抽象类中重写构造方法的情况比较少见）。
4. 接口从某种意义上来讲是一种完全抽象的抽象类，它所包含的所有方法都必须是抽象的（而且它必须是公有的，`public static abstract` 即接口中声明方法的唯一默认值）。
5. 从上面一条你可以看出，接口实际上是一种特殊的结构，Java里只有它可以实现多继承（弥补了Java类单继承的弱点）。
6. 接口和抽象类都支持基于继承关系的扩充（接口只能继承接口，抽象类既可以继承抽象类也可以继承普通类）。
7. 在编写某个接口的实现类时要用`implements`关键字进行约束，这个实现类会被要求实现接口中所声明的所有抽象方法。
8. 一个类可以实现一个或者多个接口，甚至它可以同时声明以实现父接口和它的子接口（当然这种做法是毫无意义的，所以接口间的继承要慎重设计）。
9. 当然，Java8中新增了虚拟扩展方法，它是为了更新原有jdk的某些接口时所引入的新概念，由于它打破了原来对接口的语法限制，所以不建议读者在自己的接口代码中使用`Defender`。
10. 从接口和抽象类的分离能看出Java语言的智慧，它既通过类的单继承设计解决了编程语言在描述现实问题时可能会出现的概念歧义，同时也通过增加接口来对Java类从特定功能的角度进行了无限的扩充。

<br/>

#### 下面让我们思考一下接口和抽象类在设计时的区别：

举一个简单的例子，我们来设计一个门，当然它可以开关而且每个门都有一个独一无二的出厂编号。我们生产产品有木门，玻璃门以及防盗门… 而 **门** 在这里只是一个抽象的概念，那么从抽象类的角度来设计，我们可能会写出下面这样的代码（下文中可能会使用`abscls:Door[abs:open(), abs:close()]`来表示下面这段代码）：

```java
// 可以看出抽象类的名称通常是名词
abstract class Door {
    public final int ID; // 门出厂时的编号
    protected Door(int id) {this.ID = id;}
    protected abstract void open();
    protected abstract void close();
}
```

然后通过不同的子类（如例子里的防盗门等）来分别实现子类不同的功能：

```java
class SecurityDoor extends Door {
    // 下面一行代码为抽象类中的常量ID进行了初始化
    public SecurityDoor(int id) { super(id); }
    protected void open() {/* do something... */}
    protected void close() {/* do something... */}
}
```

可以看出，这里利用抽象类对门这一抽象概念（事实上是一类事物）进行了抽象，那么接口又该如何使用呢？让我们假设这样一种场景，现在要增加一种新的门叫旋转门，那么这时候的 `Door` 显然无法满足我的需求，因为它只抽象了`open()`，`close()` 两类方法。

那么我们该如何处理这种情况呢？其实不难发现，我们所关注的重点不再是抽象一种实体本身，而是开始关注每种门具体的功能，那么给不同种类的门编写不同的抽象类再一一实现这种做法就变得毫无意义了（因为这这时的抽象是没有效果的），这时候我们就要用到接口了，**抽象类注重继承关系的抽象，而接口更注重行为的抽象**，让我们来看这样一段代码：

```java
// 可以看出接口的名称通常是形容词
interface OpenAndClosable {
    void open();
    void close();
}

interface Rotatable {
    void rotate();
}

// 看类的签名很容易理解这是一个能够开关的防盗门
class SecurityDoor implements OpenAndClosable {
    public void open() {/* do something... */}
    public void close() {/* do something... */}
}

// 看类的签名很容易理解这是一个能够旋转的旋转门
class RevolutionDoor implements Rotatable {
    public void rotate() {/* do something... */}
}
```

在上述代码中我们编写了两个不同的接口，并通过接口中声明的方法规范了该接口实现类的行为（就像可旋转的门必须重写旋转这个方法一样）。换句话说，我们设计了开关和旋转两个抽象行为，这个行为是独立于**任何**对象的，我们甚至能让收音机来实现`OpenAndClosable`这个接口。



由此可见，**使用抽象类或接口是两种截然不同的编程手段**，相信读者已经深有体会了，那么我们继续来完善这个例子，经过了接口的设计，我们已经顺利的将门的功能进行了细化或者说是分离（这种分离不宜过细，否则会让代码变得繁琐，像例子中把 `Openable` 和 `Closeble` 合并成一个接口 `OpenAndClosable` 是因为程序中这两种功能无需独立实现，所以接口要根据实际问题的粒度来慎重定义），用接口对例子中门的功能进行分离后，我们会发现他们公共的部分——出厂编号 `ID` 却丢失了，下面给出一种结合了接口与抽象类的实现：

```java
// 这是一个“可开关”接口，规范了实现它的类的“开关”行为
interface OpenAndClosable {
    void open();
    void close();
}

// 这是一个“可旋转”接口，规范了实现它的类的“旋转”行为
interface Rotatable {
    void rotate();
}

// 这是一个抽象类“门”，合并了实现类的ID，同时将以下两个类归纳成“门”类
abstract class Door {
    public final int ID;
    protected Door(int id) {this.ID = id;}
}

// 防盗门的具体实现
class SecurityDoor extends Door implements OpenAndClosable {
    protected SecurityDoor(int id) { super(id); }

    public void open() {/* do something... */}
    public void close() {/* do something... */}
}

// 旋转门的具体实现
class RevolutionDoor extends Door implements Rotatable {
    protected RevolutionDoor(int id) { super(id); }

    public void rotate() {/* do something... */}
}
```

上面这段代码很好的解释了两种不同抽象手段的抽象层次及作用。