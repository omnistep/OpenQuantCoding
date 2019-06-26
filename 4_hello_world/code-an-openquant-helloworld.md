# 6. 开始编写OpenQuant的HelloWorld！

## 用OpenQuant创建一个基础代码框架

![](.gitbook/assets/icon_labtubeblue%20%285%29.ico)打开OpenQuant中的**File**菜单，**New**一个新的**solution**, 这里有一些解决方案类别选项，这个例子中，我们选择第一种“SmartQuant Instrument Strategy Solution”， 命名为_HelloWorld，_在OpenQuant的Solution Explorer窗口中，我们可以发现当前有两个工程，**Backtest**和**MyStratey**.这就是最基础的代码框架了，一个回测场景定义工程，一个策略主体工程。

## 增加一个场景工程-Realtime

![](.gitbook/assets/icon_labtubeblue.ico)现在，我们按照设计，再增加一个实盘交易场景工程Realtime。在OpenQuant的Solution Explorer中找到刚刚建立的解决方案HelloWorld，在HelloWorld上鼠标右键，**Add**一个**New Project...**  我们增加一个场景工程， 选择SmartQuant Scenario Project， 命名为 _**Realtime**_ 

现在我们按照设计，这个解决方案（Solution）中已经有了三个工程：一个回测场景，一个实盘场景，一个策略主体; 让我们来在这三个工程中加入代码。

## 开始编写回测场景Backtest

![](.gitbook/assets/icon_labtubeblue%20%284%29.ico)在回测场景工程（Backtest）中，将场景类的名字从MyScenario改为Backtest，以便能和Realtime场景类更为明确地加以区别。修改这个类名会涉及MyScenarios.cs，MySenario.Designer.cs，Program.cs 文件中所有该类定义及调用的代码。如果用微软的Vistual Studio工具将非常的自动化完成这类代码编写工作。

然后，我们增加回测场景的定义，包括测试的合约，我们引入螺纹钢期货rb1709合约，我们假设你已经在OpenQuant中引入了rb1709合约，同时使用历史数据插件导入了2017年1月1日至2017年3月1日的Tick级别的历史数据。

![](.gitbook/assets/icon_bookbig%20%281%29.png)OpenQuant引入合约代码及导入历史数据，请参考如下资料：

