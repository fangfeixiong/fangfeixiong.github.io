---
layout: single
title:  "苹果内购流程"
date:   2017-06-04 15:00
categories: iOS开发
---

####  登录iTunesconnect

1.登录[iTunes Connect](https://itunesconnect.apple.com/)

####  协议、税务和银行业务 信息完善

1.选择协议、税务和银行业务项，如下图所示
  ![选择协议、税务和银行业务的图片](selectPanel.png)

2.选择Request
  ![协议、税务和银行业务界面](protocolTaxBank.png)

  按照一步步操作即可,等待审核（需要一点时间可以过一天再看），完毕后即可得到上图Contracts In Effects的一条PaidApplication信息！

3.编辑联系信息
  ![联系信息界面](contactInformation.png)

  所有的信息填写同一个人即可


4.税务信息
  ![税务信息](taxInformation-01.png)
  ![税务信息](taxInformation-02.png)
  ![税务信息](taxInformation-03.png)
  ![税务信息](taxInformation-04.png)

5.银行信息填写
  CNAPS Code [银行网点联行号 查询网址](http://www.tui78.com/bank/) 需要哪个银行直接去对应银行链接里面找就行

  ![银行信息](bankingInformation.png)

  所有信息填写完毕后我们就可以开始编辑内购项

#### 编辑内购项

1.进入iTunes Connect 首界面，选择我的App，选中你需要加入内购的App。

2.内购项目
  ![内购完成图片](purchaseItems.png)

  点击+按钮添加自己的内购项目

  比如我们的应用是虚拟货币，所以我们选择消耗型,根据需要选择对应的类型，如下图

  ![内购完成图片](selectPurchasePanel.png)

  填写内购项的信息
  ![内购项信息1](purchaseItem1-01.png)
  ![内购项信息2](purchaseItem1-02.png)

  这里注意图片对应app内部的相应的选项
  ![内购项信息3](purchaseItem1-03.png)


#### 内购代码

1.从自己的服务器上获取充值选项数据，用于展示选择项，每一项对应一个内购项目,带有内购项的ID信息

  ![内购选项](rechargePanel.PNG)

2.向苹果服务器请求内购信息，根据内购项的ID列表

 ```objc
  -(void)requestAppleProducts{
      SKProductsRequest *productsRequest=[[SKProductsRequest alloc] initWithProductIdentifiers:[NSSet setWithArray:@[你的内购项的ID列表]];
      productsRequest.delegate=self;
      [productsRequest start];
  }


  -(void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response
  {
      NSArray<SKProduct*> *products=[response.products sortedArrayWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(SKProduct obj1, SKProduct obj2) {
          return [obj1.price compare:obj2.price];
      }];
  }

```

  3.选择一个充值项，充值调用
  ```objc

//请求充值
  -(void)requestPurchase
  {
    //监听内购处理过程
    [[SKPaymentQueue defaultQueue] addTransactionObserver:self];

    SKProduct *product=选中的项对应的product;
    SKPayment *payment=[SKPayment paymentWithProduct:product];
    [[SKPaymentQueue defaultQueue] addPayment:payment];//执行内购
  }


//内购支付结果处理
  -(void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray<SKPaymentTransaction *> *)transactions
{
    for(SKPaymentTransaction *transation in transactions){
        switch (transation.transactionState) {
            case SKPaymentTransactionStatePurchasing:
                NSLog(@"正在唤起支付，请稍候");
                break;
            case SKPaymentTransactionStatePurchased:
                [queue finishTransaction:transation];
                [self handleTransaction:transation];
                NSLog(@"充值成功")
                break;
            case SKPaymentTransactionStateFailed:
                [queue finishTransaction:transation];
                NSLog(@"充值失败");
                break;
            case SKPaymentTransactionStateRestored:
                [queue finishTransaction:transation];
                [self handleTransaction:transation];
                NSLog(@"充值失败");
                break;
            case SKPaymentTransactionStateDeferred:
                NSLog(@"等待充值结果");
                break;
            default:
                NSLog(@"未知错误");
                break;
        }
    }
}

-(void)handleTransaction:(SKPaymentTransaction *)transaction{
    NSData *receiptData=[NSData dataWithContentsOfURL:[[NSBundle mainBundle] appStoreReceiptURL]];
    //先把将app内购的结果存储起来，如果出现了意外情况，断网了等没有将结果发送到自己的服务器，那么下次app打开时，可以查询本地没有校验的存储，然后发送给自己的服务端校验！

    //我的ApplePayDataManager 本地存储帮助类
    [[ApplePayDataManager Instance] insertPayOrderNo:transaction.transactionIdentifier receiveData:[receiptData base64EncodedStringWithOptions:0]];

    //把receiptData 发送给自己的服务器，服务器去调用苹果的接口验证，如果校验完毕从本地存储中删除存储的对应的ReceiveData记录。
}


```

#### 内购的提交审核

1.  如果是第一次内购提交审核，那么App提交审核界面上会多出内购项添加的模块，要把需要的所有内购项都添加进来，和App一起提交审核，不然App什么是没法通过的。

2.  非常重要，切记把需要的内购项目都添加到提交审核页面对应的块中！


#### 校验结果对照
![key的解释](response-01.png)
![status的解释](response-02.png)

#### 内购的附加信息

  [我的ApplePayDataManager类](https://github.com/PlayLive/Practice/tree/master/FMDBUse)

  [App内购官方参考](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Introduction.html)

  [App内购流程官方参考](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html#//apple_ref/doc/uid/TP40010573-CH104-SW1)

  处于测试阶段的ReceiveData校验地址： https://sandbox.itunes.apple.com/verifyReceipt

  正式上线后的ReceiveData校验地址： https://buy.itunes.apple.com/verifyReceipt
