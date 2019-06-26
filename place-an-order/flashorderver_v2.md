# 7.5 同时处理多个合约 FlashOrder v2

通常我们需要相同的策略同时处理多个合约，根据实际情况，我们选择几种方法来管理多个合约的信息。

* **最简单的方法**

简单的方法就是在场景文件Scenario.cs中，直接定义新的合约变量，复制相关代码即可， 主策略代码不用改变，策略运行时，你会发现一个合约就是策略的一个实例，它们都在并行的运行。

```text
            //定义合约
            Instrument instrument1 = InstrumentManager.Instruments["au1712"];
            Instrument instrument2 = InstrumentManager.Instruments["rb1801"];
            Instrument instrument3 = InstrumentManager.Instruments["c1801"];
            ... ... 


        //引入合约到主策略工程
            strategy.AddInstrument(instrument1);
            strategy.AddInstrument(instrument2);
            strategy.AddInstrument(instrument3);
            ... ... 

        //定义K线类型：时间型K线，barSize秒生成一个Bar
            BarFactory.Clear();
            BarFactory.Add(instrument1, BarType.Time, barSize);
            BarFactory.Add(instrument2, BarType.Time, barSize);
            BarFactory.Add(instrument3, BarType.Time, barSize);
            ... ...
```

* **优雅的方法,使用数组字符串**

在场景文件Scenario.cs中，可以定义一个变量数组， 采用循环代码进行加载，但这个方法中， 会出现你想加入的合约信息并没有在OpenQuant引入过，代码会报出提示， 如果有这样的情况，你应该先在OpenQuant中手动引入相关的合约信息（过程详见1.5 导入合约信息），场景文件中的相关代码如下：

```text
        public override void Run()
        {

        //定义合约
        string[] myInstruments = {"au1712", "rb1801","c1801","m1801","p1801","TA801","i1801","MA801","ru1801", 
                              "hc1801","ni1801","IF1710","IC1710","IH1710","fb1801","jm1801","RM801","ZC801", 
                              "SR801","al1712","ag1712","cu1711","OI801"};


        ... ... 


        //引入合约到主策略工程
        foreach (string symbol in myInstruments)
        {
            Instrument i = InstrumentManager.Get(symbol); //检查合约的信息是否已经在OpenQuant中引入
            if (i == null)
            {
                Console.Write("【"+symbol+"】该合约信息尚未引入OpenQuant！");
            }
            strategy.Instruments.Add(i);//引入合约到主策略工程
        }


        ... ... 


        //定义主策略中使用的合约的数据源和交易通道
        strategy.DataProvider = ProviderManager.GetDataProvider("QuantBoxCTP");
        strategy.ExecutionProvider = ProviderManager.GetExecutionProvider("QuantBoxCTP");

        //定义K线类型：时间型K线，barSize秒生成一个Bar
        BarFactory.Clear();

        foreach (string symbol in myInstruments)
        {
            Instrument i = InstrumentManager.Get(symbol);
            BarFactory.Add( i, BarType.Time, barSize);
        }

        //定义一个TickBar
        BarFactory.Add(InstrumentManager.Get("IF1711"), BarType.Tick,1);

        Initialize();

        ... ...
```

* **更好的方式，使用数据库**

Coming soon... ...

