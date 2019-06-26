**开始编写核心策略逻辑MyStrategy**

我们在这个核心逻辑中，当行情数据产生了K线中新的Bar时，我们在信息输出窗口中向外输出期待已久的HelloWorld，同时显示合约名字和当前行情形成K的Bar的平均价格。

![](/icons/icon_labtubeBlue.ico)在**MyStrategy**工程中的**MyStrategy.cs**文件中，编写代码如下

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



