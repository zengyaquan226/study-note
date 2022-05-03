# RabbitMQ

## 什么是MQ

​		MQ (message queue)，从字面意思上看，本质是个队列，FIFO 先入先出，只不过队列中存放的内容是 message 而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ 是一种非常常见的上下游 “逻辑解耦 + 物理解耦” 的消息通信服务。使用了 MQ 之后，消息发送上游只需要依赖 MQ，不 用依赖其他服务。



## 为什么要使用MP



### 流量消峰

​		举个例子，如果订单系统最多能处理一万次订单，这个处理能力应付正常时段的下单时绰绰有余，正常时段我们下单一秒后就能返回结果。但是在高峰期，如果有两万次下单操作系统是处理不了的，只能限制订单超过一万后不允许用户下单。使用消息队列做缓冲，我们可以取消这个限制，把一秒内下的订单分散成一段时间来处理，这时有些用户可能在下单十几秒后才能收到下单成功的操作，但是比不能下单的体验要好

优点：不至于要服务器直接宕机

缺点：排队会让效率降低



### 应用解耦

​		以电商应用为例，应用中有订单系统、库存系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。在这几分钟的时间里，物流系统要处理的内存被缓存在消息队列中，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可 ，中单用户感受不到物流系统的故障，提升系统的可用性。

![img](https://img-blog.csdnimg.cn/eadc9eef9b9140339207d5cb2456ed19.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5piv6Zi_5bKa5ZGQ,size_20,color_FFFFFF,t_70,g_se,x_16)



### 异步处理

​		有些服务间调用是异步的，例如A调用B，B需要花费很长时间执行，但是A需要知道B什么时候可以执行完，以前一般有两种方式，A过一段时间去调用B的查询api查询。或者A提供一个callback api,B执行完之后调用api通知A服务。这两种方式都不是很优雅，使用消息总线，可以很方便解决这个问题,A调用B服务后，只需要监听B处理完成的消息，当B处理完成后，会发送一条消息给MQ，MQ会将此消息转发给A服务。这样A服务既不用循环调用B的查询api，也不用提供callback api。同样B服务也不用做这些操作。A服务还能及时的得到异步处理成功的消息。

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-rvQc6uJJ-1630999921178)(D:\学习资料\图片\image-20210827104345364.png)]](https://img-blog.csdnimg.cn/647443dd56554010b87a3fc5487ef0f6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5piv6Zi_5bKa5ZGQ,size_20,color_FFFFFF,t_70,g_se,x_16)





## RabbitMPS



![image-20220426151806627](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220426151806627.png)	

### 四大特征

#### 生产者	

产生数据发送消息的程序是生产者

#### 交换机	

​		交换机是RabbitMQ非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推关到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

#### 队列	

​		队列是RabbitMQ内部使用的一种数据结构，尽管消息流经RabbitMQ和应用程序但它们只能存储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式

#### 消费者

​		消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。



## 工作原理图

![image-20220426153054160](D:\学习笔记\图片\image-20220426153054160.png)

### 名词解析

#### Borker	

接受和分发消息的应用，RabbitMQ Server就是Message Broker

#### Virtual host	

出于多租户和安全因素设计的，把AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的namespace.概念。当多个不同的用户使用同一个RabbitMQ server提供的服务时，可以划分出多个vhost，每个用户在自己的vhost创建exchange / queue 等



#### Connection	

publisher / consumer和broker之间的TCP连接



#### Channel	

如果每一次访问 RabbitMQ 都建立一个Connection，在消息量大的时候建立TCPConnection的开销将是巨大的，效率也较低。Channel是在connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个thread 创建单独的channel进行通讯，AMQP method包含了channel id 帮助客户端和message broker识别 channel，所以channel之间是完全隔离的。Channel作为轻量级的Connection极大减少了操作系统建立TCP connection的开销



#### Exchange	

message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到queue 中去。常用的类型有: direct (point-to-point), topic (publish-subscribe) and fanout(multicast)

#### Queue	

消息最终被送到这里等到consumer取走



#### Binding	

exchange和queue之间的虚拟连接，binding中可以包含routing key，Binding消息被保存到exchange中的查询表中，用于message的分发依据





## 安装RabbitMQ

### 安装Erlang：

<https://github.com/rabbitmq/erlang-rpm/releases/download/v23.2.6/erlang-23.2.6-1.el7.x86_64.rpm>

在linux上安装：

```
rpm -ivh erlang-23.2.6-1.el7.x86_64.rpm

# 测试
erl -version
```

### 安装socat

```
yum install -y socat
```



### 安装RabbitMQ

<https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.12/rabbitmq-server-3.8.12-1.el7.noarch.rpm>

```
rpm -ivh rabbitmq-server-3.8.12-1.el7.noarch.rpm
```



### 常用命令

```
# 添加开机启动RabbitMQ服务
chkconfig rabbitmq-server on

# 启动服务
/sbin/service rabbitmq-server start

# 查看服务状态
/sbin/service rabbitmq-server status

#停止服务
/sbin/service rabbitmq-server stop

#安装可视化管理 插件
rabbitmq-plugins enable rabbitmq_management

```



### 安装安装可视化插件管理

 执行指令

```
#安装可视化管理 插件
rabbitmq-plugins enable rabbitmq_management
```

关闭防火墙或者开启15672端口

```
# 开启防火墙 指定的15672端口
firewall-cmd --permanent --add-port=15672/tcp

# 重启生效
firewall-cmd --reload

```

学习是可以直接关闭防火墙

```
# 关闭防火墙
systemctl stop firewalld
# 下次自启不在开启
systemctl enable firewalld
```



**在浏览器上输入ip:15673进行登陆 ** 账号和密码默认都是 guest

![image-20220426201247147](D:\学习笔记\图片\image-20220426201247147.png)



**如果出现这个错误说明没有注册用户，注册一个用户即可**

```
# 创建账号 第一个账号 第二个密码
rabbitmqctl add_user admin 123

# 设置用户角色
rabbitmqctl set_user_tags admin adminstrator

# 设置用户权限 给 / 这个 vHost 后面的权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"

# 查看用户列表
rabbitmqctl list_users

```



## 简单模式（hello world）

### 配置java开发环境

对pom.xml中 导入以下的依赖

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.8.0</version>
        </dependency>

        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
    </dependencies>

```



### 生产者的代码实现

```java

/**
 * @author zyq
 * @version 1.0
 * 生产者
 */
public class Producer {
    // 创建 队列 队列的名为hello
    public static final String QUEUE = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置工厂的ip
        factory.setHost("192.168.1.22");
        // 设置用户名
        factory.setUsername("root");
        // 设置密码
        factory.setPassword("root");
        // 创建连接
        Connection connection = factory.newConnection();
        // 获取连接中的信道
        Channel channel = connection.createChannel();
        // String var1, boolean var2, boolean var3, boolean var4, Map<String, Object> var5
        /*
         * 生成队列
         * queueDeclare方法的参数：
         * 第一个参数：队列名称
         * 第二个参数：队列里的消息是否持久化（是否存入磁盘） true表示存入 默认false 存在内存
         * 第三个参数：该队列是否只能一个消费者消费 true 表示一个独占 false 表示只能多个可以一起消费
         * 第四个参数：是否自动删除 在最后一个消费者开连接之后 队列是否自动删除 true表示自动删除 false 表示不自动删除
         * 第五个参数：其他参数
         */
        channel.queueDeclare(QUEUE, false, false, false, null);
        // 需要发送的消息
        String message = "hello world";
        // 通过信道发送消息到队列中
        /*
         * basicPublish的参数
         * 第一个参数：发送到那一个交换机
         * 第二个参数：路由的key是那个 这次是队列的名称
         * 第三个参数：其他消息
         * 第四个参数：需要发送的消息
         */
        channel.basicPublish("", QUEUE, null, message.getBytes());
        System.out.println("消息发送到队列了");


    }
}

