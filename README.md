# RTOS1-summary

嵌入式实时操作系统1——初识嵌入式实时操作系统

**嵌入式实时操作系统是什么**
**嵌入式实时操作系统是一个特殊的程序，是一个支持多任务的运行环境**。嵌入式实时操作系统最大的特点就是“实时性”，如果有一个任务需要执行，实时操作系统会立即执行该任务，不会有较长的延时。典型的实时操作系统有uCOS ，RT-Thread，FreeRTOS ，VxWorks，WinCE等。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b2984f4611b64348b24e0074e4a78635.png)
嵌入式实时操作系统是一个特殊的程序(通常称为内核)，它可以创建，销毁，控制所有任务。嵌入式实时操作系统除了包含一个内核以外，还提供其他服务，如文件系统，协议栈，图形用户界面等。本文的重点在于了解嵌入式实时操作系统内核的工作原理和结构，因此文中提到的实时操作系统通常指的是操作系统内核。**实时操作系统内核通常要占用5%左右的CPU运行时间，另外内核是一个软件代码，需要额外占用ROM空间和RAM空间**。

**实时性**
实时性可以定义为：**触发条件产生后系统的反应能力**。通俗的描述就是“**天下武功唯快不破**”，能达到所需要的“快”就是实时,在不同的场合需要达到us级、ns级。实时系统不仅仅只表现在“快”上，更主要的是实时系统需要对触发事件在限定时间内做出反应，这个限定时间是根据实际需要来定，例如自动驾驶控制系统的规定时间要求很短，需要在极短时间内做出动作；一些农业温度控制系统的规定时间要求比较长，需要温度控制平滑稳定。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4b1854e971d6476b99e365b853fe4937.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)
**响应时间**
**实时性越强，其响应时间越短**。响应时间是指系统识别到一个事件到开始做出响应的时间。举一个简单的例子：一个工控设备有一个急停按键开关，用户希望按下急停开关的时候系统立即将停止所有的动作，假设用户在第1.001秒时按下了急停开关，软件系统在第1.011秒时执行了停止指令，工控设备相应的机械部件在1.211秒停止动作，此时软件系统响应时间为0.01秒，设备系统响应时间为0.21秒，设备系统的响应时间和软件系统的响应时间有一定区别，通常情况下设备系统的响应时间>软件系统的响应时间。本文中提到的响应时间指的是**软件系统的响应时间**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3060068335584a509dd6262ea9079168.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)
再举一个通俗的例子：你在玩王者荣耀，突然发现对面打野从草丛中跑出准备gank你，从你眼睛看到，到手指点击闪现，然后到你的人物闪现到塔下。这就是响应时间，高端职业玩家可能只需要100ms即可完成整套动作，而菜鸟玩家可能需要1000ms来完成整个动作。

**普通的嵌入式软件架构**
普通的嵌入式软件系统通常设计成**前后台结构**，这个结构包含一个死循环和若干中断服务程序：应用程序是一个无限循环的代码块，循环中调用相应的函数完成相应的操作（后 台），中断程序用于处理系统的异步事件（前台）。前台称做**中断级**，后台称做**任务级**。下面是一个典型的前后台结构的代码：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e01db6f9318141bca02fa92a9563cf4d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_20,color_FFFFFF,t_70,g_se,x_16)
上图中的代码的执行流程是：
1、判断按键标志位，若标志位为1就执行按键处理操作。
2、判断通讯标志位，若标志位为1就执行通讯处理操作。
3、执行LCD显示操作
4、判断传感器标志位，若标志位为1就执行传感器处理操作。
此代码中有3个中断函数：
1、GPIO外部中断，当按键按下后产生一个中断，中断函数中将键标志位置1。
2、串口空闲中断，当串口总线空闲时产生一个中断，中断函数中将通讯标志位置1。
3、定时器中断，每500ms周期性产生一个中断，中断函数中将传感器标志位置1。

