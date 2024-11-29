
> Name

牛熊小卖部策略V10_OKex合约

> Author

区班量化

> Strategy Description

前面文章讨论过，小卖部策略在特定情况下，可以做到130%的年化收益。不过这是在特定情况下，就是该商品的月度均线，从长期看是上升的，且经过了很长一段时间。如果在进入市场的短期3个月里，都处于下跌市里，那可能遭到重大打击。
究其原因，最重要的是，该策略没引入做空机制。犹如在A股市场里，只能做多，那下跌来时，只能空仓或者被动挨揍。还好数字货币是提供做空机制的，通过合约交易，我们就可以把下跌风险规避掉。
今天区班主做的策略是牛熊小卖部策略。主要思想是牛市来了，就做多，同时抬高公允价格的预期；熊市来了，就做空，同时降低公允价格的预期。猴市来了则高抛低吸。牛熊猴的判断参考之前的文章，就是以短期高低点和长期高低点的关系，决定当前市场状态。
本策略应用于ETH，这里需要指出两个坑：1、本策略基于ETH，ETH在OKex期货里，是10刀一张的；需要BTC的朋友请自己修改一下除数；2、合约下单后，如果是相同多单，会合并成一个Position，所以Position的长度最多为2，这也是导致区班主当时反复调试的地方，还好最后解决了。
注册币乎后https://m.bihu.com/signup?i=1ewtKO&s=4&c=4
搜索 物联网区块链 可以联系到作者区班主

> Strategy Arguments



|Argument|Default|Description|
|----|----|----|
|Interval|10|轮询周期(秒)|
|mnum|20|30分钟线周期|
|initRatio|0.5|初始仓位比例|
|dnum|5|日线周期|


> Source (javascript)