```



**成功后的web情况**

![image-20220426212757408](D:\学习笔记\图片\image-20220426212757408.png)

### 消费者的代码实现

```java

/**
 * @author zyq
 * @version 1.0
 * 消费者
 */
public class Consumer {
    // 队列的名称
    public static final String QUEUE = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 获取连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.1.22");
        factory.setUsername("root");
        factory.setPassword("root");
        // 获取连接
        Connection connection = factory.newConnection();
        // 获取信道
        Channel channel = connection.createChannel();

        /*
         * basicConsume的参数
         * 第一个参数：消费的队列
         * 第二个参数（autoAck）：消费成功后是否要自动应答 true 自动应答 false 手动应答
         * 第三个参数（deliverCallback）：当传送回来的消息进行的回调 即就是正常的获取到了消息
         * 第四个参数（CancelCallback）：当收费者取消了对消息的接受 回调的方法
         */
        DeliverCallback deliverCallback
                = (consumerTag, message) -> System.out.println(new String(message.getBody()));
        CancelCallback cancelCallback
                = consumerTag -> System.out.println("消费者取消了接收参数");
        channel.basicConsume(QUEUE, true, deliverCallback, cancelCallback);
    }
}

```



***成功打印情况：***

![image-20220426221208639](D:\学习笔记\图片\image-20220426221208639.png)

![image-20220426221229238](D:\学习笔记\图片\image-20220426221229238.png)



## 工作队列（work queues）

​		**简单来说就是当队列中的消息非常多的时候，并且需要保证一条消息只能被消费一次，有多个工作线程（消费者）进行消费的时候，rabbitmq采用轮询的方式进行分发信息**。

![image-20220427082338592](D:\学习笔记\图片\image-20220427082338592.png)



### 提取工具类

```java
// 工具类
public class RabbitMQUtils {
    public static Channel getChannel() {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.1.22");
        factory.setUsername("root");
        factory.setPassword("root");
        try {
            Connection connection = factory.newConnection();
            return connection.createChannel();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```



### 消费者(工作线程)代码

```java
/**
 * @author zyq
 * @version 1.0
 * 工作线程（消费者）
 */
// 代码实现之前需要保证有这个队列（hello）
public class Work01 {
    // 队列名
    public static final String QUEUE = "hello";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();

        // 成功接收消息的回调函数
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("接收到的消息:" + new String(message.getBody()));

        // 当消息被取消的时候回调函数
        CancelCallback cancelCallback = consumerTag -> System.out.println(consumerTag + "消息被取消");
        channel.basicConsume(QUEUE, true, deliverCallback, cancelCallback);

    }
}

```



### 模拟多个线程进行工作

**使用idea这个工具启动多个线程，默认不可以启动多个，做以下的配置**

![image-20220427091157650](D:\学习笔记\图片\image-20220427091157650.png)

****

***将Allow parallel run 勾上即可***



### 生产者代码实现

```java
/**
 * @author zyq
 * @version 1.0
 * 生产者（发送大量的消息到队列）
 */
public class Task01 {
    public static final String QUEUE = "hello";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 创建hello队列
        channel.queueDeclare(QUEUE, false, false, false, null);
        // 向队列发送消息
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String next = scanner.next();
            channel.basicPublish("", QUEUE, null, next.getBytes());
            System.out.println("发送消息" + next + "完毕");
        }
    }
}

```



**最终效果**

<font color=red>就是启动的线程采用轮询的方式，一个线程一个线程的方式，依次获取队列的信息</font>



## 消息应答

1. 消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生
2. RabbitMQ一旦向消费者传递了一条消息，便立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费这的消息，国为它无法接收到。
3. 为了保证消息在发送过程中不丢失，RabbitMQ引入消息应答机制，消息应答就是:消费者在接收到消息并且处理该消息之后，告诉RabbitMQ它已经处理了，RabbitMQ可以把该消息删除了。



### 自动应答和手动应答

**自动应答是接收到消息就应答，可能会造成消息的丢失，尽量不要使用**



**手动应答的的信息应答的方式**

1. Channel.basicAck(用于肯定确认)RabbitMQ已知道该消息并且成功处理，可以将其丢弃
2. Channel.basicNack(用于否定确认)
3. Channel.basicReject(用于否定确认)，与Channel.basicNack相比少了一个参数(Multiple)，不处理该消息了，直接拒绝，可以将其丢弃了。



### Multiple的解释

1. true表示批量应答channel上未应答的消息，比如channel上有传送tag的消息5,6,7,8,，当前tag是8，那么此时5-8的这些还未应答的消息就会被确认收到消息应答
2. false同上面相比只会应答tag=8的消息，5,6,7这三个消息依然不会被确认收到消息应答
3. 虽然可以减少网络的拥堵，但是可以造成消息的丢失，不推荐使用

<img src="D:\学习笔记\图片\image-20220427202330980.png" alt="image-20220427202330980" style="zoom: 50%;" />



### 消息自动重新入队

​		当消费者断开了连接，并且消息也没有进行应答，那这个消息是不可以进行删除的，mq会将其重新入队列，并且如果有消费者可以处理，会交给其处理。



### 消息手动应答

```java
// 生产者
public class Task02 {
    public static final String ASK_QUEUE = "ask_queue";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMQUtils.getChannel();
        channel.queueDeclare(ASK_QUEUE, false, false, false, null);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            channel.basicPublish("", ASK_QUEUE, null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println("发送消息:" + message);
        }
    }
}

```

```java
// 消费者
public class Worker02 {
    // 队列名 从这个队列中获取消息
    public static final String ASK_QUEUE = "ask_queue";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMQUtils.getChannel();
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            try {
                Thread.sleep(1000);
                // 模拟第二个线程使用
                // Thread.sleep(30000);
                System.out.println("接收到的消息:" + new String(message.getBody(), StandardCharsets.UTF_8));
                System.out.println(consumerTag);
                // 进行手动应答
                // multiple 是否进行批量应答
                channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        System.out.println("C1进行应答");
        // 模拟第二个线程使用
        // System.out.println("C2进行应答");
        // false表示手动应答
        channel.basicConsume(ASK_QUEUE, false, deliverCallback, System.out::println);
    }
}

```

****

***预期结果***

​		队列中的消息采用轮询的方式，一个线程一个线程的发消息，因为采用的是手动应答的方式，当执行到的线程的执行时间长的情况，将其进行关闭，这个消息不会被删除，而是会重新入队列，给可以处理的消费者进行处理。



## RabbitMQ持久化

### 队列持久化

```java
// 生产者的代码
// 创建队列 将第二个参数改成true即可
        channel.queueDeclare(ASK_QUEUE, true, false, false, null);
```

![image-20220427215215009](D:\学习笔记\图片\image-20220427215215009.png)

![image-20220427214915520](D:\学习笔记\图片\image-20220427214915520.png)



### 消息持久化

```java
// 在生产者发送消息的时候
// 进行消息持久化 props 的位置处 MessageProperties.PERSISTENT_TEXT_PLAIN 表示持久化
channel.basicPublish("", ASK_QUEUE, MessageProperties.PERSISTENT_TEXT_PLAIN, 	                                                   message.getBytes(StandardCharsets.UTF_8));
          
```

<font color=red size=4>注：但是这种情况不可以完全的保证信息不丢失</font>



## 不公平分发

**在消费者的一方在获取信息之前进行设置<font color=red>channel.bsaicQos(1)</font>即可**

```java
// 设置为不公平分发
channel.basicQos(1);
// 接收消息
channel.basicConsume(QUEUE, false, deliverCallback, cancelCallback);

