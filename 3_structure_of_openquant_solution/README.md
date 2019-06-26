# 3. 了解OpenQuant策略代码的结构

OpenQuant软件附带策略示例，这为我们展示了典型的策略代码，我们可以通过这些示例了解OpenQuant策略编程中代码应该具备的基础结构。

![](/icons/icon_labtubeBlue.ico)打开OpenQuant中的SMACrossover示例策略，这是一个经典的SMA交叉策略，打开后可以直接运行看到回测结果。这里我们着重看一下这个例子的代码结构和调用关系。

![](/assets/smacrossoversolutionexplorer.png)

在这个解决方案（Solution）中，包含如下四个工程：

**Backtest** ：回测，定义回测时的场景，引入的数据，回测的起始及结束日期，构造K线生成要素  
**MyStrategy** ：策略逻辑主体，定义均线，绘制K线及均线的主要图表，处理交易逻辑，处理成交事件，处理持仓等等  
**Optimization** ：策略优化参数定义，及策略优化的起始及结束日期  
**Realtime** ：实盘交易的场景，引入的合约，行情源及交易通道定义

如果您熟悉C\#语言开发，很快就可以发现OpenQuant的策略代码就是一个完全标准的C\#语言的解决方案。我们稍后章节会描述用微软的VisualStudio开发环境加载这些OpenQuant策略、开发、编译并调试。这个特性为我们的工作提供了极大的方便。

策略代码的结构如图所示：

![](/assets/smacrossovercodemap.png)

图：OpenQuant策略示例SMACrossover的代码结构。

