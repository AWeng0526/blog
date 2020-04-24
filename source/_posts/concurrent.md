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

当一个线程调用共享对象的wait()方法被阻塞挂起后,如果其他线程中断了该线程,则该线程会抛出InterruptedException异常并返回.

#### wait(long timeout)函数

如果一个线程调用共享对象的该方法挂起后,没有在指定的timeout ms内被其他线程调用该共享变量的notify()或notifyAll()方法唤醒,那么该函数还是会因超时而返回.

#### notify()函数

&emsp;&emsp;一个线程调用共享对象的notify()方法后,会唤醒一个在该共享变量的上调用wait系列方法后被挂起的线程.一个共享变量上可能会有多个线程在等待具体唤醒哪个等待的线程是随机的.

&emsp;&emsp;此外被唤醒的线程不能马上从wait方法返回并继续执行它必须在获取了共享对象的监视器锁后才可以返回也就是唤醒它的线程释放了共享变量上的监视器锁后被唤醒的线程也不一定会获取到共享对象的监视器锁这是因为该线程还需要和其他线程一起竞争该~Yi只有该线程竞争到了共享变量的监视器锁后才可以继续执行．

&emsp;&emsp;类似wait系列方法只有当前线程获取到了共享变量的监视器锁后才可以调用共享变量的notify()方法否则会抛出Illega!MonitorStateException异常．

#### notifyAll()函数

&emsp;&emsp;不同于在共享变量上调用notify()函数会唤醒被阻塞到该共享变量上的一个线程,notifyAll()方法则会唤醒所有在该共享变量上由于调用wait系列方法而被挂起的线程

### 等待线程执行终止的join方法

&emsp;&emsp;线程A调用线程B的join()方法后会被阻塞,直至线程B执行完毕并返回.当线程A被阻塞时,其他线程调用A的interrupt()方法中断了A线程时,线程A会抛出InterruptException异常.

### 让线程休眠的sleep方法

&emsp;&emsp;Thread类中有一个静态的sleep方法,当一个执行中的线程调用了Thread的sleep方法后,调用线程会暂时让出指定时间的执行权,也就是在这期间不参与CPU的调度,但是该线程所拥有的监视器资源,比如锁还是持有不让出的.指定的睡眠时间到了后该函数会正常返回,线程就处于就绪状态,然后参与CPU的调度,获取到CPU资源后就可以继续运行了.如果在睡眠期间其他线程调用了该线程的inten-upt()方法中断了该线程,则该线程会在调用sleep方法的地方抛出IntermptedException异常而返回.

&emsp;&emsp;当线程A在睡眠期间,另外一个线程调用线程A的interrupt()方法中断线程会抛出异常.

### 让出CPU执行权的yield方法

&emsp;&emsp;Thread类中有一个静态的yield方法,当一个线程调用yield方法时,实际就是在暗示线程调度器当前线程请求让出自己的CPU使用,但是线程调度器可以无条件忽略这个暗示.我们知道操作系统是为每个线程分配一个时间片来占有 CPU的,正常情况下当一个线程把分配给自己的时间片使用完后,线程调度器才会进行下一轮的线程调度,而当一个线程调用了Thread类的静态方法yield时,是在告诉线程调度器自己占有的时间片中还没有使用完的部分自己不想使用了,这暗示线程调度器现在就可以进行下一轮的线程调度.

&emsp;&emsp;当一个线程调用 yield 方法时,当前线程会让出CPU使用权,然后处于就绪状态,线程调度器会从线程就绪队列里面获取一个线程优先级最高的线程,当然也有可能会调度到刚刚让出CPU的那个线程来获取CPU执行权.

&emsp;&emsp;总结:sleep与yield方法的区别在于,当线程调用sleep方法时调用线程会被阻塞挂起指定的时间,在这期间线程调度器不会去调度该线程.而调用yield方法时,线程只是让出自己剩余的时间片,并没有被阻塞挂起,而是处于就绪状态,线程调度器下一次调度时就有可能调度到当前线程执行.

### 线程中断

Java中的线程中断是一种线程间的协作模式,通过设置线程的中断标志并不能直接终止该线程的执行,而是被中断的线程根据中断状态自行处理.

- void Iterrupt()方法:中断线程,例如,当线程A运行时,线程B可以调用线程A的interrupt()方法来设置线程A的中断标志为true并立即返回.设置标志仅仅是设置标志,线程A实际并没有被中断,它会继续往下执行.如果线程A因为调用了wait系列函数、join方法或者sleep方法而被阻塞挂起,这时候若线程B调用线程A的interrupt()方法,线程A会在调用这些方法的地方抛出InterruptedException异常而返回.

- boolean isInterrupted()方法：检测当前线程是否被中断,如果是返回true,否则返回false.

- boolean interrupted()方法：检测当前线程是否被中断,如果是返回true,否则返回false.与islnterrupted不同的是,该方法如果发现当前线程被中断,则会清除中断标志,并且该方法是static方法,可以通过Thread类直接调用.另外从下面的代码可以知道,在interrupted()内部是获取当前调用线程的中断标志而不是调用interrupted()方法的实例对象的中断标志.

```java
public static boolean interrupted() {
    //清除中断标志
    return currentThread().isinterrupted(true);
}
```