```



## 预取值

**在消费者的一方在获取信息之前进行设置<font color=red>channel.bsaicQos(prefetch)</font>即可**

prefetch:代表预取值的数量，即指定的信道上可以存放的未确定的消息的最大数量，一旦进入这个信道就会交给指定的消费者进行消费



```java
// 设置预取值 5 意思是信道可以存放未确定消息的最大数量
channel.basicQos(5);
```



## 发布确认



### 开启发布确认

**在生产者代码中，执行channel.confirmSelect()即开启发布确认，默认是不开启的**

```java
Channel channel = RabbitMQUtils.getChannel();
 // 开启发布的确认
channel.confirmSelect();
```



### 确认发布的方式

#### 方式一 ：单一确认发布

**单一确认是同步的发布，必须确认后才可以再次发布**

实现方式：生产者发布完信息后，马上确认使用<font color=blue>channel.waitForConfirms</font>进行发布确认

```java
//单个确认发布
    public static void messageConfirmSingle(Channel channel, String queueName) throws IOException, InterruptedException {
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            // 发布消息
            channel.basicPublish("", queueName, null, (i + "").getBytes());
            // 进行单个确认发布
            boolean flag = channel.waitForConfirms();
            if (flag) {
                System.out.println("消息发送成功");
            }
        }
    }
```



#### 方式二：批量确认发布

**批量确认发布也是同步，就是自己确定在第几个进行确认即可，还是使用<font color=blue>channel.waitForConfirms</font>进行确认**

```java
 //批量确认发布
    public static void messageConfirmBatch(Channel channel, String queueName) throws IOException, InterruptedException {
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            // 发布消息
            channel.basicPublish("", queueName, null, (i + "").getBytes());
            System.out.println("消息发送成功");
            // 进行批量发布
            if (i % 100 == 0) {
                channel.waitForConfirms();
            }
        }
        channel.waitForConfirms();
    }

```



#### 方式三：异步确认发布

**异步确认发布时异步进行的，并且可以相对于批量确认，可以准确的确定出那些事没有确认成功的**

***实现方式，在进行发送消息之前，准备消息的监听器即可***

```java
 //批量确认发布
    public static void messageConfirmAsyncBatch(Channel channel, String queueName) throws IOException, InterruptedException {
        // (ConfirmCallback ackCallback, ConfirmCallback nackCallback)
        // (long deliveryTag, boolean multiple)
        // 信息确认发布成功的回调方法
        ConfirmCallback ackCallback = (deliveryTag, multiple) -> {
            System.out.println("信息确认发布成功" + deliveryTag);
        };
        // 信息确认发布失败的回调方法
        ConfirmCallback nackCallback = (deliveryTag, multiple) -> {
            System.out.println("信息确认发布失败" + deliveryTag);
        };
        // 准备信息的监视器
        channel.addConfirmListener(ackCallback, nackCallback);
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            // 发布消息
            channel.basicPublish("", queueName, null, (i + "").getBytes());
            System.out.println("消息发送成功");
        }
    }
```

![image-20220428161335485](D:\学习笔记\图片\image-20220428161335485.png)



### 处理异步未确认的消息

解决方案：

1. 将发送的消息存放到一个容器中（ConcurrentSkipListMap）
2. 将成功确认的消息从集合中删除即可
3. 最后处理集合中的消息即可



#### 代码实现

```java
//异步确认发布
    public static void messageConfirmAsyncBatch(Channel channel, String queueName) throws IOException, InterruptedException {
        // 创建一个容器存放未成功的确认的消息
        ConcurrentSkipListMap<Long, String> map = new ConcurrentSkipListMap<>();
        // 准备信息的监视器
        // (ConfirmCallback ackCallback, ConfirmCallback nackCallback)
        // (long deliveryTag, boolean multiple)
        // 信息确认发布成功的回调方法
        ConfirmCallback ackCallback = (deliveryTag, multiple) -> {
            // 将成功确认的从容器进行删除
            if (multiple) {
                // 当进行批量删除时 true表示全部都删除
                ConcurrentNavigableMap<Long, String> longStringConcurrentNavigableMap
                        = map.headMap(deliveryTag, true);
                longStringConcurrentNavigableMap.clear();
                // System.out.println(map);
            } else {
                // 未批量删除时
                map.remove(deliveryTag);
            }
            if (map.size() == 0) {
                System.out.println("所有信息发送成功");
            }
            System.out.println("信息确认发布成功" + deliveryTag);
        };
        // 信息确认发布失败的回调方法
        ConfirmCallback nackCallback = (deliveryTag, multiple) -> {
            // 处理确认失败的消息
            System.out.println("信息确认发布失败map集合的情况 = " + map);
            System.out.println("信息确认发布失败" + deliveryTag);
        };
        channel.addConfirmListener(ackCallback, nackCallback);
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            // 消息
            String message = i + "消息";
            // 发布消息
            channel.basicPublish("", queueName, null, message.getBytes());
            // 将所有发送的消息存入到map容器中
            // k 存放进行消息的编号
            // 因为从下一个开始所有要减1
            map.put(channel.getNextPublishSeqNo() - 1, message);
            System.out.println("消息发送成功");
        }
    }
```



### 完整代码

```java
/**
 * @author zyq
 * @version 1.0
 * 进行发布消息的确认
 * 1 单个确认发布  1000条信息耗时：1194ms
 * 2 批量确认发布  1000条信息耗时：194ms
 * 3 异步批量确认发布  1000条信息耗时：98ms
 */
public class MessagePublishConfirm {
    // 总计发送消息1000条
    public static final Integer MESSAGE_COUNT = 1000;

    public static void main(String[] args) throws IOException, InterruptedException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 开启发布确认
        channel.confirmSelect();
        // 确定队列名
        String queueName = UUID.randomUUID().toString();
        // 创建队列
        channel.queueDeclare(queueName, true, false, false, null);
        // 记录开始时间
        long start = System.currentTimeMillis();
        // 调用相应的方法
        // messageConfirmSingle(channel, queueName);
        // messageConfirmBatch(channel, queueName);
        messageConfirmAsyncBatch(channel, queueName);
        // 记录结束的时间
        long end = System.currentTimeMillis();
        System.out.println("1000条信息耗时：" + (end - start) + "ms");

    }

    //单个确认发布
    public static void messageConfirmSingle(Channel channel, String queueName) throws IOException, InterruptedException {
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            // 发布消息
            channel.basicPublish("", queueName, null, (i + "").getBytes());
            // 进行单个确认发布
            boolean flag = channel.waitForConfirms();
            if (flag) {
                System.out.println("消息发送成功");
            }
        }
    }

    //批量确认发布
    public static void messageConfirmBatch(Channel channel, String queueName) throws IOException, InterruptedException {
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            // 发布消息
            channel.basicPublish("", queueName, null, (i + "").getBytes());
            System.out.println("消息发送成功");
            // 进行批量发布
            if (i % 100 == 0) {
                channel.waitForConfirms();
            }
        }
        channel.waitForConfirms();
    }

    //异步确认发布
    public static void messageConfirmAsyncBatch(Channel channel, String queueName) throws IOException, InterruptedException {
        // 创建一个容器存放未成功的确认的消息
        ConcurrentSkipListMap<Long, String> map = new ConcurrentSkipListMap<>();
        // 准备信息的监视器
        // (ConfirmCallback ackCallback, ConfirmCallback nackCallback)
        // (long deliveryTag, boolean multiple)
        // 信息确认发布成功的回调方法
        ConfirmCallback ackCallback = (deliveryTag, multiple) -> {
            // 将成功确认的从容器进行删除
            if (multiple) {
                // 当进行批量删除时 true不可以少
                ConcurrentNavigableMap<Long, String> longStringConcurrentNavigableMap
                        = map.headMap(deliveryTag, true);
                longStringConcurrentNavigableMap.clear();
                System.out.println(map);
            } else {
                // 未批量删除时
                map.remove(deliveryTag);
            }
            if (map.size() == 0) {
                System.out.println("所有信息发送成功");
            }
            System.out.println("信息确认发布成功" + deliveryTag);
        };
        // 信息确认发布失败的回调方法
        ConfirmCallback nackCallback = (deliveryTag, multiple) -> {
            // 处理确认失败的消息
            System.out.println("信息确认发布失败map集合的情况 = " + map);
            System.out.println("信息确认发布失败" + deliveryTag);
        };
        channel.addConfirmListener(ackCallback, nackCallback);
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            // 消息
            String message = i + "消息";
            // 发布消息
            channel.basicPublish("", queueName, null, message.getBytes());
            // 将所有发送的消息存入到map容器中
            // k 存放进行消息的编号
            // 因为从下一个开始所有要减1
            map.put(channel.getNextPublishSeqNo() - 1, message);
            System.out.println("消息发送成功");
        }
    }
}

