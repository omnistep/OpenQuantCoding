# 9. 使用OpenQuant的画图功能

在使用OpenQuant系统时， 我们通常要吧行情数据、指标数据、交易执行信息等在图中展示出来。OpenQuant系统菜单中打开view-chart, 或者view-chart\(gapless\)就可以看到OpenQuant的绘图板。Chart（gapless）是不考虑无数据时间的情况，例如盘后时间，或盘中休息、暂停交易等情况，绘制的是全连续的行情数据图形。

现在我们先绘制从画K线和均线开始。

## 绘制K线和均线并分区域绘制个性化指标数据

在场景文件中，先定义行情数据的K线属性，什么合约、什么周期，什么K线类型等等， OpenQuant不仅支持价格K线，同时也支持交易量K线，涨跌幅K线，Tick型K线等等其他类型。我们以时间型K线为例，场景scenario.cs代码为：

```text
using System;

using SmartQuant;

namespace OpenQuant
{
    public partial class Backtest : Scenario
    {

        //定义K线时间周期为barSize秒
        private long secBarSize =3;
        private long min1BarSize=60;


        public Backtest(Framework framework)
            : base(framework)
        {
        }

        public override void Run()
        {

            //定义要引入的合约名字
            Instrument instrument1 = InstrumentManager.Instruments["i1801"];


            strategy = new MyStrategy(framework, "Backtest");


            //引入合约
            strategy.AddInstrument(instrument1);

            //定义数据回测的起止日期
            DataSimulator.SubscribeBar = false;
            DataSimulator.DateTime1 = new DateTime(2017, 10, 13);
            DataSimulator.DateTime2 = new DateTime(2017, 10, 15); //  data from 09-01 to 09-30

            //定义一个时间类型的K线
            BarFactory.Clear();
            BarFactory.Add(instrument1, BarType.Time, secBarSize);    

            BarFactory.Add(instrument1, BarType.Time, min1BarSize);    

            Initialize();

            StartStrategy();
        }
    }
}
```

然后在主策略文件中的定义画布的参数和绘制，MyStrategy.cs的代码片段如下：

```text
    ... ... 
    public class MyStrategy : InstrumentStrategy
    {
        //定义策略名字 StrategyName，将被记录在日志中
        String StrategyName="LinesCrossover";            

        //定义SMA对象
        private SMA sma1;
        private SMA sma2;
        private SMA sma3;

            public int sma1Length = 20;
            public int sma2Length = 60;
            public int sma3Length = 80;

        ... 



        protected override void OnStrategyStart()
        {
            // Set up indicators.
            sma1 = new SMA(Bars, sma1Length);
            sma2 = new SMA(Bars, sma2Length);
            sma3 = new SMA(Bars, sma3Length);        


            //定义K线的画布，编码0号
            //颜色变量--------------------------------------------
            //Azure:天蓝色; Aquamarine:蓝晶色; Bisque:浓汤;
            //Beige:米色; BlanchedAlmond:杏仁白;BlueViolet:蓝紫;

            Group("myK_Chart", "Pad", 0);
            Group("myK_Chart", "CandleWhiteColor", Color.Red);
            Group("myK_Chart", "CandleBlackColor", Color.Lime);
            Group("Fills", "Pad", 0);


            Group("SMA1", "Pad", 0);
            Group("SMA1", "Color", Color.DodgerBlue);

            Group("SMA2", "Pad", 0);
            Group("SMA2","Width",2);
            Group("SMA2", "Color", Color.Red);

            Group("SMA3", "Pad", 0);
            Group("SMA3","Width",2);
            Group("SMA3", "Color", Color.DarkRed);        

            //定义指标的画布，编码1、2号
            Group("IndicatorCross", "Pad", 1);    
            Group("IndicatorCross","Width",2);
            Group("IndicatorCross", "Color", Color.DarkOrange);        

            Group("IndicatorTrend", "Pad", 2);    
            Group("IndicatorTrend","Width",2);
            Group("IndicatorTrend", "Color", Color.DodgerBlue);        



            //定义权益曲线的画布，编码3号
            Group("Equity", "Pad", 3);

            //定义分钟K线，使用编码4的画布
            Group("minK_Chart", "Pad", 4);
            Group("minK_Chart", "CandleWhiteColor", Color.Red);
            Group("minK_Chart", "CandleBlackColor", Color.Lime);
        }
```

在OnBar事件发生时，我们绘制K线和三条不同周期的均线 ,代码片段如下：

```text
    protected override void OnBar(Instrument instrument, Bar bar)
    {

        //当Bar形成时，增加bar数据到K线序列
        Bars.Add(bar);

        //在Bars画布上画出K线
        Log(bar, "myK_Chart");

        // Calculate performance.
        Portfolio.Performance.Update();
        // 在画布上绘制权益曲线
        Log(Portfolio.Value, "Equity");


        // Log sma.在画布上画出三条均线，如果尚未产生SMA数据时，跳出事件过程，不做处理
        if (sma1.Count == 0 || sma2.Count == 0 || sma3.Count == 0)
            return;

        Log(sma1.Last, "SMA1");
        Log(sma2.Last, "SMA2");
        Log(sma3.Last, "SMA3");


        ... ... 
        (... 略去一段指标计算代码，结果是计算indicatorCross数值)

        //绘制指标indicatorCross    
        Log(indicatorCross, "IndicatorCross");    

        (... 略去一段指标计算代码，结果是计算indicatorTrend数值)    

        //绘制指标indicatorTrend
        Log(indicatorTrend, "IndicatorTrend");

        ...  其他逻辑

    }
```

这样我们绘制的图像如下：

![](../.gitbook/assets/usingdrawingpad01.png)

上述完整LinesCrossover代码可以从如下地址获得：

![](../.gitbook/assets/icon_book%20%281%29.png) [https://github.com/omnistep/](https://github.com/omnistep/OpenQuantHelloWorld)LinesCrossover 获得。

