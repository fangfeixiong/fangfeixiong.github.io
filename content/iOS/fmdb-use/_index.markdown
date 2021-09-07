---
layout: single
title:  "FMDB的使用"
date:   2017-05-02 12:30
categories: iOS开发
---

####  FMDB的使用

  最近app中使用了苹果支付来充值，然后苹果支付结果是直接返回给客户端的，而不像支付宝和微信支付等支付平台，支付完成后会回调后台。

  苹果支付返回的结果是一个超级长的ReceiveData的字符串。然后得到字符串之后向自己的后台发请求，让自己的后台去请求苹果的服务器来校验结果。当然也可以客服端直接调用苹果的服务器来的到校验结果，然后发给自己的服务器，但是这显然是不靠谱的，容易被篡改。所以还是让自己的服务端去校验，才比较靠谱。

  这里我们会遇到一个问题，如果我们得到苹果的支付结果后，还没来得及发给自己的后端校验，遇到突然就断网了，app崩溃了，手机没电关机了等情况。那我们得到的支付结果ReceiveData就直接丢失了，再也找不到了，用户充了钱，却没有得到应有的虚拟货币，那就尴尬了。

  所以了我们就想到一个方案，收到苹果的支付结果后，先把他存储在本地。然后向自己的后端发出请求，如果自己的服务端校验完成，返回结果后，就把本地存储的对应的记录删除掉。

  每次app开启的时候，查询所有未校验的支付结果，依次发给自己的服务端进行校验，这样就不会漏掉了。当然数据可以以文件的形式存储，但是作者觉得用数据库存储可能更方便一些，以后可能也会有其他的数据需要存储，所以就决定学习一下FMDB的使用。FMDB可以直接通过CocosPod 下载集成，很方便。

  #### 先定义一些常量，比如数据库名、表名、列名，方便后面操作的引用。

``` objc

  //数据库名
  #define DataBase_Name @"yourDataBaseName.sqlite"
  //表名
  #define TableName_Table1 @"table1"

  //列1 名称
  #define Table1_ColumnName1 @"columnName1"
  //列2 名称
  #define Table1_ColumnName2 @"columnName2"
  //列3 名称
  #define Table1_ColumnName3 @"columnName3"
```

  #### 1.创建数据库
  
  ``` objc
   NSString *documentPath=[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
   NSString *databasePath=[documentPath stringByAppendingPathComponent:DataBase_Name];

   FMDatabase *database=[FMDatabase databaseWithPath:databasePath];

  ```

  #### 2.创建表
  ``` objc
  if([database open])
{
  BOOL createTableResult=[database executeUpdate: [NSString stringWithFormat:@"create table %@ (%@ text,%@ text,%@ integer)",TableName_Table1,Table1_ColumnName1,Table1_ColumnName2,Table1_ColumnName3]];
  [database close];
}
  ```

  #### 3.插入数据
  ``` objc
  if([database open])
{
  BOOL insertResult=[database executeUpdate:[NSString stringWithFormat:@"insert into %@ (%@,%@,%@) values ('%@','%@',%ld)",TableName_Table1,Table1_ColumnName1,Table1_ColumnName2,Table1_ColumnName3,@"testColomnValue1",@"testColomnValue2",10001]];
    [database close];
}
  ```

  #### 4.获取数据
  ``` objc
  if([database open])
{
    FMResultSet *resultSet= [database executeQuery:[NSString stringWithFormat:@"select * from %@ where %@=%ld",TableName_Table1,Table1_ColumnName3,10001]];
    if(resultSet==nil)
    {
        NSLog(@"select result is null");
        return payOrderList;
    }
    while([resultSet next])
    {
       NSString *columnName1Value=[resultSet stringForColumn:Table1_ColumnName1];
       NSString *columnName2Value=[resultSet stringForColumn:Table1_ColumnName2];
    }
    [database close];
}
  ```
  注意：FMResultSet 其实是对数据库的一个引用，必须在数据库开启的时候执行next方法，才能逐条获取导数据。如果关闭后执行next，是获取不到数据的。所以我们应该将FMResultSet的数据逐条获取出来，存储到自己的数据结构中，然后关闭数据库。

  #### 5.删除数据
  ``` objc
  if([database open])
{
    BOOL deleteResult=[database executeUpdate:[NSString stringWithFormat:@"delete from %@ where %@='%@' and %@=%ld",TableName_Table1,Table1_ColumnName1,@"testColomnValue1"];
    [database close];
}
  ```


  [我的FMDB练习地代码](https://github.com/PlayLive/Practice/tree/master/FMDBUse)
