# 开始一个HelloWorld

---

OpenQuant的开发体系是基于标准的微软C\#编程语言及.NET开发框架。

为了避免文字内容过于冗长，我们认为读者已经可以使用C\#编程语言并使用过Visual Studio开发工具。

## 创建第一个解决方案（C\# solution）

![](/icons/icon_labtubeBlue.ico)打开OpenQuant中的**File**菜单，**New**一个新的**solution**, 这里有一些解决方案类别选项，这个例子中，我们选择第一种“SmartQuant Instrument Strategy Solution”， 命名为_myFirstStrategy_， 保持目录应该保存在你的SVN代码版本管理体系中, 生成的解决方案（Solution）结构如下：

![](/assets/myFirstStrategyTreeDos.png)      ![](/assets/myFirstStrategyFilesTreeOQ.png)

图：一个默认解决方案包含的文件结构

在Instrument Strategy Solution类型默认代码中，主要有3个核心文件：

-**Backtest目录中的**

-Program.cs -------------------------- 整个solution的入口文件，其中有整个策略代码的入口 Main函数

-MyScenario.cs --------------------- 场景定义文件，可以定义合约订阅，定义回测起始结束日期或者定义实盘的交易通道

-**MyStrategy目录中的**

-MyStrategy.cs -----------------------策略核心文件，编写所有的核心策略逻辑

## OpenQuant的解决方案中代码的运行关系

这是一个标准的微软C\#解决方案，在这个**myFirstStrategy**解决方案（Solution）中有两个项目（Project）：Backtest，MyStrategy，Backtest项目是默认的启动项目。代码关系如下图的所示：

![](/assets/myFirstStrategyCodeMap.png)        `图： OpenQuant生成的策略代码的结构及调用关系`

![](/icons/icon_labtubeBlue.ico)为了掌握OpenQuant解决方案代码运行中的调用关系，我们在该项目文件中增加输出信息，使得程序运行时，可以呈现代码运行顺序和调用过程。

在**Program.cs **中添加代码：

```
using System;

using SmartQuant;

namespace OpenQuant
{
    class Program
    {
        static void Main(string[] args)
        {
                       System.Console.WriteLine("Program.cs程序主入口： Main()");

            System.Console.WriteLine("Program.cs程序主入口中, 实例化MyScenario()");
                       Scenario scenario = new MyScenario(Framework.Current);

                       System.Console.WriteLine("Program.cs程序主入口中, 运行Run()");
            scenario.Run();
        }
    }
}
```

在**MyScenario.cs **中添加代码：

```
using System;

using SmartQuant;

namespace OpenQuant
{
    public partial class MyScenario : Scenario
    {
        public MyScenario(Framework framework)
            : base(framework)
        {
            System.Console.WriteLine("MyScenario.cs 我的场景开始定义");
        }

        public override void Run()
        {
            strategy = new MyStrategy(framework, "Backtest");
            System.Console.WriteLine("MyScenario.cs 我的场景MyScenario.Run()中, 实例化我的策略 new MyStrategy() ");

        System.Console.WriteLine("MyScenario.cs 我的场景MyScenario.Run()中, 开始初始化过程Initialize() ");
        Initialize();

            System.Console.WriteLine("MyScenario.cs 我的场景MyScenario.Run()中, 开始策略StartStrategy()");
            StartStrategy();
        }
    }
}
```

在**MyStrategy.cs **中添加代码：

```
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
            System.Console.WriteLine("MyStrategy.cs 我的策略MyStrategy中OnStrategyStart()");
        }

        protected override void OnBar(Instrument instrument, Bar bar)
        {
            System.Console.WriteLine("MyStrategy.cs 我的策略MyStrategy中OnBar()事件");
        }
    }
}
```

![](/icons/icon_labtubeOrg.ico)添加完成后，直接运行策略，我们可以中OpenQuant的Output窗口中看到如下输出：

![](/assets/HelloWorldOutput01.png)

这样，我们很清楚地看出OpenQuant的策略运行过程，先是场景定义和初始化过程，然后实例化我的策略并运行。但策略中由于没有数据请求（data requests）, OpenQuant的回测模式就即可运行完成了整个策略过程。如果将运行模式切换至仿真Paper或实盘Live模式，策略就会一直处于等待市场数据状态，并不会自动退出，直到手动停止。

对于一个完整的程序化交易策略中，只有一个Backtest部分和一个策略逻辑主体部分，这显然是有问题的。让我们先把要做的Hello World策略放在一旁，我们进入下一章，来看看一个稍微完整的OpenQuant策略示例代码的结构。

