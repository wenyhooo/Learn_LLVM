[TOC]

# 学习LLVM认识代码编译过程
* [学习LLVM认识代码编译过程](#学习llvm认识代码编译过程)
     * [传统的编译器架构它到底长什么样子？](#传统的编译器架构它到底长什么样子)
        * [Frontend](#frontend)
        * [Optimizer](#optimizer)
        * [Backend](#backend)
     * [LLVM也分为这三个部分](#llvm也分为这三个部分)
     * [Clang是什么呢？](#clang是什么呢)
        * [LLVM和Clang的关系](#llvm和clang的关系)
        * [相对于GCC ,Clang的优点](#相对于gcc-clang的优点)
     * [编译的几个阶段](#编译的几个阶段)
        * [preprocessor阶段](#preprocessor阶段)
        * [什么是词法分析](#什么是词法分析)

LLVM是可重用的编辑器（compiler）以及工具链（toolchain）技术的集合

> LLVM的发起人是Chrils Lattner，Clang编译器作者，同时也是Swfit之父。

传统的编译器架构：GCC、Clang、LLVM等。

### 传统的编译器架构它到底长什么样子？

它分为三个阶段：

1. #### Frontend

   compiler Frontend它是编译器前端，主要负责接收源代码（OC、Swfit....）进行<font color=red>词法分析、语法分析、语义分析、生成中间代码（LLVM IR）</font>，我们的<font color=red>编译出错</font>全靠它。

   ![image-20200725170418896](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh3btykg6bj313i07sgpl.jpg)

   生成LLVM IR之后通进入到下一步Optimizer。

2. #### Optimizer

   Optimizer是代码优化器（LLVM Optimizer）, 优化目的：运行速度更快、体积更小

   ![image-20200725164225127](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh3b79cj8cj314g0aytag.jpg)

3. #### Backend

   Backend编译器后端，根据不同运行平台生成不同架构的机器码

   ### LLVM也分为这三个部分

![image-20200725180614363](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh3dml8rqaj30xc0c041w.jpg)

具体过程如下：

```c++
source code->前端编译器(OC的Clang)->优化器（Optimizer）->后端->machinecode
```

通过上图我们可以看出LLVM有如下优势：

- 不同的前端和后端，他们都使用统一的中间代码LLVM IR
- 如果增加一种新的编程语言，只需要增加实现一个新的前端即可 LLVM Optimizer之后的都可以不用做更改
- 如果增加一种新的硬件设备，只需要增加一种新的后端即可
- 相对于GCC，就灵活很多，GCC的前端、后端耦合会比较多，可能增加一种编程语言就要为此增加一种后端
- LLVM现在被称为各种静态和运行时编译语言的通用基础架构如（java、Python、Ruby）

### Clang是什么呢？

它是LLVM的子项目，属于编译器前端（也如上图所示，是在Optimizer之前的一个过程）

是基于LLVM架构，也是LLVM广义上的一部分，支持C、OC、C++语言。

#### LLVM和Clang的关系

LLVM架构：<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gh3h85p21ij31hc0u0wh7.jpg" alt="未命名.001" style="zoom:25%;" />

- 从广义来说整个包括Clang都是LLVM框架
- 从狭义上来讲LLVM又分为上图两部分，Clang是LLVM的前端
- 前端：主要做 语法、词法等分析
- 后端：主要生成对应平台的二进制代码

#### 相对于GCC ,Clang的优点

- 编译速度快，debug模式要比GCC快三倍
- 占用内存小，Clang生成的AST（语法树），所占内存时GCC的1/5
- 模块化设计，Clang采用模块化设计，易于IDE集成以及其他用途，还可自制插件接入
- 可读性好，编译过程中Clang会创建并保留大量元数据，利于debug

### 编译的几个阶段

拿main.m文件来举例学习一下

![image-20200725202745467](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh3hpmse36j30uq0amq3v.jpg)

cd 到mian.m目录 命令查看：

```shell
clang -ccc-print-phases main.m                  
```

![image-20200725201959536](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh3hhn19adj312a0aedib.jpg)

1. 0: input, "main.m", objective-c   //找到源代码
2. 1: preprocessor, {0}, objective-c-cpp-output     //预处理器 处理import 替换宏定义等
3. 2: compiler, {1}, ir //编译器 编译成IR
4. 3: backend, {2}, assembler  //后端
5. 4: assembler, {3}, object // 目标代码
6. 5: linker, {4}, image    //链接动态库等
7. 6: bind-arch, "x86_64", {5}, image // 链接到架构代码

#### preprocessor阶段

执行命令：

```shell
clang -E -rewrite-objc -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk main.m
```

结果如下

```c
int main(int argc, char * argv[]) {
    int a = 20;
    int b = 30;
    int c = a+b-40;
}
```

可以看到宏定义age被替换为值40

#### 什么是词法分析

执行命令：

```shell
clang -fmodules -E -Xclang -dump-tokens main.m
```

![image-20200725203346226](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh3hvw9v1xj319a0jadnk.jpg)
