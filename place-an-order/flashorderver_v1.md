# 7.3 FlashOrder的代码 v1

FlashOrder策略还是由3个工程组成，Backtest，MyStrategy和Realtime。

其中Backtest工程中的MyScenario.cs代码为：

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


            strategy = new MyStrategy(framework, "Backtest");


            //引入合约
            strategy.AddInstrument(instrument1);

            //定义数据回测的起止日期
            DataSimulator.SubscribeBar = false;
            DataSimulator.DateTime1 = new DateTime(2017, 08, 01);
            DataSimulator.DateTime2 = new DateTime(2017, 09, 02);

            //定义一个时间类型的K线
            BarFactory.Clear();
            BarFactory.Add(instrument1, BarType.Time, barSize);    

            Initialize();

            StartStrategy();
        }
    }
}
```

Realtime工程中的Scenario.cs代码为：

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
            Instrument instrument1 = InstrumentManager.Instruments["au1712"];

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

而MyStrategy工程中先要引入QuantBox.OQ.dll, 该文件安装QuantBoxCTP插件后，可以在OpenQuant默认安装路径中找到。MyStrategy.cs代码为：

```text
using System;
using System.Drawing;
using SmartQuant;
using QuantBox;

namespace OpenQuant
{
    public class MyStrategy : InstrumentStrategy
    {


        int orderQty = 1;          //定义交易定单的发单量多少手
        int swLongEnable  = 1;     //定义做多交易许可开关，0：禁止，1：允许
        int swShortEnable = 1;     //定义做空交易许可开关，0：禁止，1：允许

        int theOrdBOid = 0 ;         //定义做多△买入开仓定单id
        int theOrdSCid = 0 ;         //定义做多 ▼卖出平仓定单id
        string theOrdSCstatus = "";  //定义做多 ▼卖出平仓定单状态

        int theOrdSSid = 0 ;         //定义做空▽卖出开仓定单id
        int theOrdBCid = 0 ;         //定义做空 ▲买入平仓定单id

        int myCentOpen  = 1 ;        //定义开仓追价成本是多少TickSize；
        int myCentClose = 0 ;        //定义平仓追价成本是多少TickSize；
                                     //默认1：代表盈利1个TickSize挂单
                                     //    0：代表市场平价报单；
                                     //   -1：代表1个TickSize的成本报单

    public MyStrategy(Framework framework, string name)
          : base(framework, name)
        {
        }

        protected override void OnStrategyStart()
        {
            //定义K线的画布，编码0号
            Group("myK_Chart", "Pad", 0);
            Group("myK_Chart", "CandleWhiteColor", Color.Red);    
            Group("myK_Chart", "CandleBlackColor", Color.Lime);    
            Group("Fills", "Pad", 0);
            Group("Equity", "Pad", 1);

        }

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



            //就是这句期待已久的HelloWorld！同时显示bar时间，合约代码，bar中的均价
            Console.WriteLine("OnBar------HelloWorld! "+bar.DateTime.ToString()+ " "+instrument.Symbol + " ="+bar.Average.ToString());
            Console.WriteLine("swLongEnable= "+swLongEnable.ToString()+ ";  swShortEnable ="+swShortEnable.ToString());
            //显示平仓单状态
            System.Console.WriteLine("theOrdSCstatus = " + theOrdSCstatus);

            if(swLongEnable==1)
            {
                //买入-开仓
                Order orderBO = BuyLimitOrder(Instrument, orderQty, bar.Close - myCentOpen*Instrument.TickSize, "△");
                //orderBO.Open();
                Send(orderBO);      

                swLongEnable = 0;
                theOrdBOid = orderBO.Id;
                System.Console.WriteLine("▓ orderBO.Id=" + orderBO.Id.ToString());
            }


            if(swShortEnable==1)
            {
                //卖出-开仓
                Order orderSS = SellLimitOrder(Instrument, orderQty, bar.Close + myCentOpen*Instrument.TickSize, "▽");
                //orderSS.Open();
                Send(orderSS); 

                swShortEnable = 0;
                theOrdSSid = orderSS.Id;
                System.Console.WriteLine("▓ orderSS.Id=" + orderSS.Id.ToString());
            }

            /*



            */
        }

        protected override void OnFill(SmartQuant.Fill fill)
        {
            // 在画布上绘制成交记录
            Log(fill, "Fills");


            // 在Output窗口中输出fill对象的当前数据...
            System.Console.WriteLine("fill.DateTime=" + fill.DateTime.ToString());
            System.Console.WriteLine("fill.CashFlow=" + fill.CashFlow.ToString());
            System.Console.WriteLine("fill.Commission=" + fill.Commission.ToString());
            System.Console.WriteLine("fill.Instrument.Symbol=" + fill.Instrument.Symbol.ToString());
            System.Console.WriteLine("fill.Instrument.Description=" + fill.Instrument.Description.ToString());
            System.Console.WriteLine("fill.Instrument.Trade=" + fill.Instrument.Trade.ToString());
            System.Console.WriteLine("fill.Text=" + fill.Text.ToString());



            if(fill.Order.Id == theOrdBOid)    
            {
                System.Console.WriteLine("▓▓▓▓▓ --> 卖出-平仓");
                //卖出-平仓
                Order orderSC = SellLimitOrder(Instrument, orderQty, fill.Price + myCentClose*Instrument.TickSize, "▼");
                orderSC.CloseToday() ;
                Send(orderSC);  

                theOrdSCid = orderSC.Id;
                theOrdSCstatus = orderSC.Status.ToString();
                System.Console.WriteLine("▓▓▓▓▓ theOrdSCstatus = " + theOrdSCstatus);

                swLongEnable = 1;
            }

            if(fill.Order.Id == theOrdSSid)
            {
                System.Console.WriteLine("▓▓▓▓▓ --> 买入-平仓");
                //买入-平仓
                Order orderBC = BuyLimitOrder(Instrument, orderQty, fill.Price - myCentClose*Instrument.TickSize, "▲");
                orderBC.CloseToday();
                Send(orderBC);  
                System.Console.WriteLine("▓orderBC.SubSide=" + orderBC.SubSide.ToString());

                theOrdBCid = orderBC.Id;

                swShortEnable = 1;
            }



            if(fill.Order.Id == theOrdSCid)
            {
                swLongEnable = 1;
            }

            if(fill.Order.Id == theOrdBCid)
            {
                swShortEnable = 1;
            }

        }


    }
}
```

![](../.gitbook/assets/icon_labtubeorg%20%282%29.ico)运行策略后，应该通过OpenQuant自己的OrderManager及CTP通用交易客户终端查看到报单状况。

![](../.gitbook/assets/flashorderfilled.png)

