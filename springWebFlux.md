# springWebFlux

## 什么是springwebflux

**一个异步的非阻塞的框架，在servlet3.1以后才进行的支持，核心是基于Reactor的相关的API实现的**

1. 异步和同步：可以相对于调用者来进行理解，当调用者发送请求，一直等着对方的回应才去做其他的事，这个就是同步。如果调用者在等待的情况下做自己的事情，就是异步进行。
2. 阻塞和非阻塞：可以从被调用者来进行理解，当调用者发起请求，被调用者将请求做完再进行回应，这个就是阻塞。被调用者收到请求直接进行回应就是非阻塞。



**springMVC和springwebflux的区别**

![image-20220502171122316](D:\学习笔记\图片\image-20220502171122316.png)

1. 两种框架都是使用注解的方式，都可以运行在tomcat等容器中
2. springmvc是基于命令式进行的编程（即一行一行的进行执行）
3. springwebflux是基于异步响应式编程的方式



## 响应式编程

**响应式编程是一个面向数据流和变化传播的编程范式**

### java简单实现响应式编程

```java
// 模拟响应式编程 创建观察者
public class ObserverDemo extends Observable {
    public static void main(String[] args) {
        // 创建ObserverDemo对象
        ObserverDemo observer = new ObserverDemo();
        // void update(Observable o, Object arg);
        // 添加观察者
        observer.addObserver((o, arg) -> {
            System.out.println("发生了变化");
        });
        observer.addObserver((o, arg) -> {
            System.out.println("被观察者通知，准备改变");
        });
        observer.setChanged(); // 当发生改变的时候
        observer.notifyObservers(); // 进行通知
    }
}

```

![image-20220502175812088](D:\学习笔记\图片\image-20220502175812088.png)