```



**推荐使用异步确认发布**



## 交换机

**注：生产者生产的消息从不会直接发送到队列，前面是使用了默认的交换机，生产者只可以将消息发送给交换机，然后交换机通过RoutingKey来寻找队列，将消息存入队列中**



![image-20220428202754505](D:\学习笔记\图片\image-20220428202754505.png)



### 交换机的类型

1. 直接（direct）
2. 主题（topic）
3. 标题（headers）
4. 扇出（fanout）





### fanout模式（发布-订阅模式）

**就是同一条消息通过交换机发送到不同队列，相当于广播的效果**

实现的步骤：

1. 获取一个信道
2. 声明一个fonout类型的交换机
3. 生成一个队列
4. 绑定交换机和队列（通过routingkey）




#### 接收者的代码（消费者）

```java
// 发布-订阅模式的接收者（消费者）
public class ReceiveLog {
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 获取交换机 声明成 fanout模式 可以到生产者去声明
        // channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        // 生成随机简单队列
        String queue = channel.queueDeclare().getQueue();
        // 绑定交换机和队列 声明这个交换机找到这个队列 使用的routingKey为""
        channel.queueBind(queue, EXCHANGE_NAME, "");
        // 接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("C2接收到的消息:" + new String(message.getBody()));
        channel.basicConsume(queue, true, deliverCallback, consumerTag -> {});
    }
}

```



#### 发送者的代码（生产者）

```java
// 发布-订阅模式的发布者（生成者）
public class EmitLog {
    // 声明的交换机的名称
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 获取交换机 声明成 fanout模式 可以到生产者去声明
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            // 发送消息到交换机 交换机通过routingKey来寻找相应的队列
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println(message + "消息发送完毕");
        }

    }
}

```



**最终的效果为:生产者发送消息将会分到两个队列中，都会得到这些消息，这里是在消费者的一方进行的队列的声明，并且是简单随机队列，可以怎么理解一个队列和一个消费者进行了绑定，所以不存在之前的轮询或者不公平分放的情况，这里生产者声明的是交换机**



<font color=red size=5>绑定情况，web图</font>

![image-20220428214450979](D:\学习笔记\图片\image-20220428214450979.png)



### Direct模式（路由模式）

**本质和fonout是一样的，都是交换机通过routingKeys来寻找队列，将信息发送到指定的队列，只是fonout模式的routingKeys是一样，而direct的routingKeys可以一样也可以不一样，即想发给谁就发给谁。**



#### 发送者的代码（生产者）

```java
/**
 * @author zyq
 * @version 1.0
 * Direct模式（路由器的路由模式）
 */
// 生产者
public class EmitLog {
    // 声明的交换机的名称
    public static final String EXCHANGE_NAME = "receive_logs";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 获取交换机 声明成 direct(路由) 模式 可以到生产者去声明
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            // 发送消息到交换机 交换机通过routingKey来寻找相应的队列
            channel.basicPublish(EXCHANGE_NAME, "error", null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println(message + "消息发送完毕");
        }

    }
}

```

#### 接收者的代码（消费者）

```java
/**
 * @author zyq
 * @version 1.0
 * Direct模式（路由模式）
 */
// 接收者（消费者）
public class ReceiveLog {
    // 定义交换机的名称
    public static final String EXCHANGE_NAME = "receive_logs";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMQUtils.getChannel();
        // 声明队列
        channel.queueDeclare("console", false, false, false, null);

        // 进行绑定将这个队列和这个交换机的指定的routingKey(info)进行绑定
        // 也可以进行多种绑定对同一个队列
        channel.queueBind("console", EXCHANGE_NAME, "info");
        channel.queueBind("console", EXCHANGE_NAME, "walling");
        // 接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("C2接收到的消息:" + new String(message.getBody()));
        channel.basicConsume("console", true, deliverCallback, consumerTag -> {
        });

    }
}

```



### Topics模式（主题模式）

**注意事项**

1. 发送到topics的交换机的信息的routingKeys不可以随便写，必须是一个单词的列表，并且用点进行分隔，例如：one.two.three等等，但是长度最长不可以超过255个字节
2. routingKeys存在两个替换符
3.  *（星号）可以代替一个单词
4. #（井号）可以代替0个或者多个单词



* 当routingKeys为#的时候相当于发布-订阅模式
* 当toutingKeys没有* 和 # 的时候相当路由模式
* 本质还是一样的，主要相当于加了通配符，更加的灵活



#### 发送者的代码（生产者）

```java
// 生产者
public class EmitLog {
    // 声明的交换机的名称
    public static final String EXCHANGE_NAME = "topics_logs";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 获取交换机 声明成 topic(主题) 模式 可以到生产者去声明
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            // 发送消息到交换机 交换机通过routingKey来寻找相应的队列
            channel.basicPublish(EXCHANGE_NAME, "one.two.three", null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println(message + "消息发送完毕");
        }

    }
}

```



#### 接收者的代码（消费者）

```java
// 接收者（消费者）
public class ReceiveLog1 {
    // 定义交换机的名称
    public static final String EXCHANGE_NAME = "topics_logs";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMQUtils.getChannel();
        // 声明队列
        channel.queueDeclare("Q1", false, false, false, null);

        // 进行绑定将这个队列和这个交换机的指定的routingKey(info)进行绑定
        // 也可以进行多种绑定对同一个队列
        // 这个就是匹配三个单词 中间单词是two的RoutingKeys
        channel.queueBind("Q1", EXCHANGE_NAME, "*.two.*");
        // 配置开头只要是one就可以
        channel.queueBind("Q1", EXCHANGE_NAME, "one.#");
        // 接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("C1接收到的消息:" + new String(message.getBody()));
        channel.basicConsume("Q1", true, deliverCallback, consumerTag -> {
        });

    }
}

```



### 总结

**<font color=red size=4>记住交换机的特点，生产者发消息都是发给交换机的，然后交换机通过routingKeys进行找和自己相绑定的队列，然后将消息发送给队列，这几种模式，本质都是这样的，无非就是routingkeys的不同，或者加上通配符让roudingKeys更加的灵活</font>**



## 死信队列

### 死信的概念：

**死信：就是在队列中没有消费者处理的消息或者处理不了消息**



### 死信的来源

1. 消息TTL过期
2. 队列达到最大长度（队列已满，无法添加新的消息）
3. 消息被拒绝（basic.reject, basic.nack），并且不放会队列中(requeue=false)



### 死信实战

#### 代码架构图

![image-20220429095342025](D:\学习笔记\图片\image-20220429095342025.png)



#### 消费者C1的代码

```java
// 消费者01
public class ReceiveLog01 {
    // 死信交换机的名称
    public static final String DEAD_EXCHANGE = "dead_exchange";
    // 普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 声明死信交换机
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
        // 设置死信交换机和死信队列
        Map<String, Object> arguments = new HashMap<>();
        // 设置死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 设置死信交换机的routingKeys
        arguments.put("x-dead-letter-routing-key", "dead");
        // 声明普通的队列
        channel.queueDeclare("normal_queue", false, false, false, arguments);
        // 进行普通交换机和普通队列的绑定
        channel.queueBind("normal_queue", NORMAL_EXCHANGE, "normal");
        // 接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("C1接收到的消息:" + new String(message.getBody()));
        channel.basicConsume("normal_queue", true, deliverCallback, consumerTag -> {
        });
    }
}

