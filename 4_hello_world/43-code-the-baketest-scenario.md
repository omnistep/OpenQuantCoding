## 4.3 编写回测场景Backtest

![](/icons/icon_labtubeBlue.ico)

在这个场景模块中，我们增加回测场景的定义，包括测试的合约，我们引入螺纹钢期货rb1709合约，我们假设你已经在OpenQuant中引入了rb1709合约，同时使用历史数据插件导入了2017年1月1日至2017年3月1日的Tick级别的历史数据。

在**Backtest**工程的**MyScenario.cs**文件中，编写的代码如下，

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

在回测场景工程（Backtest）中，我们将场景类的名字从MyScenario改为Backtest，以便能和即将增加的Realtime场景类更为明确地加以区别。修改这个类名会涉及MyScenarios.cs，MySenario.Designer.cs，Program.cs 这3个文件中所有该类定义及调用的代码。如果用微软的Visual Studio工具将非常的自动化完成这类代码编写工作。

上面的代码public partial class **Backtest**:Scenario 修改了原来OpenQuant自动生成的public partial class **MyScenario**:Scenario 。

 为此我们也要修改这个工程的MyScenario.Designer.cs文件中这个“类”的名字， 把public partial class MyScenario改为public partial class Backtest，在Backtest工程中的**MyScenario.Designer.cs**文件中，编写代码如下

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

