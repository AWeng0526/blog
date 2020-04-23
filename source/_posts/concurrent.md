---
title: Java并发编程之美读书笔记
date: 2020-04-22 20:35:29
tags: [Java,并发]
categories: [技术,]
---

## 并发编程线程基础

### 线程创建与运行

Java中有三种线程创建方式,分别为实现Runnable接口的run 方法继承Thread类并重写run的方法,使用FutureTask方式.

<!--more-->

``` java

public static class Mythread extends Thread{
        @Override
        public void run() {
            System.out.println("I am a child thread");
        }
    }

public class RunableTask implements Runnable{
    @Override
    public void run() {
        System.out.println("I am a child thread");
    }
}

public class CallerTask implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "hello";
    }
    public static void main(String[] args) {
        // 创建异步任务
        FutureTask<String> futureTask = new FutureTask<String>(new CallerTask());
        // ...
    }
}
```

- 启动线程应当使用Thread.start(),直接调用run方法并不会开启一个新的线程.
- futureTask.get()可以获取线程返回值.

&emsp;&emsp;小结:使用继承方式的好处是方便传参,你可以在子类里面添加成员变量,通过set方法设置参数或者通过构造函数进行传递,而如果使用Runnable方式,则只能使用主线程里面被声明为final的变量.不好的地方是Java不支持多继承,如果继承了 Thread 类,那么子类不能再继承其他类,而 Runable 则没有这个限制.前两种方式都没办法拿到任务的返回结果,但是Futuretask方式可以.

### 线程通知与等待

#### wait()函数

&emsp;&emsp;当一个线程调用一个共享变量的wait()方法时,该调用线程会被阻塞挂起,直到发生下面几件事情之一才返回：(1）其他线程调用了该共享对象的notify()或者notifyAll()方法；(2）其他线程调用了该线程的interrupt()方法,该线程抛出InterruptedException异常返回.
&emsp;&emsp;另外需要注意的是,如果调用wait()方法的线程没有事先获取该对象的监视器锁,则调用wait()方法时调用线程会抛出IllegalMonitorStateException异常.

获取监视器锁的两种方法:

1. 执行synchronized同步代码块时,使用该共享变量作为参数
2. 调用该共享变量的方法,并且该方法使用了synchronized修饰

```java

synchronized (共享变量) {
    // ...
}

synchronized void add(int a,int b){
    // ...
}
```

&emsp;&emsp;假如某一线程通过synchronized获取到某一共享变量的锁,那么后续所有企图访问该元素的线程将会在获取该监视器锁的地方被挂起.

&emsp;&emsp;调用共享变量的wait()方法会释放获取到的锁,否则会引起死锁.另外需要注意的是,当前线程调用共享变量的 wait()方法后只会释放当前共享变量上的锁,如果当前线程还持有其他共享变量的锁,则这些锁是不会被释放的.
