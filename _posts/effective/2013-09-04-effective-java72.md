---
layout: post
title: 异常-优先使用标准的异常
date: 2019-09-04
Author: 邶城花语
tags: [高效 Java]
comments: true
---
### 72. 优先使用标准的异常

　　专家级程序员与缺乏经验的程序员一个最主要的区别在于，专家追求并且通常也能够实现高度的代码重用。代码重用是值得提倡的，这是一条通用的规则，异常也不例外。Java 平台类库提供了一组基本的未受检异常，它们满足了绝大多数 API 的异常抛出需求。

　　重用标准的常有多个好处。其中最主要的好处是，它使 API 更易于学习和使用，因为它与程序员已经熟悉的习惯用法一致。第二个好处是，对于用到这些 API 程序而言，它们的可读性会更好，因为它们不会出现很多程序员不熟悉的异常。最后（也是最不重要的）一点是，异常类越少，意味着内存占用（footprint）就越小，装载这些类的时间开销也越少。

　　最经常被重用的异常类型是 `IllegalArgumentException`（详见第 49 条）。当调用者传递的参数值不合适的时候，往往就会抛出这个异常。比如，假设某一个参数代表了“某个动作的重复次数”，如果程序员给这个参数传递了一个负数，就会抛出这个异常。

　　另一个经常被重用的异常是 `llegalStateException`。如果因为接收对象的状态而使调用非法，通常就会抛出这个异常。例如，如果在某个对象被正确地初始化之前，调用者就企图使用这个对象，就会抛出这个异常。

　　可以这么说，所有错误的方法调用都可以被归结为非法参数或者非法状态，但是，还有一些其他的标准异常也被用于某些特定情况下的非法参数和非法状态。如果调用者在某个不允许 null 值的参数中传递了 null，习惯的做法就是抛出 `NullPointerException` 异常，而不是 `IllegalArgumentException`。同样地，如果调用者在表示序列下标的参数中传递了越界的值，应该抛出的就是 `IndexOutOfBoundsException` 异常，而不是 `IllegalArgumentException`。

　　另一个值得了解的通用异常是 `ConcurrentModificationException`。如果检测到一个专门设计用于单线程的对象，或者与外部同步机制配合使用的对象正在（或已经）被并发地修改，就应该抛出这个异常。这个异常顶多就是一个提示，因为不可能可靠地侦测到并发的修改。

　　最后一个值得注意的标准异常是 `UnsupportedOperationException`。如果对象不支持所请求的操作，就会抛出这个异常。很少用到它，因为绝大多数对象都会支持它们实现的所有方法。如果类没有实现由它们实现的接口所定义的一个或者多个可选操作（optional operation），它就可以使用这个异常。例如，对于只支持追加操作的 List 实现，如果有人试图从列表中删除元素，它就会抛出这个异常。

　　**不要直接重用 Exception、RuntimeException、Throwable 或者 Error。** 对待这些类要像对待抽象类一样。你无法可靠地测试这些异常，因为它们是一个方法可能抛出的其他异常的超类。

　　下表概括了最常见的可重用异常：


|              异常               |                   使用场合                   |
| :-----------------------------: | :------------------------------------------: |
|    IllegalArgumentException     |             非 null 的参数值不正确             |
|      IllegalStateException      |           不适合方法调用的对象状态           |
|      NullPointerException       |      在禁止使用 null 的情况下参数值为 null      |
|    IndexOutOfBoundsExecption    |                下标参数值越界                |
| ConcurrentModificationException | 在禁止并发修改的情况下，检测到对象的并发修改 |
|  UnsupportedOperationException  |           对象不支持用户请求的方法           |

　　然这些都是 Java 平台类库中迄今为止最常被重用的异常，但是，在条件许可的情况下，其他的异常也可以被重用。例如，如果要实现诸如复数或者有理数之类的算术对象，也可以重用 `ArithmeticException` 和 `NumberFormatException`。如果某个异常能够满足你的需要，就不要犹豫，使用就是，不过一定要确保抛出异常的条件与该异常的文档中描述的条件一致。这种重用必须建立在语义的基础上，而不是建立在名称的基础之上。而且，如果希望稍微增加更多的失败一捕获（failure-capture）信息（详见第 75 条），可以放心地子类化标准异常，但要记住异常是可序列化的（详见第 12 章）。这也正是「如果没有非常正当的理由，千万不要自己编写异常类」的原因。

　　选择重用哪一种异常并非总是那么精确，因为上表中的“使用场合”并不是相互排斥的比如，以表示一副纸牌的对象为例。假设有一个处理发牌操作的方法，它的参数是发一手牌的纸牌张数。假设调用者在这个参数中传递的值大于整副纸牌的剩余张数。这种情形既可以被解释为 `IllegalArgumentException`( handSize 参数的值太大），也可以被解释为 `IllegalStateException`（纸牌对象包含的纸牌太少）。在这种情况下，**如果没有可用的参数值，就抛出 `IllegalStateException`，否则就抛出 `IllegalArgumentException`。**
