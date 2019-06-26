# 4.构建一个OpenQuant的HelloWorld策略

在对OpenQuant的代码结构有了大致的了解后，让我们来开始构建一个OpenQuant的HelloWorld系统吧...

## Hello World的需求描述

我们需要建立一个OpenQuant的代码基础构架——**HelloWorld策略**，该策略要提供历史数据回测结构，也要可以实盘连接到CTP交易柜台，还可以订阅一个全市场最热的合约，当收到每一笔Tick行情时，在输出窗口写一句HelloWorld及所订阅合约的当前报价！

## Hello World策略的构建过程

* 设计总体构架
* 新建解决方案
* 编写回测场景模块
* 编写核心策略模块
* 编写实盘场景模块
* 运行HelloWorld