```

#### 消费者C2的代码

```java
// 消费者02
public class ReceiveLog02 {
    // 死信交换机的名称
    public static final String DEAD_EXCHANGE = "dead_exchange";

    public static void main(String[] args) throws IOException {
        // 获取信道
        Channel channel = RabbitMQUtils.getChannel();
        // 声明死信的队列
        channel.queueDeclare("dead_queue", false, false, false, null);
        // 进行死信交换机和死信队列的绑定
        channel.queueBind("dead_queue", DEAD_EXCHANGE, "dead");
        // 接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("C2接收到的消息:" + new String(message.getBody()));
        channel.basicConsume("dead_queue", true, deliverCallback, consumerTag -> {
        });
    }
}

```

#### 生产者的代码

```java
// 生产者
public class EmitLog {
    // 普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] args) throws IOException {
        Channel channel = RabbitMQUtils.getChannel();
        // 声明普通交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        // 发送10条消息
        for (int i = 1; i <= 10; i++) {
            String message = "info" + i;
            // 设置消息的过期时间 为 10s 当时间过了没有被消费 变成死信队列
            AMQP.BasicProperties basicProperties =
                    new AMQP.BasicProperties()
                            .builder()
                            .expiration("10000")
                            .build();
            channel.basicPublish(NORMAL_EXCHANGE, "normal", basicProperties, message.getBytes());
        }
    }
}

```



#### 队列达到最大长度成为死信

**将生产者的设置时间可以先删除**

**在C1消费者中，在map中加一个 map.put("x-max-lenght", 6) 这里是最大6个**

```java
// 声明普通的队列
channel.queueDeclare("normal_queue", false, false, false, arguments);
// 进行普通交换机和普通队列的绑定
channel.queueBind("normal_queue", NORMAL_EXCHANGE, "normal");
```



![image-20220429112842246](D:\学习笔记\图片\image-20220429112842246.png)



**<font color=red size=4>注：如果修改了参数，需要将之前的队列进行删除</font>**



#### 消息被拒绝变成死信

**实现步骤**

1. 将消息应答设置成手动应答
2. 在应答的时候手动拒绝（channel.basicReject|channel.basicNack）

```java
 // 在ReceiveLog01类中
// 接收消息
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            String msg = new String(message.getBody());
            if ("info5".equals(msg)) {
                // 如果消息是info5将进行拒绝 并且不能重新放回队列 将其变成死信
                channel.basicReject(message.getEnvelope().getDeliveryTag(), false);
            } else {
                channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
                //channel.basicNack(message.getEnvelope().getDeliveryTag(), false, false);

                System.out.println("C1接收到的消息:" + msg);
            }
        };
        channel.basicConsume("normal_queue", false, deliverCallback, consumerTag -> {
        });
```



![image-20220429144735740](D:\学习笔记\图片\image-20220429144735740.png)



## rabbitmq整合springboot

### 导入相关的依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.73</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```



### 配置文件application.yaml

```yaml
spring:
  rabbitmq:
    username: root
    password: root
    port: 5672
    host: 192.168.1.22

  mvc:
    pathmatch:
      matching-strategy:
        ANT_PATH_MATCHER
```



### 代码架构图

![image-20220429192507250](D:\学习笔记\图片\image-20220429192507250.png)



**注：整合了springboot后，不需要在消费者上声明交换机和队列，全部交给一个配置类来完成，生产者就负责生产消息，消费者就负责消费消息即可，让代码更加的明朗，更加的清晰**



### 配置文件类

```java
/**
 * @author zyq
 * @version 1.0
 */
@Configuration
public class TTLQueueConfig {
    // 声明普通的交换机的名称
    public static final String X_EXCHANGE = "x_exchange";
    // 声明延迟交换机的名称
    public static final String Y_DEAD_EXCHANGE = "y_exchange";
    // 声明普通的队列
    public static final String A_QUEUE = "a_queue";
    public static final String B_QUEUE = "b_queue";
    // 声明延迟队列
    public static final String DEAD_QUEUE = "d_queue";

    // 向容器中存放一个普通交换机
    @Bean("xExchange")
    public DirectExchange xExchange() {
        return new DirectExchange(X_EXCHANGE);
    }

    // 向容器中存放一个延迟交换机
    @Bean("yExchange")
    public DirectExchange yExchange() {
        return new DirectExchange(Y_DEAD_EXCHANGE);
    }

    // 普通队列
    @Bean("aQueue")
    public Queue aQueue() {
        Map<String, Object> arguments = new HashMap<>(3);
        // 设置延迟交换机
        arguments.put("x-dead-letter-exchange", Y_DEAD_EXCHANGE);
        // 设置routingKeys
        arguments.put("x-dead-letter-routing-key", "YD");
        // 设置TTL 为 10s
        arguments.put("x-message-ttl", 10000);
        return QueueBuilder.durable(A_QUEUE).withArguments(arguments).build();
    }
    // 普通队列
    @Bean("bQueue")
    public Queue bQueue() {
        Map<String, Object> arguments = new HashMap<>(3);
        // 设置延迟交换机
        arguments.put("x-dead-letter-exchange", Y_DEAD_EXCHANGE);
        // 设置routingKeys
        arguments.put("x-dead-letter-routing-key", "YD");
        // 设置TTL 为 10s
        arguments.put("x-message-ttl", 40000);
        return QueueBuilder.durable(B_QUEUE).withArguments(arguments).build();
    }
    // 延迟队列
    @Bean("d_queue")
    public Queue dQueue() {
        return QueueBuilder.durable(DEAD_QUEUE).build();
    }

    // 声明绑定关系
    @Bean
    public Binding aQueueBingingX(@Qualifier("aQueue") Queue aQueue,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(aQueue).to(xExchange).with("XA");
    }

    // 声明绑定关系
    @Bean
    public Binding bQueueBingingX(@Qualifier("bQueue") Queue bQueue,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(bQueue).to(xExchange).with("XB");
    }

    // 声明绑定关系
    @Bean
    public Binding dQueueBingingX(@Qualifier("d_queue") Queue d_queue,
                                  @Qualifier("yExchange") DirectExchange yExchange) {
        return BindingBuilder.bind(d_queue).to(yExchange).with("YD");
    }
    

}

```



### 生产者的代码

```java
/**
 * @author zyq
 * @version 1.0
 * 生产者
 */
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {
    // 使用springboot自动导入的类
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable("message") String message) {
        log.info("当前时间为:{},发送的消息是:{}", new Date().toString(), message);
        // 发送消息
        // 发送到x_exchange这个交换机 的 routingKey (XA)的队列 aQueue 延迟10s的队列
        rabbitTemplate.convertAndSend("x_exchange", "XA", "延迟10s的队列" + message);
        rabbitTemplate.convertAndSend("x_exchange", "XB", "延迟40s的队列" + message);
    }
}

```



### 消费者代码

```java
/**
 * @author zyq
 * @version 1.0
 * 延迟队列消费者
 */
@Component
@Slf4j
public class TTLConsumer {
    @RabbitListener(queues = "d_queue")
    public void receiveD(Message message, Channel channel) {
        String msg = new String(message.getBody());
        log.info("当前时间：{}， 接收到的消息：{}", new Date().toString(), msg);
    }
}

```



