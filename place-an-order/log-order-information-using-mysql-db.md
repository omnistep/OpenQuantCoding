# 7.4 将交易记录存入MySQL

对于一个完善的IT系统，无论是交易系统，还是业务管理系统，我们都会按照层次结构来规划。作为一个交易系统，我们需要将交易日志存入数据，以提供业务监控、问题排查、数据统计分析。

现在我们就将FlashOrder下单的记录全部存入Mysql数据库。

![](../.gitbook/assets/icon_labtubeblue%20%289%29.ico)大致过程分为五步，第一步建立数据库及日志表；第二步在FlashOrder中引入相关的MySQL动态链接库；第三步编写一个日志存储功能的方法，让主策略代码可以调用日志功能。这个文件我们命名为OmniLog.cs，这个文件我们和主策略代码放在一个工程代码中； 第四步，我们在主策略的下单代码后面，插入调用日志存储的方法；第五步，使用MySQL工具及性能工具进行日志存储的查看和监控。

* **第一步-安装MySQL数据库及建立交易日志表**

第一步，我们安装好MySQL数据库，并建立交易报单日志表LogOrder表，建表SQL如下：

```text
CREATE TABLE `logorder` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `logdatetime` DATETIME(6) NULL DEFAULT NULL COMMENT 'log记录时间',
    `StrategyName` VARCHAR(20) NULL DEFAULT NULL COMMENT '策略名字' COLLATE 'utf8mb4_unicode_ci',
    `OrdId` INT(11) NULL DEFAULT NULL,
    `OrdDateTime` DATETIME(6) NULL DEFAULT NULL,
    `Symbol` VARCHAR(10) NULL DEFAULT NULL COLLATE 'latin1_swedish_ci',
    `TradeDay` DATE NULL DEFAULT NULL,
    `Side` VARCHAR(10) NULL DEFAULT NULL COLLATE 'latin1_swedish_ci',
    `Type` VARCHAR(10) NULL DEFAULT NULL COLLATE 'latin1_swedish_ci',
    `Qty` INT(11) NULL DEFAULT NULL,
    `Price` FLOAT NULL DEFAULT NULL,
    `Status` VARCHAR(16) NULL DEFAULT NULL COLLATE 'latin1_swedish_ci',
    `note` VARCHAR(50) NULL DEFAULT NULL COMMENT '备注' COLLATE 'utf8mb4_unicode_ci',
    PRIMARY KEY (`id`)
)
COMMENT='定单日志表，存储所有交易记录'
COLLATE='utf8mb4_unicode_ci'
ENGINE=InnoDB
AUTO_INCREMENT=72969
;
```

* **第二步-在策略工程中引入MySQL的DLL库及建立MySQL访问代码**

在FlashOrder解决方案的在MyStrategy工程中，先引入Mysql的.NET驱动库：**MySql.Data.dll** 。这个动态链接库文件你可以在MySQL的安装目录中也可以找到。

再在MyStrategy工程中增加MySQL访问功能的公共方法代码，文件名MysqlMan.cs ，该文件代码如下：

```text
/*
 *      Purpose:    MySQL DB Access  
 *          
 *      Created:    2017/09/12 
 * 
 * 
 */

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using MySql.Data.MySqlClient;


namespace OpenQuant
{
    class MysqlMan
    {


        /// <summary>
        /// 建立mysql数据库链接
        /// </summary>
        /// <returns></returns>
        public static MySqlConnection getMySqlCon()
        {
            String mysqlStrQBMD = "";
            //Mysql Server at Lab
            mysqlStrQBMD = "Database=MySQL01;Data Source=127.0.0.1;User Id=root;Password=yourPassword;pooling=false;CharSet=utf8;port=3306;Allow Zero Datetime=True";
            //MySQL Server at Home
            //mysqlStrQBMD = "Database=MySQL02;Data Source=127.0.0.1;User Id=root;Password=yourPassword;pooling=false;CharSet=utf8;port=3306;Allow Zero Datetime=True";



            MySqlConnection mysql = new MySqlConnection(mysqlStrQBMD);
            return mysql;
        }

        /// <summary>
        /// 建立执行命令语句对象
        /// </summary>
        /// <param name="sql"></param>
        /// <param name="mysql"></param>
        /// <returns></returns>
        public static MySqlCommand getSqlCommand(String sql, MySqlConnection mysql)
        {
            MySqlCommand mySqlCommand = new MySqlCommand(sql, mysql);
            return mySqlCommand;
        }

        /// <summary>
        /// 添加数据
        /// </summary>
        /// <param name="mySqlCommand"></param>
        public static void getInsert(MySqlCommand mySqlCommand)
        {
            try
            {
                mySqlCommand.ExecuteNonQuery();
            }
            catch (Exception ex)
            {
                String message = ex.Message;
                Console.WriteLine("插入数据失败了！" + message);

            }

        }

        /// <summary>
        /// 删除数据
        /// </summary>
        /// <param name="mySqlCommand"></param>
        public static void getDel(MySqlCommand mySqlCommand)
        {
            try
            {
                mySqlCommand.ExecuteNonQuery();
            }
            catch (Exception ex)
            {
                String message = ex.Message;
                Console.WriteLine("删除数据失败了！" + message);
            }
        }

        /// <summary>
        /// 修改数据
        /// </summary>
        /// <param name="mySqlCommand"></param>
        public static void getUpdate(MySqlCommand mySqlCommand)
        {
            try
            {
                mySqlCommand.ExecuteNonQuery();
            }
            catch (Exception ex)
            {

                String message = ex.Message;
                Console.WriteLine("修改数据失败了！" + message);
            }
        }



        public static int ExecuteNonQuery(string sql) 
        {
            MySqlConnection mysql = null;
            try
            {
                mysql = getMySqlCon();
                mysql.Open();
                MySqlCommand mySqlCommand = new MySqlCommand(sql, mysql);
                int result = mySqlCommand.ExecuteNonQuery();
                mysql.Close();
                return result;
            }
            catch (Exception ex)
            {
                if (mysql != null && mysql.State != System.Data.ConnectionState.Closed)
                    mysql.Close();
                String message = ex.Message;
                Console.WriteLine("ExecuteNonQuery失败了！" + message);
                return 0;
            }
        }

        public static object ExecuteScalar(string sql) 
        {
            MySqlConnection mysql=null;
            try
            {
                mysql = getMySqlCon();
                MySqlCommand mySqlCommand = new MySqlCommand(sql, mysql);
                mysql.Open();
                object result = mySqlCommand.ExecuteScalar();
                mysql.Close();
                return result;

            }
            catch (Exception ex)
            {
                if(mysql!=null && mysql.State != System.Data.ConnectionState.Closed)
                    mysql.Close();
                String message = ex.Message;
                Console.WriteLine("ExecuteNonQuery失败了！" + message);
                return null;
            }
        }
    }
}
```

