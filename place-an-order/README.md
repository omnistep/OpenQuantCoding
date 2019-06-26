# 7. 发出交易指令

## 7.1建立一个发出交易指令的策略FlashOrder {#flashorder}

在OpenQuant中，基础的交易指令代码如下：

... ... 

```text
using QuantBox;
```

... ... 

```text
//定义报单数量myOrdQty, 定义报单价格 myOrdPxU:当前价格加3跳; myOrdPxL:当前价格减3跳
int     myOrdQty = 1;
double  myOrdPxU = bar.Close + instrument.TickSize*3;
double  myOrdPxU = bar.Close - instrument.TickSize*3;
```

```text


//限价买入开仓-△
Order myBuyLimitOrder = BuyLimitOrder(instrument, myOrdQty, myOrdPxU,"买开△");
Send(myBuyLimitOrder); 
            
//限价卖出平仓-▼    
Order mySellLimitOrder = SellLimitOrder(instrument, myOrdQty, myOrdPxU , "卖平▼");
mySellLimitOrder.Close();
Send(myBuyLimitOrder); 
            
//限价卖出开仓-▽
Order mySellLimitOrder = SellLimitOrder(instrument, myOrdQty, myOrdPxU , "卖开▽");
mySellLimitOrder.SubSide = SubSide.SellShort;
Send(myBuyLimitOrder); 

//限价买入平仓-▲
Order myBuyLimitOrder = BuyLimitOrder(instrument, myOrdQty, myOrdPxU,"买开▲");
myBuyLimitOrder.SubSide = SubSide.BuyCover;
myBuyLimitOrder.Close();
Send(myBuyLimitOrder);
```



















现在我们来构建一个新的交易策略代码。在这个策略中我们将实现：

* [x] 发出期货交易中限价定单的交易指令：做多交易的买入开仓、卖出平仓；做空交易的卖出开仓、买入平仓。
* [x] 在OpenQuant 行情数据图表中标识开仓记号，在OpenQuant图表中显示账户的权益曲线。
* [x] 新增加一个日志功能，将所有的交易记录存入MySQL数据库。

我们把这个交易策略命名为FlashOrder

![](../.gitbook/assets/icon_labtubeblue%20%2810%29.ico)使用OpenQuant系统建立一个新的策略，并命名为FlashOrder

## 7.2 FlashOrder的策略逻辑 {#typeoforder}

这里的逻辑仅仅是为了使用让OpenQuant高速地发出开仓及平仓单，进行交易功能的验证。

在交易执行层面，我们只要处理四种定单的场景，即：  
1. 买入开仓定单，简记符号为 △  
2. 卖出平仓定单，简记符号为 ▼  
3. 卖出开仓定单，简记符号为 ▽  
4. 买入平仓定单，简介符号为 ▲

![](../.gitbook/assets/icon_paw%20%283%29.png)使用定单简记符号将使得研究交易策略，或输出运行日志时有很好的可阅读性。而且这组符号很容易识别和记忆。

在这个策略中，我们还需要处理两个事件：行情进入时的OnBar事件，和报单成交时的OnFill事件。为了测试上述4种定单，我们在OnBar事件发生时，将报出买入开仓△，和卖出开仓▽，当定单成交后，马上发出平仓单，为了控制每次OnBar事件发生时候，都发出开仓定单，导致资金消耗过多，我们用全局变量控制是否允许开仓进场，swLongEnable控制做多方向，swShortEnable控制做空方向。FlashOrder的总体策略逻辑如下图所示：

![](../.gitbook/assets/flashorderstrategydiagram.png)

