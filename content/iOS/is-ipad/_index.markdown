---
layout: single
title:  "判断当前设备是否是iPad"
date:   2017-05-09 15:42
categories: iOS开发
---

####  判断当前设备是否是iPad

    1.不靠谱的判断
    ```objc
      [UIDevice currentDevice].userInterfaceIdiom==UIUserInterfaceIdiomPad
    ```

      判断是否是iPad不靠谱，如果是只支持iPhone的程序，在iPad上运行了，这个判断是NO。

    2.靠谱的判断  
    ```objc
      [[UIDevice currentDevice].model isEqualToString:@"iPad"]  
    ```