代码执行如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/cdbb268c34644448bee77c59f0c1424e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_15,color_FFFFFF,t_70,g_se,x_16)
分析运行图：
1、程序判断按键标志位，标志位为0，不执行按键处理函数。
2、程序判断通讯标志位，标志位为0，不执行通讯处理函数。
3、程序执行显示处理函数，此时用户按下了按键，系统进入按键中断程序将按键标志位置1，中断完成后返回显示处理函数继续运行。
4、显示处理函数运行过程中，此时串口接收完一包数据产生了一个空闲中断，系统进入串口空闲中断程序将通讯标志位置1，中断完成后返回显示处理函数继续运行。
5、显示处理函数运行过程中，定时器产生中断，系统进入定时器中断程序将传感器标志位置1。
6、显示处理函数执行完毕，程序判断传感器标志位，标志位为1，执行通讯处理函数。
7、程序判断按键标志位，标志位为1，执行按键处理函数。
8、程序判断通讯标志位，标志位为1，执行通讯处理函数。
9、无限循环...

由上述例子可知，按键标志位和通讯标志位就绪后，程序还需要等待显示函数，传感器函数执行完毕。即使是按键处理函数的紧急性再高，也需要等待其他函数执行完毕。因此就产生了响应延迟，响应延迟的时间随机的不确定的，有的时候时几毫秒的时间，有的时候是几百毫秒甚至更长（如执行传感器读取），因此需要提高系统的实时性。

**实时操作系统**
实时操作系可以**随时剥夺正在运行任务的CPU使用权**，并将CPU使用权交给进入就绪状态的最高优先级任务，使用操作系统后的运行图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/bb327464e74549c4844acc9e20079a66.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_15,color_FFFFFF,t_70,g_se,x_16)
分析运行图：
1、低优先级的显示任务正在运行，此时用户按下了按键，系统进入按键中断程序给按键任务发送一个信号，此时按键任务进入就绪状态，中断返回时切换到按键处理任务中运行。
2、按键处理任务正在运行，此时串口接收完一包数据产生了一个空闲中断，系统进入串口空闲中断程序并给通讯处理任务发送一个信号，通讯处理任务进入就绪状态，中断返回时切换到通讯处理任务。
3、通讯处理任务执行完毕，放弃CPU使用权限，此时切换到按键处理任务。
4、按键处理任务执行完毕，放弃CPU使用权限，此时切换到显示处理任务。

由此可见，**当触发产生后实时操作系将立即中断当前的任务并执行相应的任务**。使用实时操作系可以极大的提高软件系统的实时性。

**实时操作系统组成**
实时操作系由以下3个子系统组成（以uCOS和FreeRTOS为参考对象）：
1、任务调度子系统
2、任务通信子系统
3、内存管理子系统
![在这里插入图片描述](https://img-blog.csdnimg.cn/88a01dc26987487cba01f69ef7b4138f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbGl5aW51bzIwMTc=,size_9,color_FFFFFF,t_70,g_se,x_16)
**任务调度子系统**主要是维护两个链表：**就绪表和等待表**。切换任务时从就绪表中取出最高优先级任务；任务需要延时等待时，内核将任务中就绪表中移动到等待表中；时钟节拍任务会周期性的更新等待表，并将等待时间完成的任务从等待表中移动到就绪表中。

**任务通讯子系统**主要是维护一个链表：**挂起表**。任务需要等待信号时，内核将任务移动到挂起表中，当内核收到信号时，内核将任务从挂起表中移动到就绪表中。

**内存管理子系统**，内核提供了几种动态申请内存的方式，防止出现内存碎片。




> 未完待续…
> 实时操作系统系列将持续更新
> 创作不易希望朋友们点赞，转发，评论，关注。
> 您的点赞，转发，评论，关注将是我持续更新的动力
> 作者：李巍
> Github：liyinuoman2017
