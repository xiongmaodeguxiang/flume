#### 一、flume原理框架

​	每个Flume Agent包含三个主要组件：source、channel、sink。source是从一些其他生产数据的应用中接收数据的活跃组件，每个source可以监听一个或者多个网络端口，用于接收数据或者从本地文件系统读取数据。每个source必须至少连接一个channel；

​	Channel是被动主键用来缓冲Agent已经接收，单尚未写出到另一个Agent或者存储系统的数据，Source向Channel中写入数据，Sink从Channel中读取数据。多个Source可以安全地写入相同的Channel，多个Sink可以从相同的Channel中读取数据，**它可以保证只有一个Sink将会从Channel中读取一个指定的特定的事件（即：一个事件只会被读取一次）**。

​	Sink连续轮训各自的Channel来读取和删除事件。Sink只有将事件推送到下一个阶段，或者最终的目的地，一旦在下一个阶段或目的地数据是安全的，Sink通过事务提交通知Channel删除该事件，因此**数据在Agent中是安全的不会丢失，由于单个Agent的数据安全性也决定了多个Agent链处理数据时也是安全的**。

![](C:\Users\dell\Desktop\学习资料\flume\flume原理图.png)

​	Flume Agent可以配置多个Source、Channel、Sink，Source接收到的事件可以通过Channel处理器（processor）、channel拦截器、Channel选择器将写入特定的channel中，Channel 处理器对应的类是**ChannelProcessor**类作用是处理从Source接收的Event,通过调用processEventBatch方法选择**批量发送**或者**单个发送**；**Channel拦截器**可以对事件进行修改或者删除，比如可以向报头中添加信息，用于决定事件写入的Channel，**多个Channel拦截器可以组成拦截链**，最后返回一个**处理后的事件列表**；Channel选择器是决定事件最终写入哪个Channel的组件。

​	在flume集群中，有许多种组织Agent的方法，最简单的一种就是使用单个Agent，来接收应用程序服务器的数据，且用同样的Agent将数据写入到存储系统，这样的系统可以在存储点的系统中隔离