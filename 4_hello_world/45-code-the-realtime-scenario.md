## 编写实盘场景模块-Realtime

## 增加一个场景工程-Realtime

![](/icons/icon_labtubeBlue.ico)现在，我们按照设计，再增加一个实盘交易场景工程Realtime。在OpenQuant的Solution Explorer中找到刚刚建立的解决方案HelloWorld，在HelloWorld上鼠标右键，**Add**一个**New Project...**  我们增加一个场景工程， 选择SmartQuant Scenario Project， 命名为 _**Realtime**_

![](/icons/icon_labtubeOrg.ico)现在我们按照设计，这个解决方案（Solution）中已经有了三个工程：一个回测场景，一个实盘场景，一个策略主体; 让我们来在这三个工程中加入代码。

## **引入MyStrategy工程类**

在接入实盘收行情时，我们会将解决方案中的Realtime作为启动工程，在这个工程中将实例化一个主逻辑MyStrategy类，**为此，我们先要在Realtime工程中，引入MyStrategy工程类。**正如OpenQuant生成的Backtest中已经引入了Mystrategy工程类一样。

![](/icons/icon_labtubeBlue.ico)在OpenQuant的Solution Explorer中的Realtime工程的References中，鼠标右键Add，选择Solution后，选择添加MyStrategy工程。

## **编写实盘场景Realtime的代码**

接入实时到动态行情数据，我们要定义实盘接入场景，在Realtime工程中的**Scenario.cs**文件中，编写代码如下：

```c\#
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

上面代码中，对交易通道的定义，需要注意交易通道名字需要和安装的通道插件名字一致。

HelloWolrd解决方案的Realtime工程中的Scenario.Designer.cs及Program.cs均不需要改动代码。这样就完成了Realtime工程代码的编写。