### 运行情况

**在浏览器中请求http://localhost:8080/ttl/sendMsg/helloworld**

![image-20220429212714095](D:\学习笔记\图片\image-20220429212714095.png)



### 延迟队列的优化

**之前是在创建队列的时候，就已经指定好了TTL，非常的不灵活如果需要一个新的TTL又需要写一个队列，这个肯定是不可取的，是不可以的，优化成一个可以接受自定义时间的延迟队列**



在配置类（TTLQueueConfig）中加入一下代码

```java
// 声明一个可以接受TTL的普通队列
    public static final String C_QUEUE = "c_queue";

    // 普通队列
    @Bean("cQueue")
    public Queue cQueue() {
        Map<String, Object> arguments = new HashMap<>(2);
        // 设置延迟交换机
        arguments.put("x-dead-letter-exchange", Y_DEAD_EXCHANGE);
        // 设置routingKeys
        arguments.put("x-dead-letter-routing-key", "YD");
        return QueueBuilder.durable(C_QUEUE).withArguments(arguments).build();
    }

    // 声明绑定关系
    @Bean
    public Binding cQueueBingingX(@Qualifier("cQueue") Queue cQueue,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(cQueue).to(xExchange).with("XC");
    }

```



在SendMsgController(生产者)加入以下代码

```java
@GetMapping("/sendMsg/{message}/{TTLTime}")
    public void sendMsg(@PathVariable("message") String message,
                        @PathVariable("TTLTime") String TTLTime) {
        log.info("当前时间为:{},发送的消息的TTL为:{}ms 发送的消息是:{}", new Date().toString(), TTLTime, message);
        // 发送消息
        rabbitTemplate.convertAndSend("x_exchange", "XC", message,
                msg -> {
                    msg.getMessageProperties().setExpiration(TTLTime);
                    return msg;
                });
    }
```



**进行测试，发起两次请求 http://localhost:8080/ttl/sendMsg/java/20000 **

**http://localhost:8080/ttl/sendMsg/javaweb/2000**

![image-20220429222348611](D:\学习笔记\图片\image-20220429222348611.png)



**因为RabbitMQ只会检查队列中第一个消息是否过期，如果过期则丢到死信队列， 如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行**！

**<font color=red>自己理解：因为这个是队列的数据结构，只有等排在前面的先进入死信队列中，后面的才有机会，即使他已经到了过期时间，也无法进入死信队列，只有前面的ttl小这种情况才可以正常使用，但肯定是不可取的</font>**



### 解决延迟队列问题（通过插件）

#### 下载延迟插件

<https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.8.9/rabbitmq_delayed_message_exchange-3.8.9-0199d11c.ez>



#### 将延迟插件放到RabbitMQ的插件目录下

```bash
cp rabbitmq_delayed_message_exchange-3.8.9-0199d11c.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.12/plugins/
```



#### 安装并且重启服务

```bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
systemctl restart rabbitmq-server #重启服务
```

![image-20220429225002944](D:\学习笔记\图片\image-20220429225002944.png)

**安装成功后，发现多了一个交换机的类型**



#### 代码构架图

![image-20220430092432946](D:\学习笔记\图片\image-20220430092432946.png)



#### 配置类代码

```java
@Configuration
public class DelayExchangeConfig {
    // 定义延迟交换机的名称
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    // 定义延迟队列的名称
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    // 定义routingKeys的名称
    public static final String DELAYED_ROUTING_KEY = "delayed.routingKey";

    // 定义延迟交换机
    @Bean
    public CustomExchange delayedExchange() {
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-delayed-type", "direct");
        /*
         * 方法的参数
         * String name, String type, boolean durable,
         * boolean autoDelete, Map<String, Object> arguments
         */
        return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message",
                true, false, arguments);
    }

    // 定义延迟队列
    @Bean
    public Queue delayedQueue() {
        return new Queue(DELAYED_QUEUE_NAME);
    }

    // 定义绑定关系
    @Bean
    public Binding delayedQueueBindingDelayedExchange(
            @Qualifier("delayedExchange") CustomExchange delayedExchange,
            @Qualifier("delayedQueue") Queue delayedQueue
    ) {
        return BindingBuilder.bind(delayedQueue).to(delayedExchange)
                .with(DELAYED_ROUTING_KEY).noargs();
    }
}

```

#### 生产者代码

```java
// 在SendMsgController类上添加这个方法即可

@GetMapping("/sendDelayMsg/{message}/{delayTime}")
    public void sendMsg(@PathVariable("message") String message,
                        @PathVariable("delayTime") Integer delayTime) {
        log.info("当前时间为:{},发送的消息的delay message 为:{}ms 发送的消息是:{}", new Date().toString(), delayTime, message);
        // 发送消息
        rabbitTemplate.convertAndSend(DelayExchangeConfig.DELAYED_EXCHANGE_NAME,
                DelayExchangeConfig.DELAYED_ROUTING_KEY, message,
                msg -> {
                    msg.getMessageProperties().setDelay(delayTime);
                    return msg;
                });
    }
```



#### 消费者代码

```java
// 消费者
@Slf4j
@Component
public class DelayedConsumer {
    @RabbitListener(queues = DelayExchangeConfig.DELAYED_QUEUE_NAME)
    public void receive(Message message) {
        String msg = new String(message.getBody(), StandardCharsets.UTF_8);
        log.info("当前时间为：{}, 接受到的消息为{}", new Date().toString(), msg);
    }
}

```



#### 测试结果

**在浏览器发起两次请求分别是：<http://localhost:8080/ttl/sendDelayMsg/java/20000>**

**<http://localhost:8080/ttl/sendDelayMsg/javaweb/2000>**

![image-20220430102548907](D:\学习笔记\图片\image-20220430102548907.png)



***<font color=blue>发现结果和我们需要的结果是一样的，出现的原因是将延迟从队列改到了交换机</font>***



## 发布确认高级

**考虑当服务器宕机了，rabbitmq进行重启的过程中的消息，如何保证消息不丢失，之前的如果rabbitmq重启了，直接就无法发送消息，直接会报异常，无法连接上**

![image-20220430111140307](D:\学习笔记\图片\image-20220430111140307.png)





### 发布确认-springboot版

#### 配置类

```java
// 消息确认的高级版
@Configuration
public class ConfirmConfig {
    // 定义交换机名称
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    // 定义队列名称
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    // 定义绑定的名称
    public static final String CONFIRM_ROUTING_KEY = "key1";

    // 定义一个交换机
    @Bean
    public DirectExchange confirmExchange() {
        return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }
    // 定义一个队列
    @Bean
    public Queue confirmQueue() {
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }
    // 定义绑定关系
    @Bean
    public Binding confirmQueueBindingConfirmExchange(
            @Qualifier("confirmExchange") DirectExchange confirmExchange,
            @Qualifier("confirmQueue") Queue confirmQueue) {
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTING_KEY);
    }
}

```



#### 配置文件

```yaml
spring:
  rabbitmq:
    username: root
    password: root
    port: 5672
    host: 192.168.1.22
    publisher-confirm-type: correlated

  mvc:
    pathmatch:
      matching-strategy:
        ANT_PATH_MATCHER
```



#### 生产者代码

```java
// 发布确认的高级版
@RestController
@RequestMapping("/confirm")
@Slf4j
public class ConfirmController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable("message") String message) {
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME,
                ConfirmConfig.CONFIRM_ROUTING_KEY, message);
        log.info("发送的消息为：{}", message);
    }
}
```



#### 消费者代码

