# volatile的内存语义

## 目录

* [volatile的特性](/di-san-zhang-java-nei-cun-mo-xing/volatilenei-cun-yu-yi/volatilede-te-xing.md)
* [volatile写-读建立的happens-before关系](/di-san-zhang-java-nei-cun-mo-xing/volatilenei-cun-yu-yi/volatile5199-du-jian-li-de-happens-before-guan-xi.md)
* [volatile写-读的内存语义](/di-san-zhang-java-nei-cun-mo-xing/volatilenei-cun-yu-yi/volatile5199-du-de-nei-cun-yu-yi.md)
* [volatile内存语义的实现](/di-san-zhang-java-nei-cun-mo-xing/volatilenei-cun-yu-yi/volatilenei-cun-yu-yi-de-shi-xian.md)
* [JSR-133为什么要增强volatile的内存语义](/di-san-zhang-java-nei-cun-mo-xing/volatilenei-cun-yu-yi/jsr-133wei-shi-yao-yao-zeng-qiang-volatile-de-nei-cun-yu-yi.md)

当声明共享变量为volatile后，对这个变量的读/写将会很特别。为了揭开volatile的神秘面纱，下面将介绍volatile的内存语义及volatile内存语义的实现。

