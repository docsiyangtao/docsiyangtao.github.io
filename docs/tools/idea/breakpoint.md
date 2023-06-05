说说IDEA中的debug，不再只会使用普通行断点，效率Max~                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 

## 按钮

### 横排每个按钮的作用

![image-20220327014446399](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220327014446399.png)

1. Show Execution Point：回到当前断点所在的页面（ALT + F10）
2. Step Over：执行一行代码，遇到方法不进入（F8）
3. Step Into：执行一行代码，遇到自定义方法会进入（F7）
4. Force Step Into：执行一行代码，遇到所有的方法都会进入（ALT + SHIFT + F7）
5. Step Out：执行完当前方法，回到调用该方法的下一行（SHIFT + F8）
6. Drop Frame：摧毁栈帧，回到调用该方法处
7. Run To Cursor：回到光标处，不执行光标所在行（ALT + F9）
8. Evaluate Expression：计算器，可用于计算当前方法中某个方法的结果



### 竖排每个按钮的作用

![image-20220327014513493](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220327014513493.png)

1. Rerun：重新debug程序（CTRL + F5）
2. Resume Program：执行程序，下一个断点执行前停住（F9）
3. Pause Program：暂停此次debug（）
4. Stop：终止当前程序（CTRL + F2）
5. View BreakPoints：查看所有的断点（CTRL + SHIFT + F8）
6. Mute BreakPoints：让所有断点失效（）
7. Get Thread Dump：查看线程dump文件（）



## 断点的种类

### 普通断点

普通程序语句上，没什么好说的



### 方法断点

1. 方法执行前和结束后各触发一次断点
2. 方法为接口方法是，则可以直接跳转到对应的实现类触发断点

<video src="https://vid-blog-note-1304850123.cos.ap-guangzhou.myqcloud.com/2021-12-29-23-47-30.mp4"></video>



### 异常断点

1. 声明指定类型的异常，当程序遇到该异常以后自触发断点
2. 即使对异常进行了捕获，断点仍会触发

<video src="https://vid-blog-note-1304850123.cos.ap-guangzhou.myqcloud.com/2021-12-29-23-56-09.mp4"></video>



### 监控断点

1. 断点打在某一属性上
2. 每当这个属性发生改变的时候，触发断点

<video src="https://vid-blog-note-1304850123.cos.ap-guangzhou.myqcloud.com/2021-12-30-00-14-53.mp4"></video>



### 线程断点

1. 右键设置断点的Suspend为Thread
2. 可强制切换线程来控制线程的执行顺序及查看参数

<video src="https://vid-blog-note-1304850123.cos.ap-guangzhou.myqcloud.com/2021-12-30-00-31-45.mp4"></video>



## 特殊场景

### 循环

1. 在循环中的断点可通过表达式，在符合变量条件时触发断点

<video src="https://vid-blog-note-1304850123.cos.ap-guangzhou.myqcloud.com/2021-12-30-00-46-52.mp4"></video>



### 跳过某一步骤

1. 当一个方法中有多个操作而我们不想执行某一行时可以使用force return强行退出该方法
2. 区分：Step Out和Drop Frame要想执行下一步方法就必须执行前一个方法的全部内容，不能强制退出

<video src="https://vid-blog-note-1304850123.cos.ap-guangzhou.myqcloud.com/2021-12-30-01-01-19.mp4"></video>



### 流式编程

1. 使用Trace Current Stream Chain查看流程的每个步骤、每个数据都怎么变化

<video src="https://vid-blog-note-1304850123.cos.ap-guangzhou.myqcloud.com/2021-12-30-01-14-10.mp4"></video>

