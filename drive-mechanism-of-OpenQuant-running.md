## OpenQuant策略运行的事件驱动机制

OpenQuant的策略运行机制主要是构造出扑捉市场数据的处理逻辑后，静静地等待各种事件发生，来驱动整体逻辑的运行。例如：当订阅的合约有最新报价变动时，或当行情的变动幅度超过定义时，当下单部分成交时，或发出的撤单指令成功时等等，当然有的时候也需要自己依据定时间隔去进行驱动。由于这种处理机制，可以让量化交易者精细地定义逻辑并清晰地执行。

下面就是一个成交事件处理的代码，你可以在SMACrossover策略解决方案中的MyStrategy工程的MyStrategy.cs代码中找到：

```
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

}
```

## OpenQuant策略运行的时间驱动机制

往往很多原因，我们不能只是让策略在被动等待事件，有时候需要进行主动地进行运行某些逻辑，例如，我们需要定时对系统状态及交易进行状态检查，所以我们还需要构造按照时间定时去运行的驱动机制。

使用上述的Reminder事件构造一个循环的定时器即可完成这个功能：

```
protected override void OnReminder(DateTime dateTime, object data)
{
    CheckMyBoxStatus();
    CheckMyAccountInfo();

    AddReminder(Clock.DateTime.AddSeconds(3));
}
```

##### 

![](/icons/icon_book.png)[附：OpenQuant中的事件类型](/the-events-in-openquant.md)



