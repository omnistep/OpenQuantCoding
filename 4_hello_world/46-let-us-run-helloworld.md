# 4.6 运行Hello World

HelloWorld编写完成，我们有了一个OpenQuant策略代码的基础框架，现在我们可以基于历史数据的回测模式（Backtest）下运行，也可以接入实盘交易通道，用Live模式运行。

## 4.6.1以回测模式（Backtest）运行HelloWorld

在Solution Exploer窗口中，我们确认Backtest作为程序的启动工程，这是Backtest是加粗的字体显示。如果不是这样，我们鼠标右击Backtest工程，在菜单中选择Set as StartUp Project，然后我们以Backtest模式运行

![](/assets/46_SetAsStartupProject.png)

我们直接以运行结果如下：

![](/assets/openquanthelloworldrunning.png)

## 4.6.2以实盘模式（Live）运行HelloWorld

同样，我们在Solution Exploer窗口中，我们确认Realtime作为程序的启动工程，这时Realtime是加粗的字体显示。如果不是这样，我们鼠标右击Realtime工程，在菜单中选择Set as StartUp Project，然后我们以Live模式运行。

在实盘运行时，随着行情数据的到来，我们可以看见output窗口中HelloWorld的逐行信息输出和Chart窗口中动态的K线显示。





![](/icons/icon_paw.png)现在我们已经有了一个HelloWorld解决方案，该方案已经让我们完成一个主体逻辑后，可以进行回测状态和实盘状态的定义和切换。如果需要回测，我们就将Backtest工程设置为启动项，如果进行实盘接入，我们就将Realtime工程设置为启动项。

完整的代码应该存在版本控制系统中进行管理。上述完整HelloWorld代码可以从如下地址获得：

![](/icons/icon_bookbig.png) [https://github.com/omnistep/OpenQuantHelloWorld](https://github.com/omnistep/OpenQuantHelloWorld) 获得。

