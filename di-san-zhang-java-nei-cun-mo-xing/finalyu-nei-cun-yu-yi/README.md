# final域内存语义

## 目录

* [final域的重排序规则](finalyu-de-zhong-pai-xu-gui-ze.md)
* [写final域的重排序规则](xie-final-yu-de-zhong-pai-xu-gui-ze.md)
* [读final域的重排序规则](du-final-yu-de-zhong-pai-xu-gui-ze.md)
* [final域为引用类型](finalyu-wei-yin-yong-lei-xing.md)
* [为什么final引用不能从构造函数内“溢出”](wei-shi-yao-final-yin-yong-bu-neng-cong-gou-zao-han-shu-nei-201c-yi-chu-201d.md)
* [final语义在处理器中的实现](finalyu-yi-zai-chu-li-qi-zhong-de-shi-xian.md)
* [JSR-133为什么要增强final的语义](jsr-133wei-shi-yao-yao-zeng-qiang-final-de-yu-yi.md)

与前面介绍的锁和volatile相比，对final域的读和写更像是普通的变量访问。下面将介绍final域的内存语义。

