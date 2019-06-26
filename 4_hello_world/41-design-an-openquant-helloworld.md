# 4.1 设计OpenQuant的HelloWorld

在开始编码工作前，都应该先设计一下软件的构架，即便是一个简单的功能。HelloWorld 也需要设计一下，这个HelloWorld程序就是最基础的OpenQuant策略代码。虽然简单，但具备最核心的结构，未来可以扩展出我们需要的更多的解决方案。

策略的总体结构应该是这样：

![](/assets/design_helloworld.png)

整个解决方案由三个工程组成：

1. 回测场景定义的**Backtest**：主要定义用于回测的合约，及其历史数据的回测时间段；
2. 实盘场景定义的**Realtime**：主要定义交易通道，实盘交易的合约。
3. 策略主体的**MyStrategy**：主要编写策略逻辑，我们的Hellow World代码就会出现在这里。

而优化（Optimization）的部分，目前先忽略。现在就来创建这个HelloWorld吧！

