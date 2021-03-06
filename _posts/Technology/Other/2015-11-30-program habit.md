---

layout: post
title: 我们编程的那些潜意识
category: 技术
tags: Other
keywords: tcp ip socket

---

## 前言 ##

我们潜意识里的一些编程习惯，或许是从大一学C语言开始的

计算机科学就是研究计算：如何表示和处理信息。[学会思考，而不只是编程](http://www.infoq.com/cn/news/2017/06/Dont-learn-code-Learn-think)

## 我们用参数做什么

以java为例，一个方法有一个Object参数对象，

1. 一定是为了调用这个对象的方法么？在我潜意识里，是的

    
2. 一定是为了使用这个对象的功能么，比如得到一个值？换个角度说，一个对象里的某个方法，就一定是为了这个对象服务的么？在我潜意识里，是的

    确切的说，假如A对象有一个function，`void funcA(B b)`。那么这个funcA一定是为了对象A服务么，比如，进行一个计算，初始化A对象的一个成员？
    
    有时候，funcA虽然在A对象中，但其目的仅仅是为了操作（或初始化）b
    

也就是说，参数不一定是被利用的对象，也可能是受益的对象。


## 抽取一个类

假设原来有一个比较复杂的类

    class A{
        void func(){
            1.xx
            2.xx
            3.xx
            4.xx
            5.xx
        }
    }
    
现在我们代码重构，要将步骤234抽出一个类B来，类B需要A的数据初始化，类A需要类B的计算结果。一般有两种方案

    class A{
        void func(){
            1.xx
            2.B b = new B(xx);    // b作为A的类成员跟这个差不多
            3.xx = b.func();
            4.xx
        }
    }
    
但一些框架经常

    class A{
        void func(){
            1. xx
            2. xx
        }
    }
    class B{
        void func(A a){
            1. xx = a.getxx();
            2. xx
            3. a.setxx();
        }
    }
    class Main{
        main{
            A a = new A();
            B b = new B();
            b.func(a);
        }
    }
    
比如spring ioc初始化的一段代码便是如此

    // 定义配置文件    
    ClassPathResource res = new ClassPathResource(“beans.xml”);
    // 创建bean工厂
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    // 定义读取配置文件的类
    XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
    // 加载文件中的信息到bean工厂中
    reader.loadBeanDefinitions(res);
    

两种方式的不同在于：

1. 前者只是将相关代码抽取为一个函数，然后到了另一个类里。（本质上只算是抽取了一个函数）
2. 后者将相关代码完全抽出来，A类中不用保有任何痕迹，可以算是抽取出了一个类


## 代码的运行流程由我们控制么

对于一个小程序，代码的功能跟它呈现给我们的感觉是一致的（有main方法，有初始化和close等），而使用了框架之后，就出现了我们写的代码和它实际的能力错位的情况。

框架分为两类

1. 工具型或功能型框架，比如ibatis之类，封装了数据库操作。
2. 容器型框架，使用这些框架，我们只是填几个方法，整个功能就实现了。就好比，唐代的诗人，写诗，从头到尾自己斟酌。宋代的诗人作词，曲牌韵律已定，填词就好了。它的主体，是一个叫“容器”的东西。而我们所写的类，只是其中的一个部件而已。比如quartz，它有一个调度器执行在另一个条“主线”上，不停的查询或执行下一个将要执行的任务。

除此之外，Spring与其它框架的结合，往往从代码上改变了框架的使用“感觉”。其实，spring本质是ioc（及其基础上的aop），spring为框架提供的“方便”主要是ioc提供的，包括bean的生成，生命周期的管理（比如quartz的scheduler随着ioc容器的启动而启动，shutdown而shutdown），并不会改变框架（所提供类的）的使用方式（即一些接口方法的调用）。

## 方案的重要性

笔者曾经写过一个程序，查询hbase数据，并将其写入到本地文件中。

1. 单次查询hbase的效率并不好，所以笔者先缓冲一部分，然后批量查询。
2. 使用buffer批量查询后，带来两个问题

    1. 对结果的处理必须是批量的，即不再像以前一样，查询一个，处理一个。
    2. 如果是多线程程序，这个buffer需要增加线程安全保证。

3. 仅仅批量是不够的，因此笔者使用了缓存，查询时，先到缓存中查询，如果没有命中，则查询hbase，并将查询结果放到缓存中。
4. 因为用到了缓存、批量等，中间多了许多字符串判空操作（当然还有其它算是跟业务无关的判断操作，比如集合是否为空）。没错，即使一个“正常”的查询，也要经历这么多判断操作。

好了，其实我的需求很简单，查询数据，然后写入本地文件。但谁知道牵扯到了缓存和批量等问题。考虑到hbase中的数据虽然很多，但基本不会有很大的增长，笔者用了rocksdb，现在这个世界清静了，查一个，写一个。

第一个方案，实现复杂，代码多，易出错，一个正确数据执行的代码较多（为了程序的健壮性，有些判断操作不可避免，但我们希望尽量减少这些操作。因为，仅仅正确的代码是不够的）。而第二个方案，几乎在任何方面都比第一个方案有优势，当然，因为rocksdb的数据在本地存储，所以会占用较多的本地空间。

## 一开始就考虑好最复杂的情况，想好能复用和不能复用的部分，比如后期不断地修补好的多

## 线程和面向对象（未完待续）
