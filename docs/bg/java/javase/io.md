# 输入与输出

## 输入/输出流

* 输入流InputStream：可以从文件、网络连接、内存块中读入一个字节序列的对象
* 输出流OutputStream：可以向文件、网络连接、内存块中写入一个字节序列的对象

输入与输出流共同构成了IO类层次结构的基础

因为面向字节的流不便于处理以Unicode形式存储的信息，所以从抽象类Reader和Writer中继承出来了一个专门用于处理Unicode字符的单独的类层次结构，这些类的操作都是基于两字节的Char值（Unicode码元），而不是byte。

## 流家族

* 处理字节、字符串，已二进制文件、zip压缩包等形式读写文件的输入输出流

  ![epub_34339937_163(1)](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/epub_34339937_163(1).jpg)

* 以Unicode码元为单位处理的Reader、Writer

  ![epub_34339937_165(1)](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/epub_34339937_165(1).jpg)

* 实现类如下，其中Closeable实现了接口方法可以抛出IOException，Closeable接口扩展了AutoCloseable，AutoCloseable可以抛出所有异常

  ![epub_34339937_169](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/epub_34339937_169.jpg)

  