``` javascript
/*backtest
start: 2019-01-01 00:00:00
end: 2019-10-10 00:00:00
period: 1d
exchanges: [{"eid":"OKEX","currency":"ETH_USDT","stocks":0}]
args: [["OpMode",1,10989],["MaxAmount",1,10989],["TradeFee",0.001,10989]]
*/
//注册币乎后https://m.bihu.com/signup?i=1ewtKO&s=4&c=4
//搜索 物联网区块链 可以联系到作者区班主
var status = 10; //10表示猴市初始化,11表示猴市继续;20表示牛市初始化,21表示牛市继续;30表示熊市初始化,31表示熊市继续
var dhigh;
var dlow;
var mlow;
var mhigh;
    
var operPrice;
function monkeyOper(){
   var i;
   var position;
   var account = _C(exchange.GetAccount);
   var ticker = _C(exchange.GetTicker);
   var nowPrice=ticker.Sell;
   var pAmount;
    
  /* if(status==10){ //进入猴市初始化,设定公允价格
       operPrice=mlow+mhigh;
   }else{ 
       if(nowPrice<operPrice*0.97){ //值得买
            //买平所有空单
            position = _C(exchange.GetPosition);
            for (i = 0; i < position.length; i++) {
               if(position[i].Type==PD_SHORT){ //买平空单
                 exchange.SetDirection("closesell"); 
                 exchange.Buy(nowPrice,position[i].Amount);
               }else{
                 pAmount=position[i].Amount;
               }
           }
          
           if(pAmount*10<account.Stocks*nowPrice){ //最多持半仓
              exchange.Buy(nowPrice,Math.floor(nowPrice*account.Stocks*0.1/10)); //尝试做多
              Log("猴市买入",account.Stocks*0.1);
              operPrice=nowPrice;
           } 
       }else if(nowPrice>operPrice*1.03){ //值得卖
            //卖平所有多单
            position = _C(exchange.GetPosition);
            for (i = 0; i < position.length; i++) {
               if(position[i].Type==PD_LONG){ //卖平多单
                 exchange.SetDirection("closebuy"); 
                 exchange.Sell(nowPrice,position[i].Amount);
               }else{
                 pAmount=position[i].Amount;
               }
           }
          
           if(pAmount*10<account.Stocks*nowPrice){ //最多持半仓
              exchange.Sell(nowPrice,Math.floor(nowPrice*account.Stocks*0.1/10)); //尝试做多
              Log("猴市卖出",account.Stocks*0.1);
              operPrice=nowPrice;
           } 
       }    
   }*/
}

function bullOper(){
   //去掉所有挂的空仓
   var orders = _C(exchange.GetOrders);
   var account = _C(exchange.GetAccount);
  
   for (var i = 0; i < orders.length; i++) {
       var order=orders[i];
       if(order.type==1){  //空单
           exchange.CancelOrder(order.Id);
           Log("清空单");
       }
   }
   
   var ticker = _C(exchange.GetTicker);
   var nowPrice=ticker.Sell;
   //买平所有空单
   var position = _C(exchange.GetPosition);
   var pAmount=0;
   for (i = 0; i < position.length; i++) {
       if(position[i].Type==PD_SHORT){ //买平空单,注意成交方向相反
           exchange.SetDirection("closesell"); 
           exchange.Buy(nowPrice,position[i].Amount);
       }else{
           pAmount=position[i].Amount;
       }
   }
   
   if(status==20){  //当前价格做多
       exchange.SetDirection("buy"); 
       if(pAmount*10<account.Stocks*nowPrice){ //最多持半仓
          exchange.Buy(nowPrice,Math.floor(nowPrice*account.Stocks*0.2/10)); //尝试做多
          Log("初始买入",account.Stocks*0.1);
       } 
       operPrice=nowPrice;
   }else if(nowPrice<operPrice*0.97){  //最多挂两单，大力做多
       exchange.SetDirection("buy");
       if(pAmount*10<account.Stocks*nowPrice){ //最多持半仓
          exchange.Buy(nowPrice,Math.floor(nowPrice*account.Stocks*0.3/10));
          Log("加大买入",account.Stocks*0.3);
       }
       operPrice=nowPrice;
   }
}

function bearOper(){
   //去掉所有挂的多仓
   var orders = _C(exchange.GetOrders);
   var account = _C(exchange.GetAccount);
  
   for (var i = 0; i < orders.length; i++) {
       var order=orders[i];
       if(order.type==0){  //多单
           exchange.CancelOrder(order.Id);
           Log("清多单");
       }
   }
   
   var ticker = _C(exchange.GetTicker);
   var nowPrice=ticker.Sell;
   //买平所有多单
   var position = _C(exchange.GetPosition);
   var pAmount=0;
   for (i = 0; i < position.length; i++) {
       if(position[i].Type==PD_LONG){ //卖平多单
           exchange.SetDirection("closebuy"); 
           exchange.Sell(nowPrice,position[i].Amount);
       }else{
           pAmount=position[i].Amount;
       }
   }
   
   if(status==30){  //当前价格做空
       exchange.SetDirection("sell"); 
       if(pAmount*10<account.Stocks*nowPrice){ //最多持半仓
          exchange.Sell(nowPrice,Math.floor(nowPrice*account.Stocks*0.2/10)); //尝试做空
          Log("初始卖出",account.Stocks*0.1);
       } 
       operPrice=nowPrice;
   }else if(nowPrice>operPrice*1.03){  //最多挂两单，大力做多
       exchange.SetDirection("sell");
       if(pAmount*10<account.Stocks*nowPrice){ //最多持半仓
          exchange.Sell(nowPrice,Math.floor(nowPrice*account.Stocks*0.3/10));
          Log("加大卖出",account.Stocks*0.3);
       }
       operPrice=nowPrice;
   }
}

function oper(){
    var ticker = _C(exchange.GetTicker);
    var nowPrice=ticker.Sell;
    
    var drecords = exchange.GetRecords(PERIOD_D1);
    var mrecords = exchange.GetRecords(PERIOD_M30);
    //日线5天内的高低点(不包含当前Bar)
    dhigh=TA.Highest(drecords, dnum, 'High');
    dlow=TA.Lowest(drecords, dnum, 'Low');
       
    //30分钟线10个周期内的高低点(不包含当前Bar)
    mhigh=TA.Highest(mrecords, mnum, 'High');
    mlow=TA.Lowest(mrecords, mnum, 'Low');
    
    if(mhigh>dhigh&&mlow<dlow){ //如果分钟高低点都突破了日线高低点，看重心在哪，决定是什么市
        if((mhigh+mlow)<(dhigh+dlow)*0.97){
            if(status==30){
              status=31;
            }else{
              status=30; 
            }
            bearOper();
        }else if((mhigh+mlow)>(dhigh+dlow)*1.03){
            if(status==20){
              status=21;
            }else{
              status=20; 
            }
            bullOper();
        }else{
            if(status==10){
              status=11;
            }else{
              status=10;
            }
            monkeyOper();
        }
    }else if(mhigh>dhigh){ //分钟低点突破日高点，牛开始
        if(status==20){
           status=21;
        }else{
           status=20; 
        }
        bullOper();
    }else if(mlow<dlow){  //分钟低点跌破日低点，熊开始
        if(status==30){
           status=31;
        }else{
           status=30;
        }
        bearOper();
    }else{  //没有方向，猴市
        if(status==10){
           status=11;
        }else{
           status=10;
        }
        monkeyOper();
    }
}

function main() {
    var initAccount = _C(exchange.GetAccount);
    Log(initAccount);
    exchange.SetContractType("quarter")    // 举例设置为OKEX期货当周合约
    exchange.SetMarginLevel(5);              // 设置杠杆为5倍
    while (true) {
        oper();
        Sleep(Interval*1000);
    }
}
```

> Detail

https://www.fmz.com/strategy/171038

> Last Modified

2019-10-24 13:44:56
