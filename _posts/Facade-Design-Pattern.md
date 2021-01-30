---
title: Facade Design Pattern
date: 2020-12-07 22:53:33
tags:
  - GoF
---
## 门面模式的原理与实现
`门面模式`（Facade Design Pattern）是这样定义的：
> Provide a unified interface to a set of interfaces in a subsystem. Facade Pattern defines a higher-level interface that makes the subsystem easier to use.

翻译成中文就是：门面模式为子系统提供一组统一的接口，**定义一组高层接口让子系统更易用**。除此之外，我还要强调一下，门面模式定义中的“子系统（subsystem）”也可以有多种理解方式。它**既可以是一个完整的系统，也可以是更细粒度的类或者模块**。

## 门面模式的应用场景举例
实际上，它除了解决易用性问题之外，还能解决其他很多方面的问题。

### 解决易用性问题
门面模式可以用来**封装系统的底层实现，隐藏系统的复杂性**，提供一组更加简单易用、更高层的接口。比如，Linux `系统调用函数`就可以看作一种“门面”。它是 Linux 操作系统暴露给开发者的一组“特殊”的编程接口，它封装了底层更基础的 Linux 内核调用。再比如，Linux 的 `Shell 命令`，实际上也可以看作一种门面模式的应用。它继续封装系统调用，提供更加友好、简单的命令，让我们可以直接通过执行命令来跟操作系统交互。

我们知道，类、模块、系统之间的“通信”，一般都是通过接口调用来完成的。接口设计的好坏，直接影响到类、模块、系统是否好用。所以，我们要多花点心思在接口设计上。我经常说，**完成接口设计，就相当于完成了一半的开发任务**。只要接口设计得好，那代码就差不到哪里去。

接口粒度设计得太大，太小都不好。太大会导致接口不可复用，太小会导致接口不易用。在实际的开发中，**接口的可复用性和易用性需要微妙的权衡**。针对这个问题，我的一个基本的处理原则是，尽量保持接口的可复用性，但针对特殊情况，允许提供冗余的门面接口，来提供更易用的接口。
<!--more-->

### 解决性能问题
我们通过**将多个接口调用替换为一个门面接口调用，减少网络通信成本**，提高 App 客户端的响应速度。假设有一个系统 A，提供了 a、b、c、d 四个接口。系统 B 完成某个业务功能，需要调用 A 系统的 a、b、d 接口。利用门面模式，我们提供一个包裹 a、b、d 接口调用的门面接口 x，给系统 B 直接使用。

假设我们刚刚提到的系统 A 是一个后端服务器，系统 B 是 App 客户端。App 客户端通过后端服务器提供的接口来获取数据。我们知道，App 和服务器之间是通过移动网络通信的，网络通信耗时比较多，为了提高 App 的响应速度，我们要尽量减少 App 与服务器之间的网络通信次数。

如果我们现在发现 App 客户端的响应速度比较慢，排查之后发现，是因为过多的接口调用过多的网络通信。针对这种情况，我们就可以利用门面模式，让后端服务器提供一个包裹 a、b、d 三个接口调用的接口 x。App 客户端调用一次接口 x，来获取到所有想要的数据，将网络通信的次数从 3 次减少到 1 次，也就提高了 App 的响应速度。

### 解决分布式事务问题
在一个金融系统中，有两个业务领域模型，用户和钱包。这两个业务领域模型都对外暴露了一系列接口，比如用户的增删改查接口、钱包的增删改查接口。假设有这样一个业务场景：在用户注册的时候，我们不仅会创建用户（在数据库 User 表中），还会给用户创建一个钱包（在数据库的 Wallet 表中）。对于这样一个简单的业务需求，我们可以通过依次调用用户的创建接口和钱包的创建接口来完成。但是，用户注册需要支持事务，也就是说，**创建用户和钱包的两个操作，要么都成功，要么都失败**，不能一个成功、一个失败。

要支持两个接口调用在一个事务中执行，是比较难实现的，这涉及分布式事务问题。虽然我们可以通过**引入分布式事务框架或者事后补偿的机制**来解决，但代码实现都比较复杂。而最简单的解决方案是，利用数据库事务或者 Spring 框架提供的事务，在一个事务中，执行创建用户和创建钱包这两个 SQL 操作。这就要求**两个 SQL 操作要在一个接口中完成**，所以，我们可以借鉴门面模式的思想，再设计一个包裹这两个操作的新接口，让新接口在一个事务中执行两个 SQL 操作。