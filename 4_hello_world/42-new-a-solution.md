## 4.2创建一个新的解决方案

首先，我们用OpenQuant创建一个新的解决方案。

![](/icons/icon_labtubeBlue.ico)打开OpenQuant中的**File**菜单，**New**一个新的**solution**, 这里有一些解决方案类别选项，这个例子中，我们选择第一种“SmartQuant Instrument Strategy Solution”， 命名为_HelloWorld，_

### 新建Solution，对类型的选择：

![](/assets/42_New_Solution_HelloWorld.png)

这里我们选择第一种“SmartQuant Instrument Strategy Solution”，

> OpenQuant Solution的类别：
>
> SmartQuant Instrument Strategy Solution:
>
> SmartQuant Strategy Solution:
>
> SmartQuant Sell Side Strategy Solution:



在OpenQuant的Solution Explorer窗口中，我们可以发现当前有两个工程，**Backtest**和**MyStratey**.这就是最基础的代码框架了，一个回测场景定义工程（Backtest），一个策略主体工程（MyStrategy）。

### 新建的Solution结构如下：

![](/assets/42_NewSolutionExploer.png)

接下来，让我们先来编写Backtest场景模块。

