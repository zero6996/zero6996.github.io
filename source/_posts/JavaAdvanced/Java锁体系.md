---
title: Java锁体系概述
date: 2020-7-08 18:00
categories: JavaAdvanced
tags: [Java Lock]
urlname: Java-Lock
---



# Java锁体系

Java提供了种类丰富的锁，每种锁因其特性的不同，在适当的场景下能够展现出非常高的效率。本文旨在对锁相关源码、使用场景进行举例，以介绍主流锁的知识点及其适用场景。

1. 乐观锁，悲观锁
2. 读锁（共享锁），写锁（排他锁）
3. 自旋锁，非自旋锁
4. 无锁，偏向锁，轻量级锁，重量级锁
5. 分布式锁
6. 区间锁（分段锁）`java.util.concurrent` `ConcurrentHashMap`
7. 重入锁，非重入锁
8. 公平锁，非公平锁

![Java锁体系](https://s1.ax1x.com/2020/06/28/N2aGzq.png)

## 1. 悲观锁与乐观锁

悲观锁与乐观锁是一种广义概念，体现的是看待线程同步的不同角度。

### 1.1 悲观锁

悲观锁认为自己在使用数据时一定有别的线程会来修改数据，因此在获取数据时就会先加锁，确保数据不会被别的线程修改。

- 锁实现：`synchronized`，`Lock`接口的相关实现类
- 适用场景：写操作较多的场景，先加锁以保证写操作时数据的正确性

### 1.2 乐观锁

乐观锁认为自己在使用数据时不会有别的线程来修改数据，所以不会加锁，但是在更新数据时会去判断之前有没有别的线程更新了这个数据； 如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。 

- 锁实现：Java中通过使用无锁编程实现，常采用CAS算法，例如`AtomicInteger`类的原子自增就是通过CAS自旋实现的
- 适用场景：读操作较多的场景，不加锁的特点能使读操作的性能大幅提升

### 1.3 悲观锁与乐观锁的执行过程

[见流程图](https://www.processon.com/view/link/5ef8563ff346fb1ae5810e05)

### 1.4 CAS算法

为什么乐观锁能够做到不锁定资源情况下也正确的实现线程同步呢？因为乐观锁使用到了CAS技术。

CAS全称`Compare And Swap`（比较与交换），它是一种常见的降低读写锁冲突，保证数据一致性的乐观锁机制。 CAS是一种无锁算法，在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。

#### 1.4.1 CAS算法原理

在JUC包中的原子类`AtomicInteger`就是通过CAS来实现了乐观锁，该算法会涉及到三个操作数：

1. 需要读写的内存值：V
2. 进行比较的值：A
3. 要写入的新值：B

当且仅当V值等于A值时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作，一般“更新”操作会不断重试。

CAS[算法执行原理](https://www.processon.com/view/link/5ef8696b07912929cb61a06d)

#### 1.4.2 CAS存在的问题

- 循环时间长开销大：CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来较大的开销。
- 只能保证一个共享变量的原子操作：对一个共享变量执行操作时，CAS能保证原子操作，但对多个共享变量操作时，CAS是无法保证操作的原子性的。
- JDK1.5开始提供了`AtomicReference`类来保证引用对象之间的原子性，可以把多个变量放在一个对象里进行CAS操作。
- 高并发场景下很容易发生的ABA问题

假设两个线程T1和T2访问同一个变量V，当T1访问变量V时，读取到V的值为A；此时线程T1被抢占了，T2开始执行，T2会先将变量V的值从A改成B，然后又将变量V从B改回了A；此时T1得到时间片，继续执行，它发现变量V的值还是A，认为没有发生变化，就会继续执行。这个过程中，主内存中的变量V变化就是`A->B-A`，故称ABA问题。

明面上看不会导致什么问题，但实际上的问题在于：“值是一样的”等同于“没有发生变化”这个认知；就算V值被改回去了，那么现在的V也不是以前的V了。因此有的时候，我们并不只是需要判断变量是否相同，还需要在执行过程中，判断这个变量是否已经发生了变化。

- 如何解决ABA问题

对于ABA问题带来的隐患，各种乐观锁的实现通常会使用版本戳`version`来核对记录或者对象标记，避免并发操作带来的问题。Java中的`AtomicStampedReference`实现了这个作用， 它通过包装`[E,Integer]`的元组来对对象**标记版本戳**`stamp`，从而避免ABA问题；代码实例如下：

```java
public class ABA {
    private static AtomicInteger atomicInt = new AtomicInteger(100);
    private static AtomicStampedReference atomicStampedRef =  new AtomicStampedReference(100,0);

    public static void main(String[] args) throws Exception{
        Thread intT1 = new Thread(new Runnable() {
            @Override
            public void run() {
                // 复现ABA问题
                atomicInt.compareAndSet(100, 101);
                atomicInt.compareAndSet(101, 100);
            }
        });
        Thread intT2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                }catch (Exception e){
                    e.printStackTrace();
                }
                // 会成功执行CAS操作
                boolean c3 = atomicInt.compareAndSet(100,101);
                System.out.println(c3); // true
            }
        });
        intT1.start();
        intT2.start();
        intT1.join();
        intT2.join();

        Thread refT1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                // 使用AtomicStampedReference解决ABA问题
                atomicStampedRef.compareAndSet(100,101,atomicStampedRef.getStamp(),atomicStampedRef.getStamp()+1);
                atomicStampedRef.compareAndSet(101,100,atomicStampedRef.getStamp(),atomicStampedRef.getStamp()+1);

            }
        });

        Thread refT2 = new Thread(new Runnable() {
            @Override
            public void run() {
                int stamp = atomicStampedRef.getStamp();
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                // 执行CAS操作会失败
                boolean c3 = atomicStampedRef.compareAndSet(100,101,stamp,stamp+1);
                System.out.println(c3); // false
            }
        });

        refT1.start();
        refT2.start();
    }
}
```

## 2. 自旋锁与适应性自旋锁

### 2.1 自旋锁

自旋锁是指当一个线程在获取锁时，如果锁已经被其他线程获取，那么该线程会一直循环判断锁是否被释放，直到获取到锁才会退出循环。

- 自旋锁的原理

阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间，开销较大；如果同步代码块中的内容过于简单，状态切换消耗的时间可能比用户代码执行时间还要长，为了这一小段时间区切换CPU状态，线程挂起和恢复现场的花费可能让系统得不偿失。

如果持有锁的线程能在短时间内释放资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞状态了，它们只需要等一等（自旋），等到持有锁的线程释放锁后即可获取，这样就避免了用户进程和内核切换的消耗。

![Spin-Lock](https://s1.ax1x.com/2020/07/02/NbI9PI.png)



### 2.2 适应性自旋锁

自旋锁本身是有缺点的，它并不能代替阻塞。自旋等待虽然避免了线程切换的开销，但是它会占用处理器时间；如果锁被占用时间很短，自旋等待的效果就会非常好，反之如果锁被占用时间很长，那么自旋的线程只会白浪费处理器资源。所以自旋等待的时间必须要有一定的限度，如果超过了限定次数没有成功获得锁，就挂起线程。

JDK1.6中就引入了**适应性自旋锁**，自适应意味着自旋的时间（次数）不再固定，而是由前一次在同锁上的自旋时间及锁拥有者的状态来决定；如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很可能成功，进而允许它拥有较长时间的自旋等待，反之则会直接阻塞线程。

可以通过`-XX:-UseSpining`参数来设置是否使用自旋锁，通过`-XX:PreBlockSpin`参数设置自旋次数（默认10次）。JDK1.7以后取消了用户对自旋参数的配置，均由虚拟机自动调整。



### 2.3 小结：自旋锁的优缺点

- 优点：自旋锁减少了线程的阻塞，对于**锁竞争不激烈且占用锁时间非常短**的代码来说性能能大幅提升，因为避免了线程的上下文切换。
- 缺点：如果锁竞争激烈且线程长时间占有锁，就不适合使用自旋锁了；因为自旋锁会一直占用CPU资源，如果长时间自旋无法获取锁，线程自旋的消耗大于线程阻塞挂起的消耗，就需要关闭自旋锁。

## 3. 无锁、偏向锁、轻量级锁、重量级锁

这四种是指锁的状态，专门针对synchronized的。先了解一些额外知识。

### 3.1 为什么Synchronized能实现线程同步？

首先要知道两个重要概念：Java对象头、Monitor

#### 3.1.1 Java对象头

synchronized是悲观锁，在操作同步资源之前需给同步资源加锁，这把锁就是存在Java对象头里的。

以Hotspot虚拟机为例，Hotspot虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针），如下就是一个对象的存储结构图

![对象头](https://s1.ax1x.com/2020/07/03/NjkK1A.png)

- **Mark Word**：默认存储对象的HashCode，分代年龄和锁标志位信息等。

这些信息都是与对象自身定义无关的数据，所以Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。它会根据对象的状态复用自己的存储空间，也就是说在运行期间Mark Word里存储的数据会随着锁标志位的变化而变化。

- **Klass Point**：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

> 面试题：Java中对象是如何存储的？
>
> 答：对象的实例数据存储在堆空间，对象的元数据存在方法区（元空间），对象的引用存在栈空间。

#### 3.1.2 Monitor

Monitor可以理解为一个同步工具或一种同步机制，通常被描述为一个对象。每个Java对象就有一把看不见的锁，称之为内部锁或者Monitor锁。

Monitor是线程私有的数据结构，每个线程都有一个可用monitor record列表，同时还有一个全局可用列表；每个被锁住的对象都会和一个monitor关联，同时monitor中有一个owner（所有者）字段存放拥有该锁线程的唯一标识，表示该锁被这个线程占用。

synchronized锁是JVM内置锁，通过内部对象Monitor（监视器锁）实现线程同步，监视器锁的实现依赖底层操作系统的`Mutex Lock`（互斥锁）来实现线程同步。

每个同步对象都有一个自己的Monitor：

![Monitor](https://s1.ax1x.com/2020/07/01/N7WMwt.png)

### 3.2 synchronized的锁升级

synchronized最初实现同步的方式就是阻塞和唤醒来实现线程同步，这就是JDK 6之前synchronized效率低的原因，这种**依赖于操作系统Mutex Lock所实现的锁称之为“重量级锁”**，JDK 6中为了减少获取锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。

因此目前锁一共有4种状态，级别从低到高分别是：无锁、偏向锁、轻量级锁、重量级锁，锁状态只能升级不能降级。四种锁状态对应的Mark Word内容如下：

| 锁状态   | 存储内容                                           | 标志位 |
| -------- | -------------------------------------------------- | :----: |
| 无锁     | 对象的HashCode、对象分代年龄、是否是偏向锁         |   01   |
| 偏向锁   | 偏向线程ID、偏向时间戳、对象分代年龄、是否是偏向锁 |   01   |
| 轻量级锁 | 指向栈中锁记录的指针                               |   00   |
| 重量级锁 | 指向互斥量的指针                                   |   10   |

#### 3.2.1 无锁

无锁没有对资源进行锁定，所有线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。

无锁的特点就是修改操作在循环内进行，线程会不断的尝试修改共享资源。如果没有冲突就修改成功并退出，否则继续循环尝试。如果多个线程修改同一值，必定会有一个线程修改成功，其他修改失败的线程会不断重试直到修改成功。

如上面讲到的CAS原理及其应用就是无锁的实现，无锁无法全面代替锁，但在某些特定场合非常适合，能提高性能。

#### 3.2.2 偏向锁

偏向锁是指一段同步代码一直被一个线程访问，那么该线程会自动获取锁，降低获取锁的代价。

大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获取，为了让线程获取锁的代价更低而引入偏向锁。当一个线程访问同步代码块并获取锁时，会在Mark Word里记录偏向的线程ID。在线程进入和退出同步块时不再通过CAS操作来加锁和解锁，只需简单地测试一下Mark Word里是否存储着指向当前线程的偏向锁。

如果测试成功，表示线程已经获得了锁。

如果测试失败，则需要再测试一下Mark Word中偏向锁的标识是否设置为1（表示指向当前进程），如果没有则用CAS竞争锁；如果设置了，则尝试使用CAS将对象头的偏向锁指向当前进程。

偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动释放偏向锁。**偏向锁的撤销**：需要等待全局安全点（指在这个时间点上没有字节码正在执行），首先它会暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态，撤销偏向锁后恢复到无锁（标志位01）或者轻量级锁（标志位00）的状态。

偏向锁在JDK6以后的JVM里是默认启用的，可以通过JVM参数关闭偏向锁：`-XX:-UseBiasedLocking=false`，关闭之后程序默认会进入轻量级锁状态。

#### 3.2.3 轻量级锁

轻量级锁是由偏向锁升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁竞争时 ，偏向锁就会升级为轻量级锁，其他线程会通过**自旋**的形式尝试获取锁，不会阻塞，从而提高性能。

- 轻量级锁的加锁过程
  1. 在代码块进入同步块时，如果同步对象锁状态为无锁状态（锁标志位01，是否偏向锁0），虚拟机首先将在当前线程的**栈帧中建立一个名为锁记录**（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，官方称之为Displaced Mark Word。
  2. 拷贝对象头中的Mark Word复制到锁记录中。
  3. 拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock Record里的owner指针指向对象的Mark Word。如果更新成功，则执行步骤4，否则执行步骤5
  4. 如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word的锁标志位设置为00，表示此对象处于轻量级锁定状态。
  5. 如果这个更新操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是说明当前线程已经拥有了这个对象的锁，那么可用直接进入同步块继续执行。否则说明有多个线程竞争锁，若当前只有一个等待线程，则该线程通过自旋进行等待；但当自旋超过一定次数或者一个线程持有锁，一个在自旋，又有第三个线程来访问，轻量级锁就会膨胀升级为重量级锁，锁标志位变为10。
- [加锁过程图解见](https://www.processon.com/view/link/5effeb67e0b34d4dba69a2ca)

#### 3.2.4 重量级锁

 升级为重量级锁时，锁标志的状态值变为“10”，此时Mark Word中存储的是指向重量级锁的指针，此时等待锁的线程都会进入阻塞状态。 

重量级锁是依赖对象内部的monitor锁来实现的，而monitor又依赖操作系统的MutexLock(互斥锁)来实现的，所以重量级锁也被成为互斥锁。 

#### 3.2.5 小结：锁的优缺点对比

| 锁       |                             优点                             |                       缺点                        |                 适用场景                 |
| :------- | :----------------------------------------------------------: | :-----------------------------------------------: | :--------------------------------------: |
| 偏向锁   | 加锁和解锁无需额外的消耗，和执行非同步方法相比仅存在纳秒级的差距 |   如果线程间存在锁竞争，会带来额外锁撤销的消耗    |        适用于单线程访问同步块场景        |
| 轻量级锁 |           竞争的线程不会阻塞，提高了程序的响应速度           | 如果竞争的线程始终得不到锁，自旋操作会消耗CPU资源 | 追求响应速度，同步块执行速度非常快的场景 |
| 重量级锁 |             线程竞争不使用自旋，不会消耗CPU资源              |              线程阻塞，响应时间缓慢               |      追求吞吐量，同步块执行耗时较长      |

> Java中的显式锁和隐式锁的概念
>
> - 显式锁：例如`ReentrantLock`，整个加锁跟解锁的过程需要我们手动编写代码去控制
> - 隐式锁：例如`synchronized`，加锁与解锁的过程都不需要我们在代码中人为控制，JVM会自动加锁和解锁。

## 4. 公平锁与非公平锁

- 公平锁：公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。
  - 优点：公平锁的优点是等待的线程不会饿死
  - 缺点：缺点是整体吞吐率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。

- 非公平锁：非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取到锁的场景。
  - 优点：非公平锁的优点是可以减少唤起线程的开销，整体吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程。
  - 缺点：缺点是处于等待队列中的线程可能会饿死，或者等很久才会获的锁。

> 线程饥饿：是指线程因无法访问到所需资源而无法执行下去的情况，在CPU繁忙的情况下，优先级低的线程得到执行的机会很小，或者持有锁的线程长时间执行都会可能导致“饥饿”问题。

### 4.1 图示举例

![fairLock](https://s1.ax1x.com/2020/07/07/UF2S1S.png)

如上图所示，假设有一口水井， 有管理员看守，管理员有一把锁，只有拿到锁的人才能够打水，打完水要把锁还给管理员。每个过来打水的人都要管理员的允许并拿到锁之后才能去打水，如果前面有人正在打水，那么这个想要打水的人就必须排队。管理员会查看下一个要去打水的人是不是队伍里排最前面的人，如果是的话，才会给你锁让你去打水；如果你不是排第一的人，就必须去队尾排队，这就是公平锁。 

但是对于非公平锁，管理员对打水的人没有要求。即使等待队伍里有排队等待的人，但如果在上一个人刚打完水把锁还给管理员而且管理员还没有允许等待队伍里下一个人去打水时，刚好来了一个插队的人，这个插队的人是可以直接从管理员那里拿到锁去打水，不需要排队，原本排队等待的人只能继续等待。如下图所示： 

![NonFairLock](https://s1.ax1x.com/2020/07/07/UF2ncF.png)

### 4.2 源码解读

下面通过`ReentrantLock`源码来理解公平锁和非公平锁：

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;
    abstract static class Sync extends AbstractQueuedSynchronizer {...}
    static final class NonfairSync extends Sync {...}
    static final class FairSync extends Sync {...}
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
}
```

`ReentrantLock`里有一个内部类Sync，Sync继承自AQS（`AbstractQueueSynchronizer`，抽象排队同步器），添加锁和释放锁的大部分操作实际上都是在Sync中实现的。它有公平锁`FairSync`和非公平锁`NonfairLock`两个子类，`ReentrantLock`默认使用非公平锁，也可通过构造器显式的指定使用公平锁。

下面是公平锁与非公平锁的源码解析：

```java
// 公平锁核心源码
protected final boolean tryAcquire(int acquires) {
    // 获取当前线程对象
    final Thread current = Thread.currentThread();
    // 获取当前显式锁加锁次数
    int c = getState();
    // 如果当前没有锁
    if (c == 0) {
        // 如果队列前面没有排队的线程并且能成功设置同步状态
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            // 设置当前线程为拥有独占锁的线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 反之，如果当前线程就是拥有独占锁的线程
    else if (current == getExclusiveOwnerThread()) {
        // 计算下一次加锁后的锁次数（ReentrantLock是可重入锁，一个线程可以多次加锁）
        int nextc = c + acquires;
        // 如果锁次数小于0，报“超过最大锁数”错误
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        // 反之设置加锁次数
        setState(nextc);
        return true;
    }
    // 尝试获取锁失败，返回false
    return false;
}

// 非公平锁核心源码
final boolean nonfairTryAcquire(int acquires) {
    // 获取当前线程
    final Thread current = Thread.currentThread();
    // 获取加锁次数
    int c = getState();
    // 如果没加锁
    if (c == 0) {
        // cas设置同步状态
        if (compareAndSetState(0, acquires)) {
            // 设置当前线程拥有独占锁
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 反之如果当前线程就是拥有独占锁的线程
    else if (current == getExclusiveOwnerThread()) {
        // 源码同公平锁解读
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

由源码可见， 公平锁与非公平锁的`lock()`方法唯一的区别就在于公平锁在获取同步状态时多了一个限制条件：`hasQueuedPredecessors()`

 ```java
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    // 主要判断当前线程是否位于同步队列中的第一个，是则返回true，反之false
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
 ```

公平锁就是通过同步队列来实现多个线程按照申请锁的顺序来获取锁，从而实现公平的特性。非公平锁加锁时不考虑排队等待问题，直接尝试获取锁，所以存在后申请却先获得锁的情况。 

## 5. 可重入锁和非可重入锁

可重入锁又名递归锁，是指同一个线程在外层方法获取锁时，再次进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。Java中`ReentrantLock`和`synchronized`都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。 下面看示例代码分析：

```java
public class Widget {
    public synchronized void doSomething() {
        System.out.println("方法1执行...");
        doOthers();
    }

    public synchronized void doOthers() {
        System.out.println("方法2执行...");
    }
}
```

上述代码中，类中的两个方法都是被内置锁synchronized修饰的，`doSomething()`方法中会调用`doOthers()`方法。因为内置锁是可重入的，所以同一个线程在调用`doOthers()`时可以直接获得当前对象的锁，进入`doOthers`代码块的操作。

如果是一个不可重入锁，那么当前线程在调用`doOthers()`之前需要将执行`doSomething()`时获取的当前对象的锁释放掉，实际上该对象锁已被当前线程所持有，且无法释放。所以此时会出现死锁。 

### 5.1 为什么可重入锁在嵌套时可自动获取锁？

下面通过 图示和源码来分析：

打水的例子，有多个人在排队打水，此时管理员允许锁和同一个人的多个水桶绑定。这个人用多个水桶打水时，第一个水桶和锁绑定并打完水之后，第二个水桶也可以直接和锁绑定并开始打水，所有的水桶都打完水之后打水人才会将锁还给管理员。这个人的所有打水流程都能够成功执行，后续等待的人也能够打到水。这就是可重入锁。 

![reentrant](https://s1.ax1x.com/2020/07/07/UAKVbR.png)

但如果是非可重入锁的话，此时管理员只允许锁和同一个人的一个水桶绑定。第一个水桶和锁绑定打完水之后并不会释放锁，导致第二个水桶不能和锁绑定也无法打水。当前线程出现死锁，整个等待队列中的所有线程都无法被唤醒。 

![Nonreentrant](https://s1.ax1x.com/2020/07/07/UAKGqA.png)

### 5.2 源码分析

ReentrantLock和synchronized都是重入锁，那么通过 重入锁`ReentrantLock`以及非可重入锁`NonReentrantLock`的源码来对比分析一下为什么非可重入锁在重复调用同步资源时会出现死锁。 源码如下：

```java
// 可重入锁
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 获取锁时先判断，如果当前线程是已经占用锁的线程，则status+1，并返回true
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
/** 可重入锁-释放锁
	释放锁时，也会先判断当前线程是否是已占有锁的线程，然后再判断status
	如果status等于0，才会真正释放锁。
**/
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    // 如果当前线程不是独占锁的线程，报 非法的监视器状态异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 只有status等于0时才说明该线程所有的加锁都释放了
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
// 非可重入锁：Java中的非可重入锁有哪些？
```

## 6. 独享锁与共享锁

 独享锁和共享锁同样是一种概念 ，先介绍概念在看源码解读。

-  独享锁：也叫排他锁 ，只指该锁一次只能被一个线程所持有。如果线程T对数据A加上排他锁后，则其他线程不能再对A加任何类型的锁，只有获得排他锁的线程能读写数据。JDK中的`synchronized`和JUC中的Lock相关实现类都是互斥锁。
- 共享锁：是指该线程可以被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排他锁。获得共享锁的线程只能读数据，不能修改数据。

独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或共享。

以下为`ReentrantReadWriteLock`部分源码：

```java
public class ReentrantReadWriteLock
    	implements ReadWriteLock, java.io.Serializable {
    private static final long serialVersionUID = -6992448646407690164L;
    /** Inner class providing readlock */
    private final ReentrantReadWriteLock.ReadLock readerLock;
    /** Inner class providing writelock */
    private final ReentrantReadWriteLock.WriteLock writerLock;
    /** Performs all synchronization mechanics */
    final Sync sync;
    // 默认创建非公平锁
    public ReentrantReadWriteLock() {
        this(false);
    }
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
    
    public static class ReadLock implements Lock, java.io.Serializable {
        private static final long serialVersionUID = -5992448646407690164L;
        private final Sync sync;
        protected ReadLock(ReentrantReadWriteLock lock) {
            sync = lock.sync;
        }
    }
    
    public static class WriteLock implements Lock, java.io.Serializable {
        private static final long serialVersionUID = -4992448646407690164L;
        private final Sync sync;
        protected WriteLock(ReentrantReadWriteLock lock) {
            sync = lock.sync;
        }
    }
}
```

源码可知`ReentrantReadWriteLock`有两把锁，`ReadLock`和`WriteLock`，一个读锁和一个写锁，合称读写锁。读写锁都是靠内部类Sync实现的锁，Sync是AQS的子类，这种结构在`CountDownLatch`、`ReentrantLock`、`Semaphore`里面也都存在。 

在`ReentrantReadWriteLock`里面，读锁和写锁的锁主体都是Sync，但读锁和写锁的加锁方式不一样。读锁是共享锁，写锁是独享锁。读锁的共享锁可保证并发读非常高效，而读写、写读、写写的过程是互斥的，因为读锁和写锁是分离的。所以`ReentrantReadWriteLock`的并发性相比一般的互斥锁有了很大提升。 

### 6.1 读写锁源码分析

AQS中有一个`state`参数，该字段表示有多少线程持有锁。在独享锁中这个值通常是0或者1（重入锁中state值表示重入次数），在共享锁中state就是持有锁的数量。但是在`ReentrantReadWriteLock`中有读、写两把锁，所以需要在一个整型变量state上分别描述读锁和写锁的数量；于是将state变量**“按位切割”**，切分成了两个部分，高16位表示读锁状态（读锁个数），低16位表示写锁状态（写锁个数），如下图示：

![state](https://s1.ax1x.com/2020/07/08/UVBe8U.png)

接下来查看写锁的加锁源码：

```java
protected final boolean tryAcquire(int acquires) {
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取当前锁的个数
    int c = getState();
    // 获取写锁的个数
    int w = exclusiveCount(c);
    // 如果已经有线程持有了锁(c!=0)
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        // 如果写线程数(w)为0(换言之存在读锁) 或者持有锁的线程不是当前线程就返回失败
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT) //如果写入锁数量大于最大数(65535 2的16次方-1 ) 就抛出一个Error
            throw new Error("Maximum lock count exceeded");
        // 增加重入状态数
        setState(c + acquires);
        return true;
    }
    // 如果当前线程需要阻塞那么就返回失败 或者 CAS设置同步状态失败也返回失败
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    // 如果c=0,w=0或c>0，w>0(重入),则设置当前线程为锁拥有者
    setExclusiveOwnerThread(current);
    return true;
}
```

- 写锁源码解读：
  - 首先获取当前锁的个数c，然后通过c获取写锁的个数；因为写锁是低16位，所以取低16位的最大值与当前的c做与运算（`return c & EXCLUSIVE_MASK`），高16位和0与运算后是0，剩下的就是低位运算的值，同时也是持有写锁的线程数目；
  - 在取到写锁线程数量后，首先判断是否有线程持有了锁。如果已经有线程持有了锁（c!=0），则查看当前写锁线程数量，如果写锁线程数量为0（即此时存在读锁，非写即读）或者持有锁的线程不是当前线程就返回失败（涉及到公平锁和非公平锁的实现）；
  - 如果写入锁数量大于最大数，就抛出一个Error；
  - 如果当前写线程数为0（那么读线程也应该为0，因为上面已经处理c!=0的情况），并且当前线程需要阻塞那么就返回失败；如果通过CAS增加写线程数失败也返回失败；
  -  如果c=0,w=0或者c>0,w>0（重入），则设置当前线程或锁的拥有者，返回成功！ 

`tryAcquire()`除了重入条件（当前线程为获取了写锁的线程）之外，增加了一个读锁是否存在的判断。如果存在读锁，则写锁不能被获取，原因在于：必须确保写锁的操作对读锁可见，如果允许读锁在已被获取的情况下对写锁的获取，那么正在运行的其他读线程就无法感知到当前写线程的操作。 

因此，只有等待其他读线程都释放了读锁，写锁才能被当前线程获取，而写锁一旦被获取，则其他读写线程的后续访问均被阻塞。写锁的释放与ReentrantLock的释放过程基本类似，每次释放均减少写状态，当写状态为0时表示写锁已被释放，然后等待的读写线程才能够继续访问读写锁，同时前次写线程的修改对后续的读写线程可见。 

下面是读锁的源码：

```java
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;  // 如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

可以看到在`tryAcquireShared(int unused)`方法中，如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态。如果当前线程获取了写锁或者写锁未被获取，则当前线程（线程安全，依靠CAS保证）增加读状态，成功获取读锁。读锁的每次释放（线程安全的，可能有多个读线程同时释放读锁）均减少读状态，减少的值是“1<<16”。所以读写锁才能实现读读的过程共享，而读写、写读、写写的过程互斥。 

此时，再回头看看互斥锁`ReentrantLock`中公平锁和非公平锁的加锁源码 :

![lock](https://s1.ax1x.com/2020/07/08/UVqDwn.png)

`ReentrantLock`虽然有公平锁和非公平锁两种，但是它们添加的都是独享锁。根据源码所示，当某一个线程调用lock方法获取锁时，如果同步资源没有被其他线程锁住，那么当前线程在使用CAS更新state成功后就会成功抢占该资源。而如果公共资源被占用且不是被当前线程占用，那么就会加锁失败。所以可以确定ReentrantLock无论读操作还是写操作，添加的锁都是都是独享锁。 

## 结语

本文主要内容均来自美团技术团队-[不可不说的Java"锁"事](https://tech.meituan.com/2018/11/15/java-lock.html)一文，仅做了部分个人理解上的修改，在最后读写锁源码分析这块内容尚未完全理解透彻，在此记录。