```java
@Component
@Slf4j
public class ConfirmConsumer {
    @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE_NAME)
    public void receive(Message message) {
        String msg = new String(message.getBody());
        log.info("接受到的消息为：{}", msg);
    }
}

```



**现在这么写，只是一个简单的队列的传递消息，并没有回调的功能**



#### 回调接口配置类（交换机）

**无论交换机是否基于得到消息，都会调用这个回调接口的回调方法**

```java
// 无论交换机是否可以得到消息 都会调用的回调类
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    // 进行注入到容器中的RabbitTemplate.ConfirmCallback这个类中
    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(this);
    }

    /**
     * @param correlationData 交换机回调回来的相关信息
     * @param b               true 代表 交换机已经接受 false代表失败
     * @param s               如果出现错误，错误的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean b, String s) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (b) {
            // 代表交换机成功接受到消息
            log.info("交换机接受到的消息的id为:{}", id);
        } else {
            // 代表交换机接受消息失败
            log.info("交换机没有接受到的消息id为:{}", id);
        }
    }
}

```



#### 生产者代码修改

```java
// 发布确认的高级版
@RestController
@RequestMapping("/confirm")
@Slf4j
public class ConfirmController {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable("message") String message) {
        // 这个是在有回调接口的时候，第一个参数的内容就是这个
        CorrelationData correlationData = new CorrelationData("1");
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME,
                ConfirmConfig.CONFIRM_ROUTING_KEY, message, correlationData);
        log.info("发送的消息为：{}", message);
    }
}

```



#### 测试结果

**当交换机不存在的时候，也可以知道是那个消息没有接受到，但是重启服务器还是连接不上**

![image-20220430131803301](D:\学习笔记\图片\image-20220430131803301.png)





### 回退消息

**即当路由器可以正常获到信息，但是无法路由，即无法传到指定的队列，这个时候我们需要保证数据不可以丢失，这个时候可以实现一个接口，在失败的时候进行回调**



#### 配置文件

```yaml
spring:
  rabbitmq:
    username: root
    password: root
    port: 5672
    host: 192.168.1.22
    publisher-confirm-type: correlated
    publisher-returns: true

  mvc:
    pathmatch:
      matching-strategy:
        ANT_PATH_MATCHER
```



#### 配置类

```java
// 无论交换机是否可以得到消息 都会调用的回调类
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    // 进行注入到容器中的RabbitTemplate.ConfirmCallback这个类中
    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }

    /**
     * @param correlationData 交换机回调回来的相关信息
     * @param b               true 代表 交换机已经接受 false代表失败
     * @param s               如果出现错误，错误的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean b, String s) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (b) {
            // 代表交换机成功接受到消息
            log.info("交换机接受到的消息的id为:{}", id);
        } else {
            // 代表交换机接受消息失败
            log.info("交换机没有接受到的消息id为:{}", id);
        }
    }

    // 当交换机无法进行路由的时候 会执行的回调方法
    @Override
    public void returnedMessage(ReturnedMessage returned) {
        log.error("交换机{}没有路由到{}的消息为{}，出现的原因为{}",
                returned.getExchange(), returned.getRoutingKey(), new String(returned.getMessage().getBody()),
                returned.getReplyText());
    }
}

```



#### 测试结果

当指定的队列不存在的时候，会调用这个回调函数

![image-20220501100343287](D:\学习笔记\图片\image-20220501100343287.png)



## 备份交换机

### 代码架构图

![image-20220501102133765](D:\学习笔记\图片\image-20220501102133765.png)

 

### 修改配置类

```java
// 消息确认的高级版
@Configuration
public class ConfirmConfig {
    // 定义交换机名称
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    // 定义队列名称
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    // 定义绑定的名称
    public static final String CONFIRM_ROUTING_KEY = "key1";
    // 定义一个备份交换机
    public static final String BACKUP_EXCHANGE_NAME = "backup.exchange";
    // 定义一个备份队列
    public static final String BACKUP_QUEUE_NAME = "backup.queue";
    // 定义一个警告队列
    public static final String WARNING_QUEUE_NAME = "warning.queue";


    // 定义一个交换机 
    @Bean("confirmExchange")
    public DirectExchange confirmExchange() {
        Map<String, Object> arguments = new HashMap<>();
        // 绑定备份交换机
        arguments.put("alternate-exchange", BACKUP_EXCHANGE_NAME);
        return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME)
                .durable(true).withArguments(arguments).build();
    }

    // 定义一个队列
    @Bean("confirmQueue")
    public Queue confirmQueue() {
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    // 定义绑定关系
    @Bean
    public Binding confirmQueueBindingConfirmExchange(
            @Qualifier("confirmExchange") DirectExchange confirmExchange,
            @Qualifier("confirmQueue") Queue confirmQueue) {
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTING_KEY);
    }

    // 定义一个备份交换机
    @Bean("backupExchange")
    public FanoutExchange backupExchange() {
        return ExchangeBuilder.fanoutExchange(BACKUP_EXCHANGE_NAME)
                .durable(true).build();
    }

    // 定义一个备份队列
    @Bean("backupQueue")
    public Queue backupQueue() {
        return new Queue(BACKUP_QUEUE_NAME);
    }

    // 定义一个警告队列
    @Bean("warningQueue")
    public Queue warningQueue() {
        return new Queue(WARNING_QUEUE_NAME);
    }

    // 定义绑定关系
    @Bean
    public Binding backupQueueBindingBackupExchange(
            @Qualifier("backupExchange") FanoutExchange backupExchange,
            @Qualifier("backupQueue") Queue backupQueue) {
        return BindingBuilder.bind(backupQueue).to(backupExchange);
    }

    // 定义绑定关系
    @Bean
    public Binding warningQueueBindingBackupExchange(
            @Qualifier("backupExchange") FanoutExchange backupExchange,
            @Qualifier("warningQueue") Queue warningQueue) {
        return BindingBuilder.bind(warningQueue).to(backupExchange);
    }

}

```



### 测试结果

![image-20220501111538269](D:\学习笔记\图片\image-20220501111538269.png)



**如果同时配置了备用交换机和对应的交换机的回调接口，那以备用交换机为主**



## 优先级队列

**将一些重要的消息优先进行执行，消费者那到消息后进行排序，先执行优先级高的消息，优先级的选择（0-255）值越大，优先级越高**



### 图形界面设置优先级队列

![image-20220501114608016](D:\学习笔记\图片\image-20220501114608016.png)



### 代码实现

**<font color=red>注：需要先将数据存储到队列中，不能一边存储到队列，一边进行消费，不能速度太快，无法进行排序</font>**



### 生产者代码

```java
public class Producer {
    // 创建 队列 队列的名为hello
    public static final String QUEUE = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置工厂的ip
        factory.setHost("192.168.1.22");
        // 设置用户名
        factory.setUsername("root");
        // 设置密码
        factory.setPassword("root");
        // 创建连接
        Connection connection = factory.newConnection();
        // 获取连接中的信道
        Channel channel = connection.createChannel();
        // String var1, boolean var2, boolean var3, boolean var4, Map<String, Object> var5
        /*
         * 生成队列
         * queueDeclare方法的参数：
         * 第一个参数：队列名称 从那个队列中拿东西
         * 第二个参数：队列里的消息是否持久化（是否存入磁盘） true表示存入 默认false 存在内存
         * 第三个参数：该队列是否只能一个消费者消费 true 表示一个 false 表示只能多个
         * 第四个参数：是否自动删除 在最后一个消费者开连接之后 队列是否自动删除 true表示自动删除 false 表示不自动删除
         * 第五个参数：其他参数
         */
        Map<String, Object> arguments = new HashMap<>();
        // 设置队列的最大的优先级 可以选择的是0-255 这里选择 0-10
        arguments.put("x-max-priority", 10);
        channel.queueDeclare(QUEUE, false, false, false, arguments);
        // 需要发送的消息
        // String message = "hello world";
        // 通过信道发送消息到队列中
        /*
         * basicPublish的参数
         * 第一个参数：发送到那一个交换机
         * 第二个参数：路由的key是那个 这次是队列的名称
         * 第三个参数：其他消息
         * 第四个参数：需要发送的消息
         */
        // 发送多条数据
        for (int i = 1; i < 11; i++) {
            String message = "info" + i;
            if (i == 6) {
                // 设置优先级 为 6
                AMQP.BasicProperties properties =
                        new AMQP.BasicProperties()
                        .builder().priority(6).build();
                channel.basicPublish("", QUEUE, properties, message.getBytes());
            } else {
                channel.basicPublish("", QUEUE, null, message.getBytes());
            }
        }
        System.out.println("消息发送到队列了");
    }
}

```

