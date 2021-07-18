---
title: ReentrantLock 和 synchronized
date: 2021-07-15 21:42:22
tags: [java, 锁]
categories: java
---

# ReentrantLock 和 synchronized
* synchronized 是独占锁，加锁和解锁的过程自动进行，易于操作，但不够灵活。ReentrantLock 也是独占锁，加锁和解锁的过程需要手动进行，不易操作，但非常灵活。
* synchronized 可重用，因为加锁和解锁的过程是自动进行，不必担心最后是否释放锁；ReentrantLock 也可重用，但加锁和解锁需要手动进行，且次数需一样，否则其他线程无法获得锁。
* synchronized 不可响应中断，一个线程获取不到锁就一直等着；ReentrantLock 可以响应中断。

## 使用
### 1、简单使用
```java
public class ReentrantLockTest {

  private static final Lock lock = new ReentrantLock();

  public static void main(String[] args) {
    new Thread(() -> test(), "线程A").start();
    new Thread(() -> test(), "线程B").start();
  }

  public static void test() {
    try {
      lock.lock();
      System.out.println(Thread.currentThread().getName() + "获得了锁。");
      TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      System.out.println(Thread.currentThread().getName() + "释放了锁。");
      lock.unlock();
    }
  }

}
```
### 2、公平锁实现
对于公平锁实现，就要结合着我们的可重入性质了。公平锁的含义我们上面已经说了，就是谁等的时间最长，谁就先获得锁。
```java
public class ReentrantLockTest {

  private static final Lock lock = new ReentrantLock(true);

  public static void main(String[] args) {
    new Thread(() -> test(), "线程A").start();
    new Thread(() -> test(), "线程B").start();
    new Thread(() -> test(), "线程C").start();
    new Thread(() -> test(), "线程D").start();
  }

  public static void test() {
    for (int i = 0; i < 2; i++) {
      try {
        lock.lock();
        System.out.println(Thread.currentThread().getName() + "获得了锁。");
        TimeUnit.SECONDS.sleep(2);
      } catch (InterruptedException e) {
        e.printStackTrace();
      } finally {
        System.out.println(Thread.currentThread().getName() + "释放了锁。");
        lock.unlock();
      }
    }
  }

}
```
首先 new 一个 ReentrantLock 的时候参数为 true，表明实现公平锁机制。这里多定义几个线程ABCD，然后在 test 方法中循环执行了两次加锁和解锁的过程。
```java
// 运行结果
线程A获得了锁。
线程A释放了锁。
线程B获得了锁。
线程B释放了锁。
线程C获得了锁。
线程C释放了锁。
线程D获得了锁。
线程D释放了锁。
线程A获得了锁。
线程A释放了锁。
线程B获得了锁。
线程B释放了锁。
线程C获得了锁。
线程C释放了锁。
线程D获得了锁。
线程D释放了锁。
```

### 3、非公平锁实现
非公平锁那就是随机获取，谁运气好，cpu 时间片段就轮询到哪个线程，这个线程就能获取锁。和上面的非公平锁区别非常简单，就是在 new 一个 ReentrantLock 的时候参数为 false，不写默认就是 flase。
```java
// 运行结果
线程A获得了锁。
线程A释放了锁。
线程A获得了锁。
线程A释放了锁。
线程B获得了锁。
线程B释放了锁。
线程B获得了锁。
线程B释放了锁。
线程C获得了锁。
线程C释放了锁。
线程C获得了锁。
线程C释放了锁。
线程D获得了锁。
线程D释放了锁。
线程D获得了锁。
线程D释放了锁。
```
整个过程都是随机的部分谁先谁后。

### 4、响应中断
响应中断就是一个线程获取不到锁，不会傻傻的一直等下去，ReentrantLock 会给与一个中断回应。在这个我们举一个死锁的案例。