* **第三步-编写日志存储功能OmniLog.cs**

在FlashOrder解决方案的MyStrategy工程中，增加日志存储功能OmniLog.cs文件，该文件负责提供报单日志存储方法，以便策略中可以方便地调用，保存报单日志，OmniLog.cs的代码如下：

```text
/*
 *      Purpose:    OmniLog   
 *      
 *      version:    1.0.0 
 *      create :    2017/09/19 
 *      by     :    ww 
 *
 *     //▓▓记录日志 OmniLog.logOrder(StrategyName,OrdId, DateTime.Now,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note)
 *              
 *     select id,date_format(logdatetime,'%Y-%m-%d %H:%i:%s:%f'),strategyname,ordid,date_format(orddatetime,'%Y-%m-%d %H:%i:%s:%f'),symbol,tradeday,side,type,qty,price,status,note  from logorder order by id desc 
 *  
 */

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using SmartQuant;

namespace OpenQuant
{
    public class OmniLog 
    {

        /// <summary>
        /// 记录定单 logOrder 
        /// </summary>
        /// <param name="OrdId"></param>
        public static void logOrder(String StrategyName,int OrdId,DateTime OrdDateTime,String Symbol,String Side,String Type,double Qty,double Price,String Status,String Note)
        {
            try
            {

                string sqlInsert = string.Format("insert into LogOrder(LogDateTime,StrategyName,OrdId,OrdDateTime,Symbol,side,type,qty,price,status,note) values('{0:yyyy-MM-dd HH:mm:ss.fff}','{1}','{2}','{3:yyyy-MM-dd HH:mm:ss.fff}','{4}','{5}','{6}','{7}','{8}','{9}','{10}')",
                    DateTime.Now,StrategyName,OrdId,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note);
                int row = MysqlMan.ExecuteNonQuery(sqlInsert);                


            }
            catch (Exception ex)
            {
                String message = ex.Message;
                Console.WriteLine("数据保存失败了！" + message);

            }

        }



    }
}
```

* **第四步-在主策略逻辑中增加日志存储代码**

现在我们可以在主策略逻辑中保存报单日志了，报单日志保存的调用方法是：

```text
OmniLog.logOrder(StrategyName,OrdId, DateTime.Now,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note)
```

FlashOrder的主策略代码改为：