### 消费者代码

```java
public class Consumer {
    // 队列的名称
    public static final String QUEUE = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 获取连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.1.22");
        factory.setUsername("root");
        factory.setPassword("root");
        // 获取连接
        Connection connection = factory.newConnection();
        // 获取信道
        Channel channel = connection.createChannel();

        /*
         * basicConsume的参数
         * 第一个参数：消费的队列
         * 第二个参数（autoAck）：消费成功后是否要自动应答 true 自动应答 false 手动应答
         * 第三个参数（deliverCallback）：当传送回来的消息进行的回调 即就是正常的获取到了消息
         * 第四个参数（CancelCallback）：当收费者取消了对消息的接受 回调的方法
         */
        DeliverCallback deliverCallback
                = (consumerTag, message) -> System.out.println(new String(message.getBody()));
        CancelCallback cancelCallback
                = consumerTag -> System.out.println("消费者取消了接收参数");
        channel.basicConsume(QUEUE, true, deliverCallback, cancelCallback);
    }
}

```



 ### 运行结果

![image-20220501120919064](D:\学习笔记\图片\image-20220501120919064.png)



## 集群

### 集群搭建的架构图

![image-20220501151523227](D:\学习笔记\图片\image-20220501151523227.png)

### 集群搭建步骤

```bash
# 设置三台主机并设置主机名
vi /etc/hostname   # 设置主机名

# 配置个结点的hosts文件，使之可以认识
vi /etc/hosts
192.168.1.22 node1
192.168.1.23 node2
192.168.1.24 node3

# 以确保各个节点的cookie文件使用的是同一个值：在node1上执行远程操作命令
scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie

# 启动RabbitMQ服务，顺带启动Erlang虚拟机和RabbitMQ应用服务，三台节点下执行
rabbitmq-server -detached

# 在节点2执行
# rabbitmqctl stop会将Erlang虚拟机关闭  rabbitmqctl stop_app 只关闭rabbitmq服务
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node1
# 只启动rabbitmq服务
rabbitmqctl start_app

# 在三号节点执行
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node2
rabbitmqctl start_app

# 查看集群状态
rabbitmqctl cluster_status

# 需要重新设置用户 在任意一个主机上 一个设置 整个集群有效
# 创建账号
rabbitmqctl add_user root root
# 设置用户角色
rabbitmqctl set_user_tags root administrator
# 设置用户权限
rabbitmqctl set_permissions -p "/" root ".*" ".*" ".*" 

# 脱离集群节点的节点，node2 和 node3分别执行
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
rabbitmqctl cluster_status
# 此项命令均在node1上执行 忘记node2这个节点
rabbitmqctl forget_cluster_node rabbit@node2

```



**出现这个表示集群搭建完毕，访问任意一个节点**

![image-20220501153030598](D:\学习笔记\图片\image-20220501153030598.png)





### 镜像队列

**镜像队列就是一台主机上的队列或者交换机，会被备份一份或者多份到其他的主机上，保证一台主机宕机了，保证消息不会丢失**



设置方法

![image-20220501155440389](D:\学习笔记\图片\image-20220501155440389.png)



**<font color=red size=4>注：^mirror 表明队列的名称只能以mirror开头的队列才有效，当这个服务器宕机，虽然在其他的服务器有备份，但是访问的时候只能以其他的ip进行消费</font>**



### 解决ip需要换的问题（nginx+keepalive）实现高可用

![image-20220501161350788](D:\学习笔记\图片\image-20220501161350788.png)



## Federation Exchange

### 使用场景

(broker北京)，(broker深圳)彼此之间相距甚远，网络延迟是一个不得不面对的问题。

有一个在北京的业务(Client北京)需要连接(broker北京),向其中的交换器exchangeA.发送消息，此时的网络延迟很小,(Client北京)可以迅速将消息发送至exchangeA.中，就算在开启了publisherconfirm.机制或者事务机制的情况下，也可以迅速收到确认信息。此时又有个在深圳的业务(Client深圳)需要向exchangeA发送消息，那么(Client深圳)(broker北京)之间有很大的网络延迟，(Client深圳)将发送消息至exchangeA会经历一定的延迟，尤其是在开启了publisherconfirm.机制或者事务机制的情况下，(Client深圳)会等待很长的延迟时间来接收(broker北京)的确认信息，进而必然造成这条发送线程的性能降低，甚至造成一定程度上的阻塞。

将业务(Client深圳)部署到北京的机房可以解决这个问题，但是如果(Client深圳)调用的另些服务都部署在深圳，那么又会引发新的时延问题，总不见得将所有业务全部部署在一个机房，那么容灾又何以实现?这里使用Federation插件就可以很好地解决这个问题.



### 搭建步骤

```bash
# 每台节点均需执行以下命令
rabbitmq-plugins enable rabbitmq_federation
rabbitmq-plugins enable rabbitmq_federation_management

```

![image-20220501162439418](D:\学习笔记\图片\image-20220501162439418.png)



### 原理图

**需要下游的交换机先存在，才可以进行**

![image-20220501162902839](D:\学习笔记\图片\image-20220501162902839.png)



### 实现步骤

1. 在下游创建fed_exchange交换机

2. 在下游节点（node2）配置上游节点（node1）

   ![image-20220501165404917](D:\学习笔记\图片\image-20220501165404917.png)

3. 添加policy

![image-20220501165706553](D:\学习笔记\图片\image-20220501165706553.png)

4. 成功的图

![image-20220501165737699](D:\学习笔记\图片\image-20220501165737699.png)

## Federation Queue

### 实现步骤

1. 在下游创建对应的队列
2. 在下游节点（node2）配置上游节点（node1）（和上面一样）
3. 添加policy

![image-20220501171340878](D:\学习笔记\图片\image-20220501171340878.png)



## Shovel

### 使用场景

Federation具备的数据转发功能类似，Shovel够可靠、持续地从一个Broker中的队列(作为源端，即source)拉取数据并转发至另一个Broker中的交换器(作为目的端，即destination)。

作为源端的队列和作为目的端的交换器可以同时位于同一个Broker，也可以位于不同的Broker上。

Shovel可以翻译为"铲子",是一种比较形象的比喻，这个"铲子"可以将消息从一方"铲子"另一方。Shovel行为就像优秀的客户端应用程序能够负责连接源和目的地、负责消息的读写及负责连接失败问题的处理。



### 原理图

![image-20220501171650683](D:\学习笔记\图片\image-20220501171650683.png)



### 搭建步骤

```bash
# 在需要的主机上安装
rabbitmq-plugins enable rabbitmq_shovel
rabbitmq-plugins enable rabbitmq_shovel_management
```

![image-20220501172159851](D:\学习笔记\图片\image-20220501172159851.png)

添加shevel源和目的地

![image-20220501172844081](D:\学习笔记\图片\image-20220501172844081.png)



**表示成功**

![image-20220501172917703](D:\学习笔记\图片\image-20220501172917703.png)