首先定义两个锁 lock1 和 lock2。然后使用两个线程 thread 和 thread1 构造死锁场景。正常情况下，这两个线程互相等待获取资源而处于死循环状态。但是我们此时 thread 中断，另外一个线程就可以获取资源，正常执行了。
```java
public class ReentrantLockTest {

  static Lock lock1 = new ReentrantLock();
  static Lock lock2 = new ReentrantLock();

  public static void main(String[] args) {
    Thread thread = new Thread(new ThreadDemo(lock1, lock2));
    Thread thread1 = new Thread(new ThreadDemo(lock1, lock2));
    thread.start();
    thread1.start();
    // 这是第一个线程中断
    thread.interrupt();
  }

  static class ThreadDemo implements Runnable {

    Lock firstLock;
    Lock secondLock;

    public ThreadDemo(Lock firstLock, Lock secontLock) {
      this.firstLock = firstLock;
      this.secondLock = secontLock;
    }

    @Override
    public void run() {
      try {
        firstLock.lockInterruptibly();
        TimeUnit.MILLISECONDS.sleep(50);
        secondLock.lockInterruptibly();
      } catch (Exception e) {
        e.printStackTrace();
      } finally {
        firstLock.unlock();
        secondLock.unlock();
        System.out.println(Thread.currentThread().getName() + "获取到了资源，正常结束！");
      }
    }
  }

}
```
运行测试结果
```java
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at java.lang.Thread.sleep(Thread.java:340)
	at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
	at com.example.demo.ReentrantLockTest$ThreadDemo.run(ReentrantLockTest.java:39)
	at java.lang.Thread.run(Thread.java:748)
Exception in thread "Thread-0" java.lang.IllegalMonitorStateException
	at java.util.concurrent.locks.ReentrantLock$Sync.tryRelease(ReentrantLock.java:151)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.release(AbstractQueuedSynchronizer.java:1261)
	at java.util.concurrent.locks.ReentrantLock.unlock(ReentrantLock.java:457)
	at com.example.demo.ReentrantLockTest$ThreadDemo.run(ReentrantLockTest.java:45)
	at java.lang.Thread.run(Thread.java:748)
Thread-1获取到了资源，正常结束！
```

### 5、限时等待
限时等待可以通过 tryLock 方法来实现，可以选择传入时间参数，表示等待指定的时间，无参则表示立即返回锁申请的结果：true 表示获取锁成功，false 表示获取锁失败。我们可以通过这种方法来解决死锁问题。

```java
public class ReentrantLockTest {

  static Lock lock1 = new ReentrantLock();
  static Lock lock2 = new ReentrantLock();

  public static void main(String[] args) {
    Thread thread = new Thread(new ThreadDemo(lock1, lock2));
    Thread thread1 = new Thread(new ThreadDemo(lock1, lock2));
    thread.start();
    thread1.start();
  }

  static class ThreadDemo implements Runnable {

    Lock firstLock;
    Lock secondLock;

    public ThreadDemo(Lock firstLock, Lock secontLock) {
      this.firstLock = firstLock;
      this.secondLock = secontLock;
    }

    @Override
    public void run() {
      try {
        if (!firstLock.tryLock()) {
          TimeUnit.MICROSECONDS.sleep(10);
        }
        if (!secondLock.tryLock()) {
          TimeUnit.MICROSECONDS.sleep(10);
        }
      } catch (Exception e) {
        e.printStackTrace();
      } finally {
        firstLock.unlock();
        secondLock.unlock();
        System.out.println(Thread.currentThread().getName() + "获取到了资源，正常结束！");
      }
    }
  }

}
```
这个案例中，一个线程获取 firstLock 时候第一次失败，那就等 10 毫秒后第二次获取，就这样一直不停的调试，一直等到获取相应的资源为止

当然，我们可以设置tryLock的超时等待时间tryLock(long timeout,TimeUnit unit)，也就是说一个线程在指定的时间内没有获取锁，那就会返回false，就可以再去做其他事了。