```text
using System;
using System.Drawing;
using SmartQuant;
using QuantBox;

namespace OpenQuant
{
    public class MyStrategy : InstrumentStrategy
    {
        //定义策略名字 StrategyName，将被记录在日志中
        String StrategyName="FlashOrder";            

        int orderQty = 1; //定义交易定单的发单量多少手
        int swLongEnable = 1; //定义做多交易许可开关，0：禁止，1：允许
        int swShortEnable = 1; //定义做空交易许可开关，0：禁止，1：允许

        int theOrdBOid = 0 ; //定义做多△买入开仓定单id
        int theOrdSCid = 0 ; //定义做多 ▼卖出平仓定单id
        string theOrdSCstatus = ""; //定义做多 ▼卖出平仓定单状态

        int theOrdSSid = 0 ; //定义做空▽卖出开仓定单id
        int theOrdBCid = 0 ; //定义做空 ▲买入平仓定单id

        int myCentOpen = -1 ;     //定义开仓追价成本是多少TickSize；
        int myCentClose = -1 ;     //定义平仓追价成本是多少TickSize；
                                //默认1：代表盈利1个TickSize挂单
                                // 0：代表市场平价报单；
                                // -1：代表1个TickSize的成本报单

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
            Console.WriteLine("swLongEnable= "+swLongEnable.ToString()+ "; swShortEnable ="+swShortEnable.ToString());
            //显示平仓单状态
            System.Console.WriteLine("theOrdSCstatus = " + theOrdSCstatus);

            if(swLongEnable==1)
            {
                //买入-开仓
                Order orderBO = BuyLimitOrder(Instrument, orderQty, bar.Close - myCentOpen*Instrument.TickSize, "△");
                //orderBO.Open();
                Send(orderBO);

                //▓▓记录日志OmniLog.logOrder(StrategyName,OrdId,DateTime.Now,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note)
                OmniLog.logOrder(StrategyName,orderBO.Id,orderBO.DateTime,orderBO.Instrument.Symbol,orderBO.Side.ToString() ,orderBO.Type.ToString() ,orderBO.Qty,orderBO.Price,orderBO.Status.ToString() ,"△");


                swLongEnable = 0;
                theOrdBOid = orderBO.Id;

            }


            if(swShortEnable==1)
            {
                //卖出-开仓
                Order orderSS = SellLimitOrder(Instrument, orderQty, bar.Close + myCentOpen*Instrument.TickSize, "▽");
                //orderSS.Open();
                Send(orderSS);

                //▓▓记录日志OmniLog.logOrder(StrategyName,OrdId,DateTime.Now,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note)
                OmniLog.logOrder(StrategyName,orderSS.Id ,orderSS.DateTime,orderSS.Instrument.Symbol,orderSS.Side.ToString() ,orderSS.Type.ToString() ,orderSS.Qty,orderSS.Price,orderSS.Status.ToString() ,"▽");


                swShortEnable = 0;
                theOrdSSid = orderSS.Id;

            }


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

            //▓▓记录日志OmniLog.logOrder(StrategyName,OrdId, DateTime.Now,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note)
            OmniLog.logOrder(StrategyName,fill.Order.Id ,fill.DateTime,fill.Instrument.Symbol,fill.Side.ToString() ,fill.Order.Type.ToString() ,fill.Qty,fill.Price,fill.Order.Status.ToString() ,"挂单成交");



            if(fill.Order.Id == theOrdBOid)
            {
                //卖出-平仓
                Order orderSC = SellLimitOrder(Instrument, orderQty, fill.Price + myCentClose*Instrument.TickSize, "▼");
                orderSC.CloseToday() ;
                Send(orderSC);

                theOrdSCid = orderSC.Id;
                theOrdSCstatus = orderSC.Status.ToString();

                swLongEnable = 1;

                //▓▓记录日志OmniLog.logOrder(StrategyName,OrdId, DateTime.Now,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note)
                OmniLog.logOrder(StrategyName,orderSC.Id ,orderSC.DateTime,orderSC.Instrument.Symbol,orderSC.Side.ToString() ,orderSC.Type.ToString() ,orderSC.Qty,orderSC.Price,orderSC.Status.ToString() ,"▼");

            }

            if(fill.Order.Id == theOrdSSid)
            {
                //买入-平仓
                Order orderBC = BuyLimitOrder(Instrument, orderQty, fill.Price - myCentClose*Instrument.TickSize, "▲");
                orderBC.CloseToday();
                Send(orderBC);
                System.Console.WriteLine("▓orderBC.SubSide=" + orderBC.SubSide.ToString());

                theOrdBCid = orderBC.Id;

                swShortEnable = 1;

                //▓▓记录日志OmniLog.logOrder(StrategyName,OrdId, DateTime.Now,OrdDateTime,Symbol,Side,Type,Qty,Price,Status,Note)
                OmniLog.logOrder(StrategyName,orderBC.Id ,orderBC.DateTime,orderBC.Instrument.Symbol,orderBC.Side.ToString() ,orderBC.Type.ToString() ,orderBC.Qty,orderBC.Price,orderBC.Status.ToString() ,"▲");

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

* **第五步-使用MySQL工具进行查看及监控**

![](../.gitbook/assets/icon_labtubeorg%20%283%29.ico)这时运行策略FlashOrder,当有报单时，OmniLog会将所有报单记录存入数据库的表LogOrder

在MySQL中记录定单日志如下：

![](../.gitbook/assets/table_logorder.png)

通过Mysql的Workbench工具可以监控数据库的状态，Dashboard工具如下图所示：

![](../.gitbook/assets/mysqldashboard01.png)我们会发现这样写日志的效率还是有些慢，但对于没有做高频交易（HFT）的情况，这些工具是适用的。