[http://www.smartquant.cn/book/domestic\_market\_data.html](http://www.smartquant.cn/book/domestic_market_data.html)

1. 在**Backtest**工程的**MyScenario.cs**文件中，编写代码如下

```text
using System;

using SmartQuant;

namespace OpenQuant
{
    public partial class Backtest : Scenario
    {
        //定义K线时间周期为barSize秒
        private long barSize = 60;

        public Backtest(Framework framework)
            : base(framework)
        {
        }

        public override void Run()
        {
            //定义要引入的合约名字
            Instrument instrument1 = InstrumentManager.Instruments["rb1709"];


            strategy = new MyStrategy(framework, "HelloWorld");

            //引入合约
            strategy.AddInstrument(instrument1);

            //定义数据回测的起止日期
            DataSimulator.SubscribeBar = false;
            DataSimulator.DateTime1 = new DateTime(2017, 01, 01);
            DataSimulator.DateTime2 = new DateTime(2017, 03, 01);

            //定义一个时间类型的K线
            BarFactory.Clear();
            BarFactory.Add(instrument1, BarType.Time, barSize);


            Initialize();

            StartStrategy();
        }
    }
}
```

上面的代码public partial class **Backtest**:Scenario 修改了原来OpenQuant自动生成的public partial class **MyScenario**:Scenario , 为此我们也要修改这个工程中的MyScenario.Designer.cs中这个“类”的名字， 把自动生成的public partial class MyScenario改为public partial class Backtest

在Backtest工程中的**MyScenario.Designer.cs**文件中，编写代码如下

```text
using System;

using SmartQuant;

namespace OpenQuant
{
    public partial class Backtest
    {
        public void Initialize()
        {
        }
    }
}
```

在Backtest工程中的**Program.cs**文件中，编写代码如下，同样将原有的MyScenario改为Backtest

```text
using System;

using SmartQuant;

namespace OpenQuant
{
    class Program
    {
        static void Main(string[] args)
        {
            Scenario scenario = new Backtest(Framework.Current);

            scenario.Run();
        }
    }
}
```

我们从这个工程的入口程序中，我们可以看到，程序开始运行后，就从一个叫做Backtest的场景类初始化，并开始让场景运行。

1. **开始编写核心策略逻辑MyStrategy**

我们在这个核心逻辑中，当行情数据产生了K线中新的Bar时，我们在信息输出窗口中向外输出期待已久的HelloWorld，同时显示合约名字和当前行情形成K的Bar的平均价格。

![](.gitbook/assets/icon_labtubeblue%20%288%29.ico)在**MyStrategy**工程中的**MyStrategy.cs**文件中，编写代码如下

```text
using System;

using SmartQuant;

namespace OpenQuant
{
    public class MyStrategy : InstrumentStrategy
    {
        public MyStrategy(Framework framework, string name)
            : base(framework, name)
        {
        }

        protected override void OnStrategyStart()
        {
            //定义K线的画布，编码0号
            Group("myK-Chart", "Pad", 1);
        }

        protected override void OnBar(Instrument instrument, Bar bar)
        {
        //当Bar形成时，增加bar数据到K线序列
        Bars.Add(bar);

        //在Bars画布上画出K线
        Log(bar, "myK-Chart");

        //就是这句期待已久的HelloWorld！同时显示bar时间，合约代码，bar中的均价
        Console.WriteLine("HelloWorld! "+bar.DateTime.ToString()+ " "+instrument.Symbol + "  ="+bar.Average.ToString());
        }
    }
}
```

![](.gitbook/assets/icon_paw%20%281%29.png)这时你有了一个可以回测合约的历史数据，基础框架，运行结果如下：

![](.gitbook/assets/openquanthelloworldrunning.png)

## 开始编写实盘接入场景Realtime

1. **引入MyStrategy工程类**

在接入实盘收行情时，我们会将解决方案中的Realtime作为启动工程，在这个工程中将实例化一个主逻辑MyStrategy类，**为此，我们先要在Realtime工程中，引入MyStrategy工程类。**正如OpenQuant生成的Backtest中已经引入了Mystrategy工程类一样。

![](.gitbook/assets/icon_labtubeblue%20%287%29.ico)在OpenQuant的Solution Explorer中的Realtime工程的References中，鼠标右键Add，选择Solution后，选择添加MyStrategy工程。

1. **编写实盘场景Realtime的代码**

接入实时到动态行情数据，我们要定义实盘接入场景，在Realtime工程中的**Scenario.cs**文件中，编写代码如下：

```text
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using SmartQuant;

namespace OpenQuant
{
    public partial class Realtime : Scenario
    {
        //定义K线时间周期为barSize秒
        private long barSize = 3;

         public Realtime(Framework framework)
            : base(framework)
        {
        }

        public override void Run()
        {
            //定义合约
            Instrument instrument1 = InstrumentManager.Instruments["ag1712"];

            strategy = new MyStrategy(framework, "HelloWorld");

            //引入合约到主策略工程
            strategy.AddInstrument(instrument1);

            //定义主策略中使用的合约的数据源和交易通道
            strategy.DataProvider = ProviderManager.GetDataProvider("QuantBoxCTP");
            strategy.ExecutionProvider = ProviderManager.GetExecutionProvider("QuantBoxCTP");

            //定义K线类型：时间型K线，barSize秒生成一个Bar
            BarFactory.Clear();
            BarFactory.Add(instrument1, BarType.Time, barSize);

            Initialize();

            //Run the strategy
            StartStrategy();

        }
    }
}
```

这里HelloWolrd解决方案的Realtime工程中的Scenario.Designer.cs及Program.cs均不需要改动代码。

![](.gitbook/assets/icon_labtubeorg.ico)在OpenQuant的Solution Explorer中，将Realtime工程设置为启动工程（Set as StartUp Project），然后把OpenQuant切换为实盘模式（Live）后，运行HelloWorld解决方案，并观察输出：

![](.gitbook/assets/icon_paw%20%286%29.png)现在我们已经有了一个HelloWorld解决方案，该方案已经让我们完成一个主体逻辑后，可以进行回测状态和实盘状态的定义和切换。如果需要回测，我们就将Backtest工程设置为启动项，如果进行实盘接入，我们就将Realtime工程设置为启动项。

完整的代码应该存在版本控制系统中进行管理。

上述完整HelloWorld代码可以从如下地址获得：

![](.gitbook/assets/icon_book%20%282%29.png) [https://github.com/omnistep/OpenQuantHelloWorld](https://github.com/omnistep/OpenQuantHelloWorld) 获得。

