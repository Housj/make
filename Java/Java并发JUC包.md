# AQS实现

jdk的JUC包(java.util.concurrent)提供大量Java并发工具提供使用，基本由Doug Lea编写，很多地方值得学习和借鉴，是进阶升级必经之路

本文从JUC包中常用的对象锁、并发工具的使用和功能特性入手，带着问题，由浅到深，一步步剖析并发底层AQS抽象类具体实现

## 名词解释

### 1 AQS

AQS是一个抽象类，类全路径java.util.concurrent.locks.AbstractQueuedSynchronizer，抽象队列同步器，是基于模板模式开发的并发工具抽象类，有如下并发类基于AQS实现：

![img](https:////upload-images.jianshu.io/upload_images/5618238-885e190f5a4ac242.png?imageMogr2/auto-orient/strip|imageView2/2/w/967/format/webp)

### 2 CAS

CAS是Conmpare And Swap(比较和交换)的缩写，是一个原子操作指令

CAS机制当中使用了3个基本操作数：内存地址addr，预期旧的值oldVal，要修改的新值newVal
 更新一个变量的时候，只有当变量的预期值oldVal和内存地址addr当中的实际值相同时，才会将内存地址addr对应的值修改为newVal

基于乐观锁的思路，通过CAS再不断尝试和比较，可以对变量值线程安全地更新

### 3 线程中断

线程中断是一种线程协作机制，用于协作其他线程中断任务的执行

当线程处于阻塞等待状态，例如调用了wait()、join()、sleep()方法之后，调用线程的interrupt()方法之后，线程会马上退出阻塞并收到InterruptedException；

当线程处于运行状态，调用线程的interrupt()方法之后，线程并不会马上中断执行，需要在线程的具体任务执行逻辑中通过调用isInterrupted() 方法检测线程中断标志位，然后主动响应中断，通常是抛出InterruptedException

## 对象锁特性

下面先介绍对象锁、并发工具有哪些基本特性，后面再逐步展开这些特性如何实现

### 1 显式获取

以ReentrantLock锁为例，主要支持以下4种方式显式获取锁

- **(1) 阻塞等待获取**



```csharp
ReentrantLock lock = new ReentrantLock();
// 一直阻塞等待，直到获取成功
lock.lock();
```

- **(2) 无阻塞尝试获取**



```csharp
ReentrantLock lock = new ReentrantLock();
// 尝试获取锁，如果锁已被其他线程占用，则不阻塞等待直接返回false
// 返回true - 锁是空闲的且被本线程获取，或者已经被本线程持有
// 返回false - 获取锁失败
boolean isGetLock = lock.tryLock();
```

- **(3) 指定时间内阻塞等待获取**



```csharp
ReentrantLock lock = new ReentrantLock();
try {
    // 尝试在指定时间内获取锁
    // 返回true - 锁是空闲的且被本线程获取，或者已经被本线程持有
    // 返回false - 指定时间内未获取到锁
    lock.tryLock(10, TimeUnit.SECONDS);
} catch (InterruptedException e) {
    // 内部调用isInterrupted() 方法检测线程中断标志位，主动响应中断
    e.printStackTrace();
}
```

- **(4) 响应中断获取**



```csharp
ReentrantLock lock = new ReentrantLock();
try {
    // 响应中断获取锁
    // 如果调用线程的thread.interrupt()方法设置线程中断，线程退出阻塞等待并抛出中断异常
    lock.lockInterruptibly();
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

### 2 显式释放



```csharp
ReentrantLock lock = new ReentrantLock();
lock.lock();
// ... 各种业务操作
// 显式释放锁
lock.unlock();
```

### 3 可重入

已经获取到锁的线程，再次请求该锁可以直接获得

### 4 可共享

指同一个资源允许多个线程共享，例如读写锁的读锁允许多个线程共享，共享锁可以让多个线程并发安全地访问数据，提高程序执效率

### 5 公平、非公平

公平锁：多个线程采用先到先得的公平方式竞争锁。每次加锁前都会检查等待队列里面有没有线程排队，没有才会尝试获取锁。
 非公平锁：当一个线程采用非公平的方式获取锁时，该线程会首先去尝试获取锁而不是等待。如果没有获取成功，才会进入等待队列

因为非公平锁方式可以使后来的线程有一定几率直接获取锁，减少了线程挂起等待的几率，性能优于公平锁

## AQS实现原理

### 1 基本概念

#### (1) Condition接口

类似Object的wait()、wait(long timeout)、notify()以及notifyAll()的方法结合synchronized内置锁可以实现可以实现等待/通知模式，实现Lock接口的ReentrantLock、ReentrantReadWriteLock等对象锁也有类似功能：

Condition接口定义了await()、awaitNanos(long)、signal()、signalAll()等方法，配合对象锁实例实现等待/通知功能，原理是基于AQS内部类ConditionObject实现Condition接口，线程await后阻塞并进入CLH队列(下面提到)，等待其他线程调用signal方法后被唤醒

#### (2) CLH队列

CLH队列，CLH是算法提出者Craig, Landin, Hagersten的名字简称

AQS内部维护着一个双向FIFO的CLH队列，AQS依赖它来管理等待中的线程，如果线程获取同步竞争资源失败时，会将线程阻塞，并加入到CLH同步队列；当竞争资源空闲时，基于CLH队列阻塞线程并分配资源

CLH的head节点保存当前占用资源的线程，或者是没有线程信息，其他节点保存排队线程信息

![img](https:////upload-images.jianshu.io/upload_images/5618238-8b912290b8532302.png?imageMogr2/auto-orient/strip|imageView2/2/w/448/format/webp)

CLH

CLH中每一个节点的状态(waitStatus)取值如下：

- **CANCELLED(1)**：表示当前节点已取消调度。当timeout或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的节点将不会再变化
- **SIGNAL(-1)**：表示后继节点在等待当前节点唤醒。后继节点入队后进入休眠状态之前，会将前驱节点的状态更新为SIGNAL
- **CONDITION(-2)**：表示节点等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的节点将从等待队列转移到同步队列中，等待获取同步锁
- **PROPAGATE(-3)**：共享模式下，前驱节点不仅会唤醒其后继节点，同时也可能会唤醒后继的后继节点
- **0**：新节点入队时的默认状态

#### (3) 资源共享方式

AQS定义两种资源共享方式：
 Exclusive 独占，只有一个线程能执行，如ReentrantLock
 Share 共享，多个线程可同时执行，如Semaphore/CountDownLatch

#### (4) 阻塞/唤醒线程的方式

AQS 基于sun.misc.Unsafe类提供的park方法阻塞线程，unpark方法唤醒线程，被park方法阻塞的线程能响应interrupt()中断请求退出阻塞

### 2 基本设计

核心设计思路：AQS提供一个框架，用于实现依赖于CLH队列的阻塞锁和相关的并发同步器。**子类通过实现判定是否能获取/释放资源的protect方法，AQS基于这些protect方法实现对线程的排队、唤醒的线程调度策略**

AQS还提供一个支持线程安全原子更新的int类型变量作为同步状态值(state)，子类可以根据实际需求，灵活定义该变量代表的意义进行更新

通过子类重新定义的系列protect方法如下：

- boolean tryAcquire(int) 独占方式尝试获取资源，成功则返回true，失败则返回false
- boolean tryRelease(int) 独占方式尝试释放资源，成功则返回true，失败则返回false
- int tryAcquireShared(int) 共享方式尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源
- boolean tryReleaseShared(int) 共享方式尝试释放资源，如果释放后允许唤醒后续等待节点返回true，否则返回false

这些方法始终由需要需要调度协作的线程来调用，子类须以非阻塞的方式重新定义这些方法

AQS基于上述tryXXX方法，对外提供下列方法来获取/释放资源：

- void acquire(int) 独占方式获取到资源，线程直接返回，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响
- boolean release(int) 独占方式下线程释放资源，先释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源
- void acquireShared(int) 共享方式获取资源
- boolean releaseShared(int) 共享方式释放资源

以独占模式为例：获取/释放资源的核心的实现如下：



```objectivec
 Acquire:
     while (!tryAcquire(arg)) {
        如果线程尚未排队，则将其加入队列；
     }

 Release:
     if (tryRelease(arg))
        唤醒CLH中第一个排队线程
```

到这里，有点绕，下面一张图把上面介绍到的设计思路再重新捋一捋：

![img](https:////upload-images.jianshu.io/upload_images/5618238-90bec84778552f6e.png?imageMogr2/auto-orient/strip|imageView2/2/w/814/format/webp)

AQS基本设计

## 特性实现

下面介绍基于AQS的对象锁、并发工具的一系列功能特性的实现原理

### 1 显式获取

该特性还是以ReentrantLock锁为例，ReentrantLock是可重入对象锁，线程每次请求获取成功一次锁，同步状态值state加1，释放锁state减1，state为0代表没有任何线程持有锁

ReentrantLock锁支持公平/非公平特性，下面的显式获取特性以公平锁为例

#### (1) 阻塞等待获取

基本实现如下：

- 1、ReentrantLock实现AQS的tryAcquire(int)方法，先判断：如果没有任何线程持有锁，或者当前线程已经持有锁，则返回true，否则返回false
- 2、AQS的acquire(int)方法判断当前节点是否为head且基于tryAcquire(int)能否获得资源，如果不能获得，则加入CLH队列排队阻塞等待
- 3、ReentrantLock的lock()方法基于AQS的acquire(int)方法阻塞等待获取锁

ReentrantLock中的tryAcquire(int)方法实现：



```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    // 没有任何线程持有锁
    if (c == 0) {
        // 通过CLH队列的head判断没有别的线程在比当前更早acquires
        // 且基于CAS设置state成功(期望的state旧值为0)
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            // 设置持有锁的线程为当前线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 持有锁的线程为当前线程
    else if (current == getExclusiveOwnerThread()) {
        // 仅仅在当前线程，单线程，不用基于CAS更新
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // 其他线程已经持有锁
    return false;
}
```

AQS的acquire(int)方法实现



```java
public final void acquire(int arg) {
        // tryAcquire检查释放能获取成功
        // addWaiter 构建CLH的节点对象并入队
        // acquireQueued线程阻塞等待
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        // acquireQueued返回true，代表线程在获取资源的过程中被中断
        // 则调用该方法将线程中断标志位设置为true
        selfInterrupt();
}


final boolean acquireQueued(final Node node, int arg) {
    // 标记是否成功拿到资源
    boolean failed = true;
    try {
        // 标记等待过程中是否被中断过
        boolean interrupted = false;
        // 循环直到资源释放
        for (;;) {
            // 拿到前驱节点
            final Node p = node.predecessor();
            
            // 如果前驱是head，即本节点是第二个节点，才有资格去尝试获取资源
            // 可能是head释放完资源唤醒本节点，也可能被interrupt()
            if (p == head && tryAcquire(arg)) {
                // 成功获取资源
                setHead(node);
                // help GC
                p.next = null; 
                failed = false;
                return interrupted;
            }
            
            // 需要排队阻塞等待
            // 如果在过程中线程中断，不响应中断
            // 且继续排队获取资源，设置interrupted变量为true
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

#### (2) 无阻塞尝试获取

ReentrantLock中的tryLock()的实现仅仅是非公平锁实现，实现逻辑基本与tryAcquire一致，不同的是没有通过hasQueuedPredecessors()检查CLH队列的head是否有其他线程在等待，这样当资源释放时，有线程请求资源能插队优先获取

ReentrantLock中tryLock()具体实现如下：



```java
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}

final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    // 没有任何线程持有锁
    if (c == 0) {
        // 基于CAS设置state成功(期望的state旧值为0)
        // 没有检查CLH队列中是否有线程在等待
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 持有锁的线程为当前线程
    else if (current == getExclusiveOwnerThread()) {
        // 仅仅在当前线程，单线程，不用基于CAS更新
        int nextc = c + acquires;
        if (nextc < 0) // overflow，整数溢出
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // 其他线程已经持有锁
    return false;
}
```

#### (3) 指定时间内阻塞等待获取

基本实现如下：

- 1、ReentrantLock的tryLock(long, TimeUnit)调用AQS的tryAcquireNanos(int, long)方法
- 2、AQS的tryAcquireNanos先调用tryAcquire(int)尝试获取，获取不到再调用doAcquireNanos(int, long)方法
- 3、AQS的doAcquireNanos判断当前节点是否为head且基于tryAcquire(int)能否获得资源，如果不能获得且超时时间大于1微秒，则休眠一段时间后再尝试获取

ReentrantLock中的实现如下：



```java
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}

public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    // 如果线程已经被interrupt()方法设置中断 
    if (Thread.interrupted())
        throw new InterruptedException();
    // 先tryAcquire尝试获取锁 
    return tryAcquire(arg) ||
        doAcquireNanos(arg, nanosTimeout);
}
```

AQS中的实现如下：



```java
private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    // 获取到资源的截止时间   
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    // 标记是否成功拿到资源
    boolean failed = true;
    try {
        for (;;) {
            // 拿到前驱节点
            final Node p = node.predecessor();
            // 如果前驱是head，即本节点是第二个节点，才有资格去尝试获取资源
            // 可能是head释放完资源唤醒本节点，也可能被interrupt()
            if (p == head && tryAcquire(arg)) {
                // 成功获取资源
                setHead(node);
                // help GC
                p.next = null; 
                failed = false;
                return true;
            }
            // 更新剩余超时时间
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            // 排队是否需要排队阻塞等待 
            // 且超时时间大于1微秒，则线程休眠到超时时间到了再尝试获取
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);

            // 如果线程已经被interrupt()方法设置中断
            // 则不再排队，直接退出   
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

#### (4) 响应中断获取

ReentrantLock响应中断获取锁的方式是：当线程在park方法休眠中响应thead.interrupt()方法中断唤醒时，检查到线程中断标志位为true，主动抛出异常，核心实现在AQS的doAcquireInterruptibly(int)方法中

基本实现与阻塞等待获取类似，只是调用从AQS的acquire(int)方法，改为调用AQS的doAcquireInterruptibly(int)方法



```java
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    // 标记是否成功拿到资源
    boolean failed = true;
    try {
        for (;;) {
            // 拿到前驱节点
            final Node p = node.predecessor();
            
            // 如果前驱是head，即本节点是第二个节点，才有资格去尝试获取资源
            // 可能是head释放完资源唤醒本节点，也可能被interrupt()
            if (p == head && tryAcquire(arg)) {
                // 成功获取资源
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            
            // 需要排队阻塞等待
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 从排队阻塞中唤醒，如果检查到中断标志位为true
                parkAndCheckInterrupt())
                // 主动响应中断
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### 2 显式释放

AQS资源共享方式分为独占式和共享式，这里先以ReentrantLock为例介绍独占式资源的显式释放，共享式后面会介绍到

与显式获取有类似之处，ReentrantLock显式释放基本实现如下：

- 1、ReentrantLock实现AQS的tryRelease(int)方法，方法将state变量减1，如果state变成0代表没有任何线程持有锁，返回true，否则返回false
- 2、AQS的release(int)方法基于tryRelease(int)排队是否有任何线程持有资源，如果没有，则唤醒CLH队列中头节点的线程
- 3、被唤醒后的线程继续执行acquireQueued(Node,int)或者doAcquireNanos(int, long)或者doAcquireInterruptibly(int)中for(;;)中的逻辑，继续尝试获取资源

ReentrantLock中tryRelease(int)方法实现如下：



```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    // 只有持有锁的线程才有资格释放锁
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
        
    // 标识是否没有任何线程持有锁    
    boolean free = false;
    
    // 没有任何线程持有锁
    // 可重入锁每lock一次都需要对应一次unlock
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

AQS中的release(int)方法实现如下：



```java
public final boolean release(int arg) {
    // 尝试释放资源
    if (tryRelease(arg)) {
        Node h = head;
        // 头节点不为空
        // 后继节点入队后进入休眠状态之前，会将前驱节点的状态更新为SIGNAL(-1)
        // 头节点状态为0，代表没有后继的等待节点
        if (h != null && h.waitStatus != 0)
            // 唤醒第二个节点
            // 头节点是占用资源的线程，第二个节点才是首个等待资源的线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

### 3 可重入

可重入的实现比较简单，以ReentrantLock为例，主要是在tryAcquire(int)方法中实现，持有锁的线程是不是当前线程，如果是，更新同步状态值state，并返回true，代表能获取锁

### 4 可共享

可共享资源以ReentrantReadWriteLock为例，跟独占锁ReentrantLock的区别主要在于，获取的时候，多个线程允许共享读锁，当写锁释放时，多个阻塞等待读锁的线程能同时获取到

ReentrantReadWriteLock类中将AQS的state同步状态值定义为，高16位为读锁持有数，低16位为写锁持有锁

ReentrantReadWriteLock中tryAcquireShared(int)、tryReleaseShared(int)实现的逻辑较长，主要涉及读写互斥、可重入判断、读锁对写锁的让步，篇幅所限，这里就不展开了

**获取读锁(ReadLock.lock())主要实现如下**：

- 1、ReentrantReadWriteLock实现AQS的tryAcquireShared(int)方法，判断当前线程能否获得读锁
- 2、AQS的acquireShared(int)先基于tryAcquireShared(int)尝试获取资源，如果获取失败，则加入CLH队列排队阻塞等待
- 3、ReentrantReadWriteLock的ReadLock.lock()方法基于AQS的acquireShared(int)方法阻塞等待获取锁

AQS共享模式获取资源的具体实现如下：



```java
public final void acquireShared(int arg) {
    // tryAcquireShared返回负数代表获取共享资源失败
    // 则通过进入等待队列，直到获取到资源为止才返回
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

// 与前面介绍到的acquireQueued逻辑基本一致
// 不同的是将tryAcquire改为tryAcquireShared
// 还有资源获取成功后将传播给CLH队列上等待该资源的节点
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
     // 标记是否成功拿到资源
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                // 资源获取成功
                if (r >= 0) {
                    // 传播给CLH队列上等待该资源的节点                             
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            // 需要排队阻塞等待
            // 如果在过程中线程中断，不响应中断
            // 且继续排队获取资源，设置interrupted变量为true
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

//  资源传播给CLH队列上等待该资源的节点 
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; 
    setHead(node);
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            // 释放共享资源
            doReleaseShared();
    }
}
```

**释放读锁(ReadLock.unlock())主要实现如下**：
 ReentrantReadWriteLock共享资源的释放主要实现如下：

- 1、ReentrantReadWriteLock实现AQS的tryReleaseShared(int)方法，判断读锁释放后是否还有线程持有读锁
- 2、AQS的releaseShared(int)基于tryReleaseShared(int)判断是否需要CLH队列中的休眠线程，如果需要就执行doReleaseShared()
- 3、ReentrantReadWriteLock的ReadLock.unlock()方法基于AQS的releaseShared(int)方法释放锁

AQS共享模式释放资源具体实现如下：



```java
public final boolean releaseShared(int arg) {
    // 允许唤醒CLH中的休眠线程
    if (tryReleaseShared(arg)) {
        // 执行资源释放
        doReleaseShared();
        return true;
    }
    return false;
}
    
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            // 当前节点正在等待资源
            if (ws == Node.SIGNAL) {
                // 当前节点被其他线程唤醒了
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            
                unparkSuccessor(h);
            }
            // 进入else的条件是，当前节点刚刚成为头节点
            // 尾节点刚刚加入CLH队列，还没在休眠前将前驱节点状态改为SIGNAL
            // CAS失败是尾节点已经在休眠前将前驱节点状态改为SIGNAL
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;               
        }
        // 每次唤醒后驱节点后，线程进入doAcquireShared方法，然后更新head
        // 如果h变量在本轮循环中没有被改变，说明head == tail，队列中节点全部被唤醒
        if (h == head)                 
            break;
    }
}
```

### 5 公平、非公平

这个特性实现比较简单，以ReentrantLock锁为例，公平锁直接基于AQS的acquire(int)获取资源，而非公平锁先尝试插队：基于CAS，期望state同步变量值为0(没有任何线程持有锁)，更新为1，如果全部CAS更新失败再进行排队



```java
// 公平锁实现
final void lock() {
    acquire(1);
}

// 非公平锁实现
final void lock() {
    // 第1次CAS
    // state值为0代表没有任何线程持有锁，直接插队获得锁
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}

// 在nonfairTryAcquire方法中再次CAS尝试获取锁
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 第2次CAS尝试获取锁
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 已经获得锁
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

## 总结

AQS的state变量值的含义不一定代表资源，不同的AQS的继承类可以对state变量值有不同的定义

例如在countDownLatch类中，state变量值代表还需释放的latch计数(可以理解为需要打开的门闩数)，需要每个门闩都打开，门才能打开，所有等待线程才会开始执行，每次countDown()就会对state变量减1，如果state变量减为0，则唤醒CLH队列中的休眠线程



# 乐观锁悲观锁AtomicInteger的CAS算法

https://juejin.im/user/5b55e6aef265da0faf71cb50/posts

## 前言

Java中有各式各样的锁，主流的锁和概念如下：

![img](https://user-gold-cdn.xitu.io/2018/12/1/167684787d8b7b05?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这篇文章主要是为了让大家通过乐观锁和悲观锁出发，理解CAS算法，因为CAS是整个Concurrent包的基础。

## 乐观锁和悲观锁

首先，java和数据库中都有这种概念，他只是一种广义的概念（从线程同步的角度上看）：

悲观锁：悲观的认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java中，synchronized关键字和Lock的实现类都是悲观锁。

乐观锁：乐观的认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。

根据从上面的概念描述我们可以发现：

- **悲观锁适合写操作多的场景**，先加锁可以保证写操作时数据正确。
- **乐观锁适合读操作多的场景**，不加锁的特点能够使其读操作的性能大幅提升。

从代码层面理解：

悲观锁：

```
// ------------------------- 悲观锁的使用方法 -------------------------
// synchronized
public synchronized void testMethod() {
    // 操作同步资源
}
// ReentrantLock
private ReentrantLock lock = new ReentrantLock(); // 需要保证多个线程使用的是同一个锁
public void modifyPublicResources() {
    lock.lock();
    // 操作同步资源
    lock.unlock();
}复制代码
```

乐观锁：

```
private AtomicInteger atomicInteger = new AtomicInteger();  // 需要保证多个线程使用的是同一个AtomicInteger
atomicInteger.incrementAndGet(); //执行自增1复制代码
```

悲观锁基本上比较好理解：就是在显示的锁定资源后再操作同步资源。

那么问题来了：

乐观锁不锁定资源是如何实现线程同步的呢？

答案是**CAS**

### CAS

CAS全称 Compare And Swap（比较与交换），本质上是一种无锁算法：就是在没有锁的情况下实现同步。

CAS相关的三个操作数：

- 需要读写的内存值 V。
- 需要进行比较的值 A。
- 要写入的新值 B。

当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

先看一下基本的定义：

什么是unsafe呢？Java没办法直接访问底层操作系统，但是JVM为我们提供了一个后门，它后门就是unsafe。unsafe为我们提供了**硬件级别的原子操作**。

对于valueOffset对象，是通过unsafe.objectFieldOffset方法得到，所代表的是**AtomicInteger对象value成员变量在内存中的偏移量**。我们可以简单地把valueOffset理解为value变量的内存地址。

![img](https://user-gold-cdn.xitu.io/2018/12/1/167687e1599040cf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

接下来看incrementAndGet：

```
// ------------------------- JDK 8 -------------------------
// AtomicInteger 自增方法
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

// Unsafe.class
public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
      var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  return var5;
}复制代码
```

getAndAddInt方法的入参：var1是aotmicInteger对象，var2是valueOffset，var4是1；它本质上在循环获取对象aotmicInteger中的偏移量（即valueoffset）处的值var5，然后判断内存的值是否等于var5；如果相等则将内存的值设置为var5+1；否则继续循环重试，直到设置成功时再推出循环。

CAS操作封装在compareAndSwapInt方法内部，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。后续JDK通过CPU的cmpxchg指令，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止。

这里native方法比较多，如果觉得不太好理解，我们可以通俗的总结一下：

**循环体当中做了三件事：** 

**1.获取当前值。 （通过volatile关键字保证可见性）**

**2.计算出目标值：本例子中为****当前值+1** 

**3.进行CAS操作，如果成功则跳出循环，如果失败则重复上述步骤。** 



## CAS的问题

### ABA问题

什么是ABA呢？

因为CAS中的当前值和目标值都是随机的，假设内存中有一个值为A的变量，存储在地址V当中：

**内存地址V→A**

此时有三个线程想使用CAS的方式更新这个变量值，每个线程的执行时间有略微的偏差。线程1和线程2已经获得当前值，线程3还未获得当前值。

线程1：获取到了A，计算目标值，期望更新为B

线程2：获取到了A，计算目标值，期望更新为B

线程3：还没有获取到当前值



接下来，线程1先一步执行成功，把当前值成功从A更新为B；同时线程2因为某种原因被阻塞住，没有做更新操作；线程3在线程1更新之后，获得了当前值B。

**内存地址V→B**

线程1：获取到了A，**成功更新为B**

线程2：获取到了A，计算目标值，期望更新为B，**Block**

线程3：**获取当前值B**，计算目标值，期望更新为A



线程2仍然处于阻塞状态，线程3继续执行，成功把当前值从B更新成了A。

**内存地址V→A**

线程1：获取到了A，成功更新为B，已返回

线程2：获取到了A，计算目标值，期望更新为B，**Block**

线程3：获取当前值B，**成功更新为A**

**
**

最后，线程2终于恢复了运行状态，由于阻塞之前已经获得了“当前值”A，并且经过compare检测，内存地址V中的实际值也是A，所以成功把变量值A更新成了B。

**内存地址V→B**

线程1：获取到了A，成功更新为B，已返回

线程2：获取到了“当前值”A，成功更新为B

线程3：获取当前值B，成功更新为A，已返回

**这个过程中，线程2获取到的变量值A是一个旧值，尽管和当前的实际值相同，但内存地址V中的变量已经经历了A->B->A的改变。**

可这样的话看起来好像也没毛病。

接下来我们来结合实际的场景分析它：

我们假设有一个CAS原理的ATM，小明有100元存款，要取钱50元。

由于提款机硬件出了点小问题，提款操作被同时提交两次，两个线程都是获取当前值100元，要更新成50元。理想情况下，应该一个线程更新成功，另一个线程更新失败，小明的存款只被扣一次。

**存款余额：100元**

**ATM线程1：获取当前值100，期望更新为50**

**ATM线程2：获取当前值100，期望更新为50**



线程1首先执行成功，把余额从100改成50。线程2因为某种原因阻塞了。**这时候，他的妈妈刚好给小明汇款50元**。

**存款余额：50元**

**ATM线程1：获取当前值100，成功更新为50**

**ATM线程2：获取当前值100，期望更新为50，Block**

**线程3（他妈来存钱了）：获取当前值50，期望更新为100**



线程2仍然是阻塞状态，线程3执行成功，把余额从50改成100。

**存款余额：100元**

**ATM线程1：获取当前值100，成功更新为50，已返回**

**ATM线程2：获取当前值100，期望更新为50，Block**

**线程3（他妈来存钱了）：获取当前值50，成功更新为100**



线程2恢复运行，由于阻塞之前已经获得了“当前值”100，并且经过compare检测，此时存款实际值也是100，所以成功把变量值100更新成了50。

**存款余额：50元**

**ATM线程1：获取当前值100，成功更新为50，已返回**

**ATM线程2：获取“当前值”100，成功更新为50**

**线程3（他妈来存钱了）：获取当前值50，成功更新为100，已返回**

这下问题就来了，小明的50元钱白白没有了，原本线程2应当提交失败，小灰的正确余额应该保持为100元，结果由于ABA问题提交成功了。

### 如何解决ABA问题呢

思路和乐观锁差不多，采用版本号就行了，在compare阶段不仅要比较期望值A和地址V中的实际值，还要比较版本号是否一致。

我们仍然以最初的例子来说明一下，假设地址V中存储着变量值A，当前版本号是01。线程1获得了当前值A和版本号01，想要更新为B，但是被阻塞了。

**版本号01：内存地址V→A**

线程1：获取当前值A，版本号01，期望更新为B

这时候发生ABA问题，内存中的值发生了多次改变，最后仍然是A，版本号提升为03

**
**

**版本号03：内存地址V→A**

线程1：获取当前值A，版本号01，期望更新为B

随后线程1恢复运行，发现版本号不相等，所以更新失败。

具体的可以参考java中的**AtomicStampedReference它用版本号比较做了CAS机制。**

## 总结

CAS原理差不多了，它虽然高效，但是有如下问题：

1、ABA问题，可以通过版本号解决

2、循环时间长，开销比较大：如果并发量相当高，CAS操作长时间不成功时，会导致其一直自旋，带来CPU消耗比较大

补充一下自旋锁和非自旋锁的概念：

![img](https://user-gold-cdn.xitu.io/2018/12/1/16768b625c4f6bca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

CAS作为concurrent包基础中的基础，在战胜并发编程的旅途中有着举足轻重的地位，接下来我们将基于CAS讲解，Concurrent包中最基础的组件，我们常用的ReetrantLock SemaPhore LinkedBlockingQueue ArrayBlockingQueue都是基于它实现的。它就是**AQS。**

**传送门：https://juejin.im/post/5c021b59f265da6175737f0b**


作者：lyowish
链接：https://juejin.im/post/5c021da16fb9a049e65ffcbf
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



# 基于ReentrantLock理解AQS同步队列

## 前言

之前介绍过并发问题的解决方式就是一般通过锁，concurrent包中最重要的接口就是lock接口，它可以显示的获取或者释放锁，对于lock接口来说最常见的实现就是ReetrantLock（可重入锁），而ReetrantLock的实现又离不开AQS。

AQS是concurrent包中最核心的并发组件，在读本文之前建议先阅读：

https://juejin.im/post/5c021da16fb9a049e65ffcbf 彻底理解CAS机制，因为CAS在整个ReetrantLock中随处可见，它是lock的基础。

网上有许多类似文章，但是这一部分的东西比较抽象，需要不断理解，本文将基于源码分析concurrent包的最核心的组件AQS，将不好理解的部分尽量用图片来分析彻底理解ReetrantLock的原理

这部分是concurrent包的核心，理解了之后再去理解SemaPhore LinkedBlockingQueue ArrayBlockingQueue 等就信手拈来了，所以会花比较多的篇幅

## 重入锁ReetrantLock

### Lock接口

先大概看一看lock接口

```
public interface Lock {
    // 加锁
    void lock();
    // 可中断获取锁，获取锁的过程中可以中断。
    void lockInterruptibly() throws InterruptedException;
    //立即返回的获取锁，返回true表示成功，false表示失败
    boolean tryLock();
    //根据传入时间立即返回的获取锁，返回true表示成功，false表示失败
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    //解锁
    void unlock();
    //获取等待的条件队列（之后会详细分析）
    Condition newCondition();
}复制代码
```

而我们一般使用ReetrantLock：









```
Lock lock = new ReentrantLock(); 
lock.lock(); 
try{ 
 //业务代码...... 
}finally{ 
 lock.unlock(); 
}复制代码
```

它在使用上是比较简单的，在正式分析之前我们先看看什么是公平锁和非公平锁

### 公平锁和非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入FIFO队列，队列中的第一个线程才能获得锁。

用一个打水的例子来理解：

![img](https://user-gold-cdn.xitu.io/2018/12/2/1676e3e42c3d4c39?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**公平锁的优点是等待锁的线程不会夯死。缺点是吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。**



非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取锁的场景。

![img](https://user-gold-cdn.xitu.io/2018/12/2/1676e3f96bc48ce6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**非公平锁的优点是可以减少唤起线程的开销（因为可能有的线程可以直接获取到锁，CPU也就不用唤醒它），所以整体的吞吐效率高。缺点是处于等待队列中的线程可能会夯死（试想恰好每次有新线程来，它恰巧都每次获取到锁，此时还在排队等待获取锁的线程就悲剧了****），或者等很久才会获得锁。**

### 总结

公平锁和非公平锁的差异在于是否按照申请锁的顺序来获取锁，非公平锁可能会出现有多个线程等待时，有一个人品特别的好的线程直接没有等待而直接获取到了锁的情况，他们各有利弊;ReetrantLock在构造时默认是非公平的，可以通过参数控制。

## ReetrantLock与AQS

这里以ReentrantLock为例，简单讲解ReentrantLock与AQS的关系

![img](https://user-gold-cdn.xitu.io/2018/12/2/1676e47ee14e6eda?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2018/12/2/1676e534bfc9c110?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

从上图我们可以总结：

1. ReetrantLock:实现了lock接口，它的内部类有Sync、NonfairSync、FairSync（他们三个是继承了AQS），创建构造ReetrantLock时可以指定是非公平锁(NonfairSync)还是公平锁(FairSync)
2. Sync:他是抽象类，也是ReetrantLock内部类，实现了tryRelease方法，tryAccquire方法由它的子类NonfairSync、FairSync自己实现。
3. AQS：它是一个抽象类，但是值得注意的是它代码中却没有一个抽象方法，其中获取锁(tryAcquire方法)和释放锁(tryRelease方法)也没有提供默认实现，需要子类重写这两个方法实现具体逻辑（典型的模板方法设计模式）。
4. Node：AQS的内部类，本质上是一个双向链表，用来管理获取锁的线程（后续详细解读）

### 这样设计的结构有几点好处：

\1. 首先为什么要有Sync这个内部类呢？

 因为无论是NonfairSync还是FairSync，他们解锁的过程是一样的，不同只是加锁的过程，Sync提供加锁的模板方法让子类自行实现

\2. AQS为什么要声明为Abstract，内部却没有任何abstract方法？

这是因为AQS只是作为一个基础组件，从上图可以看出countDownLatch,Semaphore等并发组件都依赖了它，它并不希望直接作为直接操作类对外输出，而更倾向于作为一个基础并发组件，为真正的实现类提供基础设施，例如构建同步队列，控制同步状态等。

AQS是采用模板方法的设计模式，它作为基础组并发件，封装了一层核心并发操作（比如获取资源成功后封装成Node加入队列，对队列双向链表的处理），但是实现上分为两种模式，即**共享模式(如Semaphore)与独占模式（如ReetrantLock，这两个模式的本质区别在于多个线程能不能共享一把锁）**，而这两种模式的加锁与解锁实现方式是不一样的，但AQS只关注内部公共方法实现并不关心外部不同模式的实现，所以提供了模板方法给子类使用：例如：

ReentrantLock需要自己实现tryAcquire()方法和tryRelease()方法，而实现共享模式的Semaphore，则需要实现tryAcquireShared()方法和tryReleaseShared()方法，这样做的好处？因为无论是共享模式还是独占模式，其基础的实现都是同一套组件(AQS)，只不过是加锁解锁的逻辑不同罢了，更重要的是如果我们需要自定义锁的话，也变得非常简单，只需要选择不同的模式实现不同的加锁和解锁的模板方法即可。

### 总结

ReetrantLock:实现了lock接口，内部类有Sync、NonfairSync、FairSync（他们三个是继承了AQS）这里用了模板方法的设计模式。

### Node和AQS工作原理

之前介绍AQS是提供基础设施，如构建同步队列，控制同步状态等，它的工作原理是怎样的呢？

我们先看看AQS类中几个重要的字段：







```
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer{
//指向同步队列队头
private transient volatile Node head;

//指向同步的队尾
private transient volatile Node tail;

//同步状态，0代表锁未被占用，1代表锁已被占用
private volatile int state;

//省略.
}
复制代码
```



再看看Node这个内部类：**它是对每一个访问同步代码块的线程的封装**

关于等待状态，我们暂时只需关注SIGNAL 和初始化状态即可

![img](https://user-gold-cdn.xitu.io/2018/12/2/1676e838ec68e07e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

AQS本质上就是由node构成的双向链表，内部有node head和node tail。

AQS通过定义的state字段来控制同步状态,当state=0时，说明没有锁资源被站东，当state=1时，说明有线程目前正在使用锁的资源，这个时候其他线程必须加入同步队列进行等待；

既然要加入队列，那么AQS是内部通过内部类Node构成FIFO的同步队列实现线程获取锁排队，同时利用内部类ConditionObject构建条件队列，当调用condition.wait()方法后，线程将会加入条件队列中，而当调用signal()方法后，线程将从条件队列移动到同步队列中进行锁竞争。注意这里涉及到两种队列，一种的**同步队列**，当锁资源已经被占用，而又有线程请求锁而等待的后将加入同步队列等待，而另一种则是**条件队列**(可有多个)，通过Condition调用await()方法释放锁后，将加入等待队列。

条件队列可以暂时先放一边，下一节再详细分析，因为当我们调用ReetrantLock.lock()方法时，实际操作的是基于node结构的同步队列，此时AQS中的**state**变量则是代表**同步状态**，加锁后，如果此时state的值为0，则说明当前线程可以获取到锁，同时将state设置为1，表示获取成功。如果调用ReetrantLock.lock()方法时state已为1，也就是当前锁已被其他线程持有，那么当前执行线程将被封装为Node结点加入同步队列等待。

![img](https://user-gold-cdn.xitu.io/2018/12/2/1676e9e8227d0743?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 总结

如上图所示为AQS的同步队列模型；

1. AQS内部有一个由Node组成的同步队列，它是双向链表结构
2. AQS内部通过state来控制同步状态,当执行lock时，如果state=0时，说明没有任何线程占有共享资源的锁，此时线程会获取到锁并把state设置为1；当state=1时，则说明有线程目前正在使用共享变量，其他线程必须加入同步队列进行等待.

AQS内部分为共享模式(如Semaphore)和独占模式(如ReetrantLock)，无论是共享模式还是独占模式的实现类，都维持着一个虚拟的同步队列，当请求锁的线程超过现有模式的限制时，会将线程包装成Node结点并将线程当前必要的信息存储到node结点中，然后加入同步队列等会获取锁，而这系列操作都有AQS协助我们完成，这也是作为基础组件的原因，无论是Semaphore还是ReetrantLock，其内部绝大多数方法都是间接调用AQS完成的。

接下来我们看详细实现

## 基于ReetrantLock分析AQS独占锁模式的实现

### 非公平锁的获取锁

AQS的实现依赖于内部的同步队列(就是一个由node构成的FIFO的双向链表对列)来完成对同步状态(state)的管理，当前线程获取锁失败时，AQS会将该线程封装成一个Node并将其加入同步队列，同时会阻塞当前线程，当同步资源释放时，又会将头结点head中的线程唤醒，让其尝试获取同步状态。这里从ReetrantLock入手分析AQS的具体实现，我们先以非公平锁为例进行分析。

### 获取锁

来看ReetrantLock的源码：







```
//默认构造，创建非公平锁NonfairSync
public ReentrantLock() {
    sync = new NonfairSync();
}
//根据传入参数创建锁类型
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}

//加锁操作
public void lock() {
     sync.lock();
}

复制代码
```



这里说明ReetrantLock默认构造方法就是构造一个非公平锁，调用lock方法时候：







```
/**
 * 非公平锁实现
 */
static final class NonfairSync extends Sync {
    //加锁
    final void lock() {
        //执行CAS操作，本质就是CAS更新state：
        //判断state是否为0，如果为0则把0更新为1，并返回true否则返回false
        if (compareAndSetState(0, 1))
       //成功则将独占锁线程设置为当前线程  
          setExclusiveOwnerThread(Thread.currentThread());
        else
            //否则再次请求同步状态
            acquire(1);
    }
}

复制代码
```



也就是说，通过CAS机制保证并发的情况下只有一个线程可以成功将state设置为1，获取到锁；

此时，其它线程在执行compareAndSetState时，因为state此时不是0，所以会失败并返回false，执行acquire(1);

### 加入同步队列







```
public final void acquire(int arg) {
    //再次尝试获取同步状态
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

复制代码
```



这里传入参数arg是state的值，因为要获取锁，而status为0时是释放锁，1则是获取锁，所以这里一般传递参数为1，进入方法后首先会执行tryAcquire(1)方法，在前面分析过该方法在AQS中并没有具体实现，而是交由子类实现，因此该方法是由**ReetrantLock**类**内部类实现**的







```
//NonfairSync类
static final class NonfairSync extends Sync {

    protected final boolean tryAcquire(int acquires) {
         return nonfairTryAcquire(acquires);
     }
 }
复制代码
```



![img](https://user-gold-cdn.xitu.io/2018/12/2/1676ec4fc041032f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

假设有三个线程：线程1已经获得到了锁，线程2正在同步队列中排队，此时线程3执行lock方法尝试获取锁的时，线程1正好释放了锁，将state更新为0，那么线程3就可能在线程2还没有被唤醒之前去获取到这个锁。

如果此时还没有获取到锁（nonfairTryAcquire返回false），那么接下来会把该线程封装成node去同步队列里排队，代码层面上执行的是acquireQueued(addWaiter(Node.EXCLUSIVE), arg)

ReetrantLock为独占锁，所以传入的参数为Node.EXCLUSIVE







```
private Node addWaiter(Node mode) {
    //将请求同步状态失败的线程封装成结点
    Node node = new Node(Thread.currentThread(), mode);

    Node pred = tail;
    //如果是第一个结点加入肯定为空，跳过。
    //如果非第一个结点则直接执行CAS入队操作，尝试在尾部快速添加
    if (pred != null) {
        node.prev = pred;
        //使用CAS执行尾部结点替换，尝试在尾部快速添加
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //如果第一次加入或者CAS操作没有成功执行enq入队操作
    enq(node);
    return node;
}
复制代码
```



其中tail是AQS的成员变量，指向队尾(这点前面的我们分析过AQS维持的是一个双向的链表结构同步队列)，如果第一次获取到锁，AQS还没有初始化，则为tail肯定为空，那么将执行enq(node)操作，如果非第一个结点即tail指向不为null，直接**尝试执行CAS操作加入队尾**（再一次使用CAS操作实现线程安全），如果**CAS操作失败或第一次加入同步队列**还是会执行enq(node)，继续看enq(node)：







```
private Node enq(final Node node) {
    //死循环
    for (;;) {
         Node t = tail;
         //如果队列为null，即没有头结点
         if (t == null) { // Must initialize
             //创建并使用CAS设置头结点
             if (compareAndSetHead(new Node()))
                 tail = head;
         } else {//队尾添加新结点
             node.prev = t;
             if (compareAndSetTail(t, node)) {
                 t.next = node;
                 return t;
             }
         }
     }
}
复制代码
```



这个方法使用一个死循环进行CAS操作，可以解决多线程并发问题。这里做了两件事：

一是队列不存在的创建新结点并初始化tail、head：使用compareAndSetHead设置头结点，head和tail都指向head。

二是队列已存在，则将新结点node添加到队尾。

**注意****addWaiter****和****enq****这两个方法都存在同样的代码将线程设置为同步队列的队尾：**

```
             node.prev = t;
             if (compareAndSetTail(t, node)) {
                 t.next = node;
                 return t;
             }复制代码
```

这是因为，在多线程环境中，假设线程1、2、3、4同时执行addWaiter()方法入队，而此时头节点不为null，那么他们会同时执行addWaiter中的compareAndSetTail方法将队尾指向它，添加到队尾。

但这个时候CAS操作保证只有一个可以成功，假设此时线程1成功添加到队尾，那么线程2、3、4执行CAS都会失败，那么线程2、3、4会在enq这个方法内部**死循环执行compareAndSetTail**方法将队尾指向它，直到成功添加到队尾为止。enq这个方法在内部对并发情况下进行补偿。

### 自旋

回到之前的`acquire()`方法，添加到同步队列后，结点就会进入一个自旋过程，自旋的意思就是原地转圈圈：即结点都在观察时机准备获取同步状态,自旋过程是在`acquireQueued(addWaiter(Node.EXCLUSIVE), arg))`方法中执行的，先看前半部分







```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        //自旋，死循环
        for (;;) {
            //获取前结点
            final Node p = node.predecessor();
            当且仅当p为头结点才尝试获取同步状态,FIFO
            if (p == head && tryAcquire(arg)) {
                //此时当前node前驱节点为head且已经tryAcquire获取到了锁，正在执行了它的相关信息
                //已经没有任何用处了，所以现在需要考虑将它GC掉
                //将node设置为头结点
                setHead(node);
                //清空原来头结点的引用便于GC
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //如果前驱结点不是head，判断是否挂起线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;       
        }
    } finally {
        if (failed)
            //最终都没能获取同步状态，结束该线程的请求
            cancelAcquire(node);
    }
}
复制代码
```









```
//设置为头结点
private void setHead(Node node) {
        head = node;
        //清空结点数据以便于GC
        node.thread = null;
        node.prev = null;
}
复制代码
```



死循环中，如果满足了if (p == head && tryAcquire(arg))

如下图，会执行sethead方法：

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="421"></svg>)

当然如果前驱结点不是head而它又没有获取到锁，那么执行如下：







```
//如果前驱结点不是head，判断是否挂起线程
if (shouldParkAfterFailedAcquire(p, node) &&parkAndCheckInterrupt())

      interrupted = true;
}

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //获取当前结点的等待状态
        int ws = pred.waitStatus;
        //如果为等待唤醒（SIGNAL）状态则返回true
        if (ws == Node.SIGNAL)
            return true;
        //如果ws>0 则说明是结束状态，
        //遍历前驱结点直到找到没有结束状态的结点
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //如果ws小于0又不是SIGNAL状态，
            //则将其设置为SIGNAL状态，代表该结点的线程正在等待唤醒。
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

private final boolean parkAndCheckInterrupt() {
        //将当前线程挂起,线程会阻塞住
        LockSupport.park(this);
        //获取线程中断状态,interrupted()是判断当前中断状态，
        //并非中断线程，因此可能true也可能false,并返回
        return Thread.interrupted();
}

复制代码
```



这段代码有个设计比较好的点：

通常我们在设计队列时，我们需要考虑如何最大化的减少后续排队节点对于CPU的消耗，而在AQS中，只要当前节点的前驱节点不是头结点，再把当前节点加到队列后就会执行LockSupport.park(this);将当前线程挂起，这样可以最大程度减少CPU消耗。

### 总结：

是不是还是有点一头雾水？

没关系，为了方便理解：我们假设ABC三个线程现在同时去获取锁，A首先获取到锁后一直不释放，BC加入队列。那么对于AQS的同步队列结构是如何变化的呢？

**1、A直接获取到锁:**

代码执行路径：

(ReetranLock.lock()-> compareAndSetState(0, 1) -> setExclusiveOwnerThread(Thread.currentThread())

此时AQS结构还没有初始化：

![img](https://user-gold-cdn.xitu.io/2018/12/3/16773697e309231a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**2、B尝试获取锁：**

因为A存在把state设置为1，所以B获取锁失败，进行入队操作加入同步队列，入队时发现AQS还没有初始化(AQS中的tail为null)，会在入队前初始化代码在enq方法的死循环中：

![img](https://user-gold-cdn.xitu.io/2018/12/3/167736bd22385bdb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

初始化之后改变tail的prev指向，把自己加到队尾：

![img](https://user-gold-cdn.xitu.io/2018/12/3/1677370f08fe59c5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

接着会执行acquireQueued方法：

```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}复制代码
```

第一次执行：发现自己前序节点是head节点，于是乎再次尝试获取锁，获取失败后再shouldParkAfterFailedAcquire方法中把前序节点设置为Singal状态

第二次执行：再次尝试获取锁，但因为前序节点是Signal状态了,所以执行parkAndCheckInterrupt把自己休眠起来进行自旋

![img](https://user-gold-cdn.xitu.io/2018/12/3/167743b684a0e673?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**3、C尝试获取锁：**

C获取锁和B完全一样，不同的是它的前序节点是B，所以它并不会一直尝试获取锁，只会呆在B后面park住

![img](https://user-gold-cdn.xitu.io/2018/12/3/167743aa97cd14dc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

AQS通过最简单的CAS和LockSupport的park，设计出了高效的队列模型和机制：

1、AQS结构其实是在第二个线程获取锁的时候再初始化的，就是lazy-Init的思想，最大程度减少不必要的代码执行的开销

2、为了最大程度上提升效率，尽量避免线程间的通讯，采用了双向链表的Node结构去存储线程

3、为了最大程度上避免CPU上下文切换执行的消耗，在设计排队线程时，只有头结点的下一个的线程在一直重复执行获取锁，队列后面的线程会通过LockSupport进行休眠。

### 非公平锁的释放锁

上代码：







```
//ReentrantLock类的unlock
public void unlock() {
    sync.release(1);
}

//AQS类的release()方法
public final boolean release(int arg) {
    //尝试释放锁
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒后继结点的线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}

//ReentrantLock类中的内部类Sync实现的tryRelease(int releases) 
protected final boolean tryRelease(int releases) {

      int c = getState() - releases;
      if (Thread.currentThread() != getExclusiveOwnerThread())
          throw new IllegalMonitorStateException();
      boolean free = false;
      //判断状态是否为0，如果是则说明已释放同步状态
      if (c == 0) {
          free = true;
          //设置Owner为null
          setExclusiveOwnerThread(null);
      }
      //设置更新同步状态
      setState(c);
      return free;
  }

复制代码
```



一句话总结：释放锁首先就是把volatile类型的变量state减1。state从1变成0.

unparkSuccessor(h)的作用的唤醒后续的节点：







```
private void unparkSuccessor(Node node) {
    //这里，node是head节点。
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}

复制代码
```



从代码执行操作来看，这里主要作用是用unpark()唤醒同步队列中最前边未放弃线程(也就是状态为CANCELLED的线程结点s)。此时，回忆前面分析进入自旋的函数acquireQueued()，s结点的线程被唤醒后，会进入acquireQueued()函数的if (p == head && tryAcquire(arg))的判断，然后s把自己设置成head结点，表示自己已经获取到资源了，最终acquire()也返回了，这就是独占锁释放的过程。 

### 总结

回到之前的图：A B C三个线程获取锁，A已经获取到了锁，BC在队列里面，此时A释放锁

![img](https://user-gold-cdn.xitu.io/2018/12/3/167743aa97cd14dc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

执行b.unpark，B被唤醒后继续执行







```
if (p == head && tryAcquire(arg))
复制代码
```



因为B的前序节点是head，所以会执行tryAcquire方法尝试获取锁，获取到锁之后执行setHead方法把自己设置为头节点，并且把之前的头结点也就是上图中的new Node()设置为null以便于GC：







```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                //获取到锁之后将当前node设置为头结点 head指向当前节点node
                setHead(node);
                //p.next就是之前的头结点，它没有用了，所以把它gc掉
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}复制代码
```



![img](https://user-gold-cdn.xitu.io/2018/12/3/167745fd437a69f3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

总之，AQS内部有一个同步队列，线程获取同步状态失败之后会被封装成node通过park进行自旋，而在释放同步状态时，通过unpark进行唤醒后面一个线程，让后面线程得以继续获取锁。

### ReetrantLock中的公平锁

了解完ReetrantLock中非公平锁的实现后，我们再来看看公平锁。与非公平锁不同的是，**在获取锁的时，公平锁的获取顺序是完全遵循时间上的FIFO规则**，也就是说先请求的线程一定会先获取锁，后来的线程肯定需要排队。

下面比较一下公平锁和非公平锁lock方法： 

![img](https://user-gold-cdn.xitu.io/2018/12/3/16774707f746a91c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

再比较一下公平锁和非公平锁lock方法：tryAcquire方法：

![img](https://user-gold-cdn.xitu.io/2018/12/3/1677475ef51cd0e4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2018/12/3/1677475c31a29b9b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

唯一的差别就是`hasQueuedPredecessors()`判断同步队列是否存在结点，这就是非公平锁与公平锁最大的区别，**即公平锁在线程请求到来时先会判断同步队列是否存在结点**，如果存在先执行同步队列中的结点线程，当前线程将封装成node加入同步队列等待。而非公平锁呢，当线程请求到来时，不管同步队列是否存在线程结点，直接上去尝试获取同步状态，获取成功直接访问共享资源，但请注意在绝大多数情况下，非公平锁才是我们理想的选择，毕竟从效率上来说非公平锁总是胜于公平锁。

## 总结

 以上便是ReentrantLock的内部实现原理，这里我们简单进行小结，重入锁ReentrantLock，是一个基于AQS并发框架的并发控制类，其内部实现了3个类，分别是Sync、NoFairSync以及FairSync类，其中Sync继承自AQS，实现了释放锁的模板方法tryRelease(int)，而NoFairSync和FairSync都继承自Sync，实现各种获取锁的方法tryAcquire(int)。

ReentrantLock的所有方法实现几乎都间接调用了这3个类，因此当我们在使用ReentrantLock时，大部分使用都是在间接调用AQS同步器中的方法。

AQS在设计时将性能优化到了极致，具体体现在同步队列的park和unpark，初始化AQS时的懒加载，以及线程之间通过Node这样的数据结构从而避免线程间通讯造成的额外开销，这种由释放锁的线程主动唤醒后续线程的方式也是我们再实际过程中可以借鉴的。

AQS还不止于同步队列，接下来我们会继续探讨**AQS的条件队列**




# 死磕java concurrent包系列（三）基于ReentrantLock理解AQS的条件队列

## 基于Codition分析AQS的条件队列

## 前言

上一篇我们讲了AQS中的同步队列队列，现在我们研究一下条件队列。

在java中最常见的加锁方式就是synchorinzed和Reentrantlock，我们都说Reentrantlock比synchorinzed更加灵活，其实就灵活在Reentrantlock中的条件队列的用法上。

### Condition接口

它是在java1.5中引入的一个接口，主要是为了替代object类中的wait、notify方法，以一种更灵活的方式解决线程之间的通信问题：







```
public interface Condition {

 //使当前线程进入等待状态直到被通知(signal)
 void await() throws InterruptedException;

 //当前线程进入等待状态，直到被唤醒，该方法不响应中断要求
 void awaitUninterruptibly();

 //调用该方法，当前线程进入等待状态，直到被唤醒或被中断或超时
 //其中nanosTimeout指的等待超时时间，单位纳秒
 long awaitNanos(long nanosTimeout) throws InterruptedException;

  //同awaitNanos，但可以指明时间单位
  boolean await(long time, TimeUnit unit) throws InterruptedException;

 //调用该方法当前线程进入等待状态，直到被唤醒、中断或到达某个时
 //间期限(deadline),如果没到指定时间就被唤醒，返回true，其他情况返回false
  boolean awaitUntil(Date deadline) throws InterruptedException;

 //唤醒一个等待在Condition上的线程，该线程从等待方法返回前必须
 //获取与Condition相关联的锁，功能与notify()相同
  void signal();

 //唤醒所有等待在Condition上的线程，该线程从等待方法返回前必须
 //获取与Condition相关联的锁，功能与notifyAll()相同
  void signalAll();
}
复制代码
```



最重要的是await方法使线程进入等待状态，再通过signal方法唤醒。接下来我们结合实际例子分析。

### Condition可以解决什么问题

假设有一个生产者-消费者的场景：

1、生产者有两个线程产生烤鸡；消费者有两个线程消费烤鸡

2、四个线程一起执行，但同时只能有一个生产者线程生成烤鸡，一个消费者线程消费烤鸡。

3、只有产生了烤鸡，才能通知消费线程去消费，否则只能等着；

4、只有消费了烤鸡，才能通知生产者线程去生产，否则只能等着

于是乎，我们使用ReentrantLock控制并发，并使用它生成两组Condition对象，productCondition和consumeCondition：前者控制生产者线程，后者控制消费者线程。当isHaveChicken为true时，代表烤鸡生成完毕，生产线程必须进入等待状态同时唤醒消费线程进行消费，消费线程消费完毕后将flag设置为false，代表烤鸡消费完成，进入等待状态，同时唤醒生产线程生产烤鸡。。。。。。







```
package com.springsingleton.demo.Chicken;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ChikenStore {

  ReentrantLock reentrantLock = new ReentrantLock();

  Condition productCondition = reentrantLock.newCondition();

  Condition consumeCondition = reentrantLock.newCondition();

  private int count = 0;

  private volatile boolean isHaveChicken = false;

  //生产
  public void ProductChicken() {
    reentrantLock.lock();
    while (isHaveChicken) {
      try {
        System.out.println("有烤鸡了" + Thread.currentThread().getName() + "不生产了");
        productCondition.await();
      } catch (Exception e) {
        System.out.println("error" + e.getMessage());
      }
    }
    count++;
    System.out.println(Thread.currentThread().getName() + "产生了第" + count + "个烤鸡，赶紧开始卖");
    isHaveChicken = true;
    consumeCondition.signal();
    reentrantLock.unlock();
  }

  public void SellChicken() {
    reentrantLock.lock();
    while (!isHaveChicken) {
      try {
        System.out.println("没有烤鸡了" + Thread.currentThread().getName() + "不卖了");
        consumeCondition.await();
      } catch (Exception e) {
        System.out.println("error" + e.getMessage());
      }
    }
    count--;
    isHaveChicken = false;
    System.out.println(Thread.currentThread().getName() + "卖掉了第" + count + 1 + "个烤鸡，赶紧开始生产");
    productCondition.signal();
    reentrantLock.unlock();
  }

  public static void main(String[] args) {
    ChikenStore chikenStore = new ChikenStore();
    new Thread(() -> {
      Thread.currentThread().setName("生产者1号");
      while (true) {
        chikenStore.ProductChicken();
      }
    }).start();
    new Thread(() -> {
      Thread.currentThread().setName("生产者2号");
      for (; ; ) {
        chikenStore.ProductChicken();
      }
    }).start();
    new Thread(() -> {
      Thread.currentThread().setName("消费者1号");
      while (true) {
        chikenStore.SellChicken();
      }
    }).start();
    new Thread(() -> {
      Thread.currentThread().setName("消费者2号");
      while (true) {
        chikenStore.SellChicken();
      }
    }).start();

  }
}

复制代码
```

输出：







```
生产者1号产生了第1个烤鸡，赶紧开始卖
有烤鸡了生产者1号不生产了
有烤鸡了生产者2号不生产了
消费者1号卖掉了第01个烤鸡，赶紧开始生产
没有烤鸡了消费者1号不卖了
生产者1号产生了第1个烤鸡，赶紧开始卖
有烤鸡了生产者1号不生产了
消费者1号卖掉了第01个烤鸡，赶紧开始生产
没有烤鸡了消费者1号不卖了
没有烤鸡了消费者2号不卖了
生产者2号产生了第1个烤鸡，赶紧开始卖
有烤鸡了生产者2号不生产了
消费者1号卖掉了第01个烤鸡，赶紧开始生产
没有烤鸡了消费者1号不卖了
生产者1号产生了第1个烤鸡，赶紧开始卖
有烤鸡了生产者1号不生产了
消费者2号卖掉了第01个烤鸡，赶紧开始生产
没有烤鸡了消费者2号不卖了
复制代码
```



如果用synchorinzed的话：







```
package com.springsingleton.demo.Chicken;

public class ChickenStoreSync {

  private int count = 0;

  private volatile boolean isHaveChicken = false;

  public synchronized void ProductChicken() {
    while (isHaveChicken) {
      try {
        System.out.println("有烤鸡了" + Thread.currentThread().getName() + "不生产了");
        this.wait();
      } catch (Exception e) {
        System.out.println("error" + e.getMessage());
      }
    }
    count++;
    System.out.println(Thread.currentThread().getName() + "产生了第" + count + "个烤鸡，赶紧开始卖");
    isHaveChicken = true;
    notifyAll();
  }

  public synchronized void SellChicken() {
    while (!isHaveChicken) {
      try {
        System.out.println("没有烤鸡了" + Thread.currentThread().getName() + "不卖了");
        this.wait();
      } catch (Exception e) {
        System.out.println("error" + e.getMessage());
      }
    }
    count--;
    isHaveChicken = false;
    System.out.println(Thread.currentThread().getName() + "卖掉了第" + count + 1 + "个烤鸡，赶紧开始生产");
    notifyAll();
  }

  public static void main(String[] args) {
    ChickenStoreSync chikenStore = new ChickenStoreSync();
    new Thread(() -> {
      Thread.currentThread().setName("生产者1号");
      while (true) {
        chikenStore.ProductChicken();
      }
    }).start();
    new Thread(() -> {
      Thread.currentThread().setName("生产者2号");
      for (; ; ) {
        chikenStore.ProductChicken();
      }
    }).start();
    new Thread(() -> {
      Thread.currentThread().setName("消费者1号");
      while (true) {
        chikenStore.SellChicken();
      }
    }).start();
    new Thread(() -> {
      Thread.currentThread().setName("消费者2号");
      while (true) {
        chikenStore.SellChicken();
      }
    }).start();

  }
}
复制代码
```



如上代码，在调用notify()或者 notifyAll()方法时，由于synchronized等待队列中同时存在生产者线程和消费者线程，所以我们**并不能保证被唤醒的到底是消费者线程还是生产者线程**，而Codition则可以避免这种情况。

### AQS中Condition的实现原理

Condition的具体实现类是AQS的内部类ConditionObject，之前我们分析过AQS中存在两种队列，一种是同步队列，一种是条件队列，而条件队列是基于Condition实现的。注意在使用Condition前**必须获得锁（因为condition一般是由lock构造出来的，它依赖于lock）**，同时在Condition的条件队列上的也有一个Node节点，其结点的waitStatus的值为CONDITION。在实现类ConditionObject中有两个结点分别是firstWaiter和lastWaiter，firstWaiter代表等待队列第一个等待结点，lastWaiter代表等待队列最后一个等待结点







```
public class ConditionObject implements Condition, java.io.Serializable {
    //等待队列第一个等待结点
    private transient Node firstWaiter;
    //等待队列最后一个等待结点
    private transient Node lastWaiter;
    //省略.......
}
复制代码
```



每个Condition都对应着一个条件队列；一个锁上可以创建多个Condition对象，那么也就存在多个条件队列。条件队列同样是一个FIFO的队列，在队列中每一个节点都包含了一个线程的引用，而该线程就是Condition对象上等待的线程。

当一个线程调用了await()相关的方法，那么该线程将会**释放锁**，并**构建一个Node节点封装当前线程的相关信息加入到条件队列**中进行等待，直到被唤醒、中断、超时才从队列中移出。Condition中的等待队列模型如下

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="454"></svg>)

正如图所示，Node节点的数据结构，和同步队列的node相比，Condtion中等待队列的是一个单向的，而且使用的变量是nextWaiter而不是next，这点我们在前面分析结点Node的数据结构时讲过。firstWaiter指向条件队列的头结点，lastWaiter指向条件队列的尾结点，条件队列中结点的状态只有两种即CANCELLED和CONDITION，前者表示线程已结束需要从等待队列中移除，后者表示条件结点等待被唤醒。

每个Codition对象对于一个条件队列，也就是说AQS中只能存在一个同步队列，但可拥有多个条件队列（之前烤鸡的例子就有两个new出来的condition的队列）。下面从代码层面看看被调用await()方法(其他await()实现原理类似)的线程是如何加入等待队列的，而又是如何从等待队列中被唤醒的。







```
public final void await() throws InterruptedException {
      //判断线程是否被中断
      if (Thread.interrupted())
          throw new InterruptedException();
      //创建新结点加入等待队列并返回
      Node node = addConditionWaiter();
      //释放当前线程锁即释放同步状态
      int savedState = fullyRelease(node);
      int interruptMode = 0;
      //判断结点是否同步队列(SyncQueue)中,即是否被唤醒
      while (!isOnSyncQueue(node)) {
          //挂起线程
          LockSupport.park(this);
          //判断是否被中断唤醒，如果是退出循环。
          if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
              break;
      }
      //被唤醒后 自旋操作争取获得锁，同时判断线程是否被中断
      if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
          interruptMode = REINTERRUPT;
       // clean up if cancelled
      if (node.nextWaiter != null) 
          //清理等待队列中不为CONDITION状态的结点
          unlinkCancelledWaiters();
      if (interruptMode != 0)
          reportInterruptAfterWait(interruptMode);
  }
复制代码
```



再看看addConditionWaiter方法，添加到等待队列：







```
 private Node addConditionWaiter() {
    Node t = lastWaiter;
      // 判断是否为结束状态的结点并移除
      if (t != null && t.waitStatus != Node.CONDITION) {
          unlinkCancelledWaiters();
          t = lastWaiter;
      }
      //创建新结点状态为CONDITION
      Node node = new Node(Thread.currentThread(), Node.CONDITION);
      //加入等待队列
      if (t == null)
          firstWaiter = node;
      else
          t.nextWaiter = node;
      lastWaiter = node;
      return node;
}
复制代码
```



await()方法主要做了3件事：

一是调用addConditionWaiter()方法将当前线程封装成node结点加入等待队列。

二是调用fullyRelease(node)方法释放同步状态并唤醒后继结点的线程。

三是调用isOnSyncQueue(node)方法判断结点是否在同步队列中，这里是个while循环，如果同步队列中没有该结点就直接挂起该线程，需要明白的是如果线程被唤醒后就调用acquireQueued(node, savedState)执行自旋操作争取锁，即当前线程结点从等待队列转移到同步队列并开始努力获取锁。

接下来看看Singnal







```
 public final void signal() {
     //判断是否持有独占锁，如果不是抛出异常
   if (!isHeldExclusively())
          throw new IllegalMonitorStateException();
      Node first = firstWaiter;
      //唤醒等待队列第一个结点的线程
      if (first != null)
          doSignal(first);
 }

复制代码
```



这里`signal()`方法做了两件事：

一是判断当前线程是否持有独占锁，没有就抛异常。

二是唤醒等待队列的第一个结点，即执行`doSignal(first)`







```
private void doSignal(Node first) {
     do {
             //移除条件等待队列中的第一个结点，
             //如果后继结点为null，那么说明没有其他结点了，所以将尾结点也设置为null
            if ( (firstWaiter = first.nextWaiter) == null)
                 lastWaiter = null;
             first.nextWaiter = null;
          //如果被通知节点没有进入到同步队列并且条件等待队列还有不为空的节点，则继续循环通知后续结点
         } while (!transferForSignal(first) &&
                  (first = firstWaiter) != null);
        }

//transferForSignal方法
final boolean transferForSignal(Node node) {
    //尝试设置唤醒结点的waitStatus为0，即初始化状态
    //如果compareAndSetWaitStatus返回false，说明当期结点node的waitStatus已不为
    //CONDITION状态，那么只能是结束状态了，所以返回false
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0)){
         return false;
    }
    //加入同步队列并返回前驱结点p
    Node p = enq(node);
    int ws = p.waitStatus;
    //判断前驱结点是否为结束结点(CANCELLED=1)或者在设置
    //前驱节点状态为Node.SIGNAL状态失败时，唤醒被通知节点代表的线程
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL)){
        //唤醒node结点的线程
        LockSupport.unpark(node.thread);
        return true;
    }
}

复制代码
```



doSignal(first)方法中做了2件事：

一是从条件队列移除被唤醒的节点，然后重新维护条件条件队列的firstWaiter和lastWaiter的指向。

二是将从条件队列移除的结点加入同步队列(在transferForSignal()方法中完成的)，如果进入到同步队列失败并且条件队列还有不为空的节点，则继续循环唤醒后续其他结点的线程。

### 总结：

signal()被调用后，先判断当前线程是否获取锁，如果有，那么唤醒当前Condition对象中条件队列的第一个结点的线程，并从条件队列中移除该结点，移动到同步队列中，如果加入同步队列失败（此时只有可能线程被取消），那么继续循环唤醒条件队列中的其他结点的线程，如果成功加入同步队列，那么如果其前驱结点是否已结束或者设置前驱节点状态为Node.SIGNAL状态失败，则通过LockSupport.unpark()唤醒被通知节点代表的线程，到此signal()任务完成，注意被唤醒后的线程，将从前面的await()方法中的while循环中退出，因为此时该线程的结点已在同步队列中，那么while (!isOnSyncQueue(node))将不在符合循环条件，进而调用AQS的acquireQueued()方法加入获取同步状态的竞争中，这就是等待唤醒机制的整个流程实现原理，流程如下图（注意无论是同步队列还是条件队列使用的Node数据结构都是同一个，不过是使用的内部变量不同罢了）

![img](https://user-gold-cdn.xitu.io/2018/12/8/1678c83d4de93520?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



# 死磕java concurrent包系列（四）基于AQS的条件队列彻底理解ArrayBlockingQueue

# 阻塞队列概览

上篇文章我们分析了AQS中的同步队列和条件队列，而ArrayBlockingQueue和LinkedBlockingQueue正是基于AQS实现的，如果对AQS和ReentrantLock的条件队列不熟悉的话，建议去看https://juejin.im/post/5c053e546fb9a049fc034924，它与我们平时接触的LinkedList和ArrayList相比，最大的特点就是：

- 阻塞添加 当阻塞队列的元素已经满的时，队列会阻塞加入元素的线程（让线程睡一会），等队列不满时再重新唤醒它执行入队操作
- 阻塞移出 阻塞移出是在队列元素为空的时候，删除队列元素的线程会被阻塞，直到队列不为空再执行删除操作 我们先看一下代码，BlockingQueue继承自Queue接口：

```
public interface BlockingQueue<E> extends Queue<E> {


    boolean add(E e); 

    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException; 

    void put(E e) throws InterruptedException; 

    E take() throws InterruptedException; 

    E poll(long timeout, TimeUnit unit) throws InterruptedException; 

    boolean remove(Object o); 
}

    //除了上述方法还有继承自Queue接口的方法 
    //获取但不移除此队列的头元素,没有则跑异常NoSuchElementException 
    E element(); 

    //获取但不移除此队列的头；如果此队列为空，则返回 null。 
    E peek(); 

    //获取并移除此队列的头，如果此队列为空，则返回 null。 
    E poll();

复制代码
```

总结一下：

- 插入方法
  - add(E e) :添加到队列，成功则返回true，失败则抛异常
  - offer(E e):成功返回true,如果队列满则返回false
  - put(E e):将元素添加到队列尾部，如果队列满则一直阻塞直到队列有空位为止
- 删除方法
  - remove(E e) :删除指定元素，成功则返回true，失败则返回false
  - poll(E e):获取并移出队列的头元素，若队列为空，则返回null
  - take(E e):获取并移出队列头元素，若没有元素，则一直阻塞
- 查询方法
  - element():获取但不移除此队列的头元素,没有则跑异常NoSuchElementException
  - peek():获取但不移除此队列的头；如果此队列为空，则返回 null。

这就是阻塞队列基本的增删查方法，接下来我们看一下如何使用它。 #ArrayBlockingQueue阻塞队列的使用方法 再次回到上一篇文章的场景，基于生产者-消费者，生产者产生烤鸡，消费者消费烤鸡，如果使用ArrayBlockingQueue来实现，会比直接通过condition队列实现简单一些：

```
package com.springsingleton.demo.Chicken;

import java.util.concurrent.ArrayBlockingQueue;

public class ArrayBlockingQueueTest {

  //定义吃鸡队列，队列大小是1
  private ArrayBlockingQueue arrayBlockingQueue = new ArrayBlockingQueue(1);

  @SuppressWarnings("unchecked")
  private void product() {
    Chicken chicken = new Chicken();
    try {
      arrayBlockingQueue.put(chicken);
      System.out.println(Thread.currentThread().getName()+" has produced a Chicken");
    }catch (InterruptedException e){
      System.out.println(e.getMessage());
    }
  }

  private void consume(){
    try {
      //每次消费前先睡一秒钟
      Thread.sleep(1000);
      arrayBlockingQueue.take();
      System.out.println(Thread.currentThread().getName()+" has eaten a Chicken");
    }catch (InterruptedException e){
      System.out.println(e.getMessage());
    }
  }

  public static void main(String args[]){
    ArrayBlockingQueueTest arrayBlockingQueueTest = new ArrayBlockingQueueTest();
    new Thread( ()->{
      while (true){
        Thread.currentThread().setName("生产者一号");
        arrayBlockingQueueTest.product();
      }
    }
    ).start();
    new Thread( ()->{
      while (true){
        Thread.currentThread().setName("生产者二号");
        arrayBlockingQueueTest.product();
      }
    }
    ).start();
    new Thread( ()->{
      while (true){
        Thread.currentThread().setName("吃鸡者一号");
        arrayBlockingQueueTest.consume();
      }
    }
    ).start();
    new Thread( ()->{
      while (true){
        Thread.currentThread().setName("吃鸡者二号");
        arrayBlockingQueueTest.consume();
      }
    }
    ).start();
  }
}

复制代码
```

输出如下：

![QQ20181210-212319.gif](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="420" height="191"></svg>)

我们稍微瞥一眼它的构造方法：



```
//默认非公平阻塞队列
ArrayBlockingQueue queue = new ArrayBlockingQueue(666);
//公平阻塞队列
ArrayBlockingQueue queue1 = new ArrayBlockingQueue(666,true);

//构造方法源码
public ArrayBlockingQueue(int capacity) {
     this(capacity, false);
 }

public ArrayBlockingQueue(int capacity, boolean fair) {
     if (capacity <= 0)
         throw new IllegalArgumentException();
     this.items = new Object[capacity];
     lock = new ReentrantLock(fair);
     notEmpty = lock.newCondition();
     notFull =  lock.newCondition();
 }
复制代码
```

通过构造方法发现：它的内部通过一个ReentrantLock和两个条件队列构成，既然是ReentrantLock，那么就有公平和非公平之分了，不懂ReetrantLock的去看上一篇文章：[juejin.im/post/5c021b…](https://juejin.im/post/5c021b59f265da6175737f0b) ArrayBlockingQueue中的元素存在公平访问与非公平访问的区别，对于公平访问队列，被阻塞的线程可以按照阻塞的先后顺序访问队列，即先阻塞的线程先访问队列。而非公平队列，当队列可用时，阻塞的线程将进入争夺访问资源的竞争中，也就是说谁先抢到谁就执行，没有固定的先后顺序。

# ArrayBlockingQueue源码分析

ArrayBlockingQueue的内部是通过一个可重入锁ReentrantLock和两个Condition条件对象来实现阻塞，这里先看看其内部成员变量

```
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {

    /** 存储数据的数组 */
    final Object[] items;

    /**获取数据的索引，主要用于take，poll，peek，remove方法 */
    int takeIndex;

    /**添加数据的索引，主要用于 put, offer, or add 方法*/
    int putIndex;

    /** 队列元素的个数 */
    int count;


    /** 控制并非访问的锁 */
    final ReentrantLock lock;

    /**notEmpty条件对象，用于通知take方法队列已有元素，可执行获取操作 */
    private final Condition notEmpty;

    /**notFull条件对象，用于通知put方法队列未满，可执行添加操作 */
    private final Condition notFull;


}

复制代码
```

ArrayBlockingQueue内部确实是通过数组对象items来存储所有的数据，ArrayBlockingQueue通过**一个ReentrantLock来同时控制添加线程与移除线程的并发访问**，这点与LinkedBlockingQueue区别很大(稍后会分析)。 notEmpty条件队列则是用于存放等待或唤醒调用take方法的线程，告诉他们队列已有元素，可以执行获取操作。 同理notFull条件对象是用于等待或唤醒调用put方法的线程，告诉它们，队列未满，可以执行添加元素的操作。 takeIndex代表的是下一个方法(take，poll，peek，remove)被调用时获取数组元素的索引，putIndex则代表下一个方法（put, offer, or add）被调用时元素添加到数组中的索引。图示如下



![image.png](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1240" height="448"></svg>)



## ArrayBlockingQueue的阻塞添加

我们先来看看非阻塞的情况，也就是之前总结过得add和offer方法，都是非阻塞的添加到队列，只是一个失败返回fase，另一个会抛异常：

```
//add方法实现，内部间接调用了offer(e)
public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }

//offer方法
public boolean offer(E e) {
     //非空校验
     checkNotNull(e);
     final ReentrantLock lock = this.lock;
     //加锁
     lock.lock();
     try {
         //判断队列是否满
         if (count == items.length)
             return false;
         else {
              //入队
             enqueue(e);
             return true;
         }
     } finally {
         lock.unlock();
     }
 }

//入队操作
private void enqueue(E x) {
    //获取当前存放数据的数组
    final Object[] items = this.items;
    //通过putIndex索引对数组进行赋值
    items[putIndex] = x;
    //索引自增，如果已是最后一个位置，重新设置 putIndex = 0;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;//队列中元素数量加1
    //唤醒调用take()方法的线程，执行元素获取操作。
    notEmpty.signal();
}
复制代码
```

源码很简单：其中需要注意的是enqueue(E x)方法，这个方法内部通过putIndex索引直接将元素添加到数组items中，这里可能会疑惑的是当putIndex索引大小等于数组长度时，需要将putIndex重新设置为0，这是因为当前队列执行元素获取时总是从队列头部获取，而添加元素从中从队列尾部获取所以当队列索引（从0开始）与数组长度相等时，下次我们就需要从数组头部开始添加了，如下图演示 ：

- 假设队列总共长度length为5，putindex指向的是最后一个空的array：下标为4

  ![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1240" height="391"></svg>)

  

- 此时元素1被移出：takeindex指向元素2

  ![image.png](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1240" height="384"></svg>)

  

- 此时元素5被加入队列：下标为4的putindex自增后恰好等于队列长度5，那么下一次只能从队列头开始添加元素：

  ![image.png](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1240" height="412"></svg>)

  

接下来我们看看阻塞添加方法put：

```
//put方法，阻塞时可中断
 public void put(E e) throws InterruptedException {
     checkNotNull(e);
      final ReentrantLock lock = this.lock;
      lock.lockInterruptibly();//该方法可中断
      try {
          //当队列元素个数与数组长度相等时，无法添加元素
          while (count == items.length)
              //将当前调用线程挂起，添加到notFull条件队列中等待唤醒
              notFull.await();
          enqueue(e);//如果队列没有满直接添加
      } finally {
          lock.unlock();
      }
  }

复制代码
```

put方法是一个阻塞的方法，如果队列元素已满，那么当前线程将会被notFull条件队列挂起加到条件队列中，直到队列有元素被移出才会唤醒执行添加操作。但如果队列没有满，那么就直接调用enqueue(e)方法将元素加入到数组队列中。

## 总结

三个添加方法即put，offer，add，其中offer，add在正常情况下都是无阻塞的添加，而put方法是阻塞添加。这就是阻塞队列的添加过程。说白了就是当队列满时通过条件对象Condtion来阻塞当前调用put方法的线程，直到线程又再次被唤醒执行。 为了方便理解，总得来说put方法的执行存在以下两种情况：

- 队列已满，那么新到来的put线程将添加到notFull的条件队列中等待

- 有移除线程执行移除操作，移除成功同时唤醒put线程，如下图所示

   

  假设队列全满时：

  ![image.png](https://user-gold-cdn.xitu.io/2018/12/11/1679c745179af2b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**接下来有5个线程通过put方法阻塞入队，他们全部被阻塞，而线程被包装为Node队列存在条件队列中：**

![image.png](https://user-gold-cdn.xitu.io/2018/12/11/1679c7453ea49dcb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**此时元素1被移出了，那么会调用notfull.signal方法，唤醒条件队列的WaitNode，waitNode唤醒后，会调用enqueue()方法入队：**



![image.png](https://user-gold-cdn.xitu.io/2018/12/11/1679c7453f62c334?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## ArrayBlockingQueue的阻塞移出

同样的，我们先看非阻塞的移出，poll和remove。 其中：poll()，获取并删除队列头元素，队列没有数据就返回null，内部通过dequeue()方法删除头元素

```
public E poll() {
      final ReentrantLock lock = this.lock;
       lock.lock();
       try {
           //判断队列是否为null
           return (count == 0) ? null : dequeue();
       } finally {
           lock.unlock();
       }
    }
 //移除队列头元素并返回
 private E dequeue() {
     //拿到当前数组的数据
     final Object[] items = this.items;
      @SuppressWarnings("unchecked")
      //获取要删除的对象
      E x = (E) items[takeIndex];
      将数组中takeIndex索引位置设置为null
      items[takeIndex] = null;
      //takeIndex索引加1并判断是否与数组长度相等，
      //如果相等说明已到尽头，恢复为0
      if (++takeIndex == items.length)
          takeIndex = 0;
      count--;//队列个数减1
      if (itrs != null)
          //同时更新迭代器中的元素数据
          itrs.elementDequeued();
      //移出了元素说明队列有空位，唤醒notFull条件对象添加线程，执行添加操作
      notFull.signal();
      return x;
    }

复制代码
```

总结就是加锁之后获取要删除的对象（**注意，这里的lock和添加时候的lock是同一个lock，意味着同一时间只能添加或者删除，不能并发执行**），之后将数组的takeindex进行处理，并在有空位之后唤醒添加队列的线程执行添加操作，接下来看remove方法：

```
public boolean remove(Object o) {
    if (o == null) return false;
    //获取数组数据
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();//加锁
    try {
        //如果此时队列不为null，这里是为了防止并发情况
        if (count > 0) {
            //获取下一个要添加元素时的索引
            final int putIndex = this.putIndex;
            //获取当前要被删除元素的索引
            int i = takeIndex;
            //执行循环查找要删除的元素
            do {
                //找到要删除的元素
                if (o.equals(items[i])) {
                    removeAt(i);//执行删除
                    return true;//删除成功返回true
                }
                //当前删除索引执行加1后判断是否与数组长度相等
                //若为true，说明索引已到数组尽头，将i设置为0
                if (++i == items.length)
                    i = 0; 
            } while (i != putIndex);//继承查找
        }
        return false;
    } finally {
        lock.unlock();
    }
}

//根据索引删除元素，实际上是把删除索引之后的元素往前移动一个位置
void removeAt(final int removeIndex) {

     final Object[] items = this.items;
      //先判断要删除的元素是否为当前队列头元素
      if (removeIndex == takeIndex) {
          //如果是就简单了：直接删除
          items[takeIndex] = null;
          if (++takeIndex == items.length)
              takeIndex = 0;
          count--;//队列元素减1
          if (itrs != null)
              itrs.elementDequeued();//更新迭代器中的数据
      } else {
      //如果要删除的元素不在队列头部，
      //那么只需循环迭代把删除元素后面的所有元素往前移动一个位置
          //获取下一个要被添加的元素的索引，作为循环判断结束条件
          final int putIndex = this.putIndex;
          //执行循环
          for (int i = removeIndex;;) {
              //获取要删除节点索引的下一个索引
              int next = i + 1;
              //判断是否已为数组长度，如果是从数组头部（索引为0）开始找
              if (next == items.length)
                  next = 0;
               //如果查找的索引不等于要添加元素的索引，说明元素可以再移动
              if (next != putIndex) {
                  items[i] = items[next];//把后一个元素前移覆盖要删除的元
                  i = next;
              } else {
              //在removeIndex索引之后的元素都往前移动完毕后清空最后一个元素
                  items[i] = null;
                  this.putIndex = i;
                  break;//结束循环
              }
          }
          count--;//队列元素减1
          if (itrs != null)
              itrs.removedAt(removeIndex);//更新迭代器数据
      }
      notFull.signal();//唤醒添加线程
    }

复制代码
```

remove(Object o)方法的删除过程相对复杂些，因为该方法并不是直接从队列头部删除元素，而是删除指定的位置。 首先线程先获取锁，再一步判断队列count>0,这点是保证并发情况下删除操作安全执行。接着获取下一个要添加源的索引putIndex以及takeIndex索引 ，作为后续循环的结束判断，因为只要putIndex与takeIndex不相等就说明队列没有结束。然后通过while循环找到要删除的元素索引，执行removeAt(i)方法删除，在removeAt(i)方法中实际上做了两件事：

- 一是如果删除的元素正好在队列头，那么就不需要对后面的数组做任何操作，直接删除，并唤醒添加线程即可
- 二是如果要删除的元素并不是队列头元素，删除之后需要将数组重新reformat一样：从要删除元素的索引removeIndex之后的元素都往前移动一个位置，那么要删除的元素就被removeIndex之后的元素替换，从而也就完成了删除操作。

接着看take()方法，是一个阻塞方法，直接获取队列头元素并删除。

```
//从队列头部删除，队列没有元素就阻塞，可中断
 public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
      lock.lockInterruptibly();//中断
      try {
          //如果队列没有元素
          while (count == 0)
              //执行阻塞操作
              notEmpty.await();
          return dequeue();//如果队列有元素执行删除操作
      } finally {
          lock.unlock();
      }
    }
复制代码
```

take方法其实很简单，有就删除没有就阻塞，注意这个阻塞是可以中断的，如果队列没有数据那么就加入notEmpty条件队列等待(有数据就直接取走，dequeue方法之前分析过了)，如果有新的put线程添加了数据，那么put操作将会唤醒take线程，执行take操作。图示如下 **假设队列全空时：**

![image.png](https://user-gold-cdn.xitu.io/2018/12/11/1679c74543f71bbb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**这个时候有五个线程调用take方法拿元素：**

![image.png](https://user-gold-cdn.xitu.io/2018/12/11/1679c7454439bace?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**这个时候有有一个元素666被put进队列：**

![image.png](https://user-gold-cdn.xitu.io/2018/12/11/1679c745582450e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 总结

ArrayBlockingQueue内部通过一把锁ReentrantLock和两个AQS条件队列实现了阻塞的入队和删除：

- 元素满时，阻塞put线程，封装为node节点在notFull条件队列中，此时如果有线程移出元素，在移出后会唤醒notFull条件队列，让条件队列中的put线程继续尝试进行put
- 元素空时，阻塞take线程，封装为node节点在notEmpty条件队列中，此时如果有线程加入元素，在移出后会唤醒notEmpty条件队列，让条件队列中的take线程继续尝试进行take

# 死磕java concurrent包系列（五）基于AQS的条件队列把LinkedBlockingQueue“扒光”

# LinkedBlockingQueue的基础

LinkedBlockingQueue是一个基于链表的阻塞队列，实际使用上与ArrayBlockingQueue完全一样，我们只需要把之前烤鸡的例子中的Queue对象替换一下即可。如果对于ArrayBlockingQueue不熟悉，可以去看看https://juejin.im/post/5c0f79f3f265da61561f1bec

# LinkedBlockingQueue源码分析

源码在node上注释写明了，它是基于一个“two lock queue”算法实现的，感兴趣的同学可以参考这篇paper：[www.cs.rochester.edu/u/scott/pap…](http://www.cs.rochester.edu/u/scott/papers/1996_PODC_queues.pdf) 这篇文章为了提升在多处理器的机器上的更好性能的并发而提出了这个算法，其中心思想是：通过两把锁分别控制并发，入队时：只需要锁Tail Node，出队时，只需要锁Head Node。 回到LinkedBlockingQueue,先看看内部成员变量：

```
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
    private static final long serialVersionUID = -6903933977591709194L;

    /**
     * Linked list node class
     */
    static class Node<E> {
        E item;

        /**
         * One of:
         * - the real successor Node
         * - this Node, meaning the successor is head.next
         * - null, meaning there is no successor (this is the last node)
         */
        Node<E> next;

        Node(E x) { item = x; }
    }

    /** The capacity bound, or Integer.MAX_VALUE if none */
    private final int capacity;

    /** Current number of elements */
    private final AtomicInteger count = new AtomicInteger();

    /**
     * Head of linked list.
     * Invariant: head.item == null
     */
    transient Node<E> head;

    /**
     * Tail of linked list.
     * Invariant: last.next == null
     */
    private transient Node<E> last;

    /** Lock held by take, poll, etc */
    private final ReentrantLock takeLock = new ReentrantLock();

    /** Wait queue for waiting takes */
    private final Condition notEmpty = takeLock.newCondition();

    /** Lock held by put, offer, etc */
    private final ReentrantLock putLock = new ReentrantLock();

    /** Wait queue for waiting puts */
    private final Condition notFull = putLock.newCondition();
复制代码
```

每个添加到LinkedBlockingQueue队列中的数据都将被封装成Node节点(这个node不同于AQS中的node，它是一个单向链表)，其中head和last分别指向队列的头结点和尾结点。与ArrayBlockingQueue不同的是，LinkedBlockingQueue内部分别使用了takeLock 和 putLock 对并发进行控制，也就是说，**添加和删除操作并不是互斥操作**，可以同时进行，这样也就可以大大提高吞吐量。这里再次强调如果没有给LinkedBlockingQueue指定容量大小，其默认值将是Integer.MAX_VALUE，如果存在添加速度大于删除速度时候，有可能会内存溢出，这点在使用前希望慎重考虑。至于LinkedBlockingQueue的实现原理图与ArrayBlockingQueue是类似的，除了对添加和移除方法使用单独的锁控制外，两者都使用了不同的Condition条件对象作为等待队列，用于挂起take线程和put线程。 总结如下图：

![image.png](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1240" height="235"></svg>)



## LinkedBlockingQueue的阻塞添加

同样的，添加的方法主要有：add offer 和put。我们先看看非阻塞添加的add和offer方法，这两个方法的区别同样是添加失败时，add方法是抛异常，offer方法是返回false

```
public boolean add(E e) {
     if (offer(e))
         return true;
     else
         throw new IllegalStateException("Queue full");
}
复制代码
public boolean offer(E e) {
        if (e == null) throw new NullPointerException();
        //因为存在并发操作移出和入队互不冲突，与arrayBlockingQueue不同，count被声明为Atomic
        final AtomicInteger count = this.count;
        //队列满了直接返回
        if (count.get() == capacity)
            return false;
        int c = -1;
        Node<E> node = new Node<E>(e);
        final ReentrantLock putLock = this.putLock;
        putLock.lock();
        try {
            //因为存在并发问题，加锁之后再次判断一下队列有没有满
            if (count.get() < capacity) {
                //入队
                enqueue(node);
                //容量+1返回旧值
                c = count.getAndIncrement();
                //因为在入队时可能同时有出队的线程同时把元素移除，所以在入队后做一个补偿，
                //如果队列还有空间，那么唤醒一个如归的线程执行添加操作
                if (c + 1 < capacity)
                    notFull.signal();
            }
        } finally {
            putLock.unlock();
        }
        //c==0，只有可能最开始就是一个空队列（注意上面的c返回的是旧值）此时因为刚好添加了一个元素，
        //所以唤醒消费的线程去取移出元素
        if (c == 0)
            signalNotEmpty();
        return c >= 0;
    }
复制代码
//入队操作
private void enqueue(Node<E> node) {
     //队列尾节点指向新的node节点
     last = last.next = node;
}

//signalNotEmpty方法去唤醒移出元素的线程,为什么要先获取锁才能signal呢？不懂的同学回去看看AQS：
//因为条件队列是基于AQS的锁存在的，用法上必须要这么用，否则会抛出异常
private void signalNotEmpty() {
      final ReentrantLock takeLock = this.takeLock;
      takeLock.lock();
          //唤醒获取并删除元素的线程
          notEmpty.signal();
      } finally {
          takeLock.unlock();
      }
  }

复制代码
```

这里的Offer()方法做了两件事:

- 第一件事是判断队列是否满，满了就直接释放锁，没满就将节点封装成Node入队，然后加锁后再次判断队列添加完成后是否已满，不满就继续唤醒等到在条件对象notFull上的添加线程。
- 第二件事是，判断是否需要唤醒等到在notEmpty条件对象上的消费线程。

接下来看看put方法，与offer方法如出一辙：

```
 public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        int c = -1;
        Node<E> node = new Node<E>(e);
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        //锁可被中断
        putLock.lockInterruptibly();
        try {
          //队列满时加入notFull条件队列
            while (count.get() == capacity) {
                notFull.await();
            }
            enqueue(node);
            c = count.getAndIncrement();
            //队列还没有满时，继续唤醒添加线程
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        //c==0，只有可能最开始就是一个空队列（注意上面的c返回的是旧值）此时因为刚好添加了一个元素，
        //所以唤醒消费的线程去取移出元素
        if (c == 0)
            signalNotEmpty();
    }

复制代码
```

这里有几个问题：

**问题1：**
为什么添加完成后是继续唤醒在条件队列notFull上的添加线程而不是像ArrayBlockingQueue那样直接唤醒notEmpty条件对象上的消费线程?

**分析1：** 先回想一下ArrayBlockingQueue：它内部只有一个锁，在内部完成添加元素操作后直接唤醒消费线程去消费。如果ArrayBlockingQueue在添加元素之后再唤醒添加线程的话，消费的线程就可能一直被block，无法执行。 而为了避免这种情况，对于LinkedBlockingQueue来说，他有两个锁，添加和删除元素不是互斥的，添加的过程中可能已经删除好几个元素了，所以他在设计上要尽可能的去唤醒两个条件队列。 **添加线程在队列没有满时自己直接唤醒自己的其他添加线程，如果没有等待的添加线程，直接结束了。如果有就直到队列元素已满才结束挂起。注意消费线程的执行过程也是如此。这也是为什么LinkedBlockingQueue的吞吐量要相对大些的原因。**

**问题2：** 为什么if (c == 0)时才去唤醒消费线程呢

**分析2：** 什么情况下c等于0呢？c值是添加元素前队列的大小，也就是说，之前是空队列，空队列时会有什么情况呢，空队列会阻塞所有的take进程，将其封装到notEmpty的条件队列中。这个时候，c之前是0，现在在执行了enqueue方法后，队列中有元素了，所以他需要立即唤醒阻塞的take进程，否则阻塞的take进程就一直block在队列里，一直沉睡下去。 为什么c>0时，就不会唤醒呢？因为take方法和put方法一样，take方法每次take完元素后，如果队列还有值，它会继续唤醒take队列，也就是说他只要没有被await()阻塞，他就会一直不断的唤醒take线程，而不需要再添加的时候再去唤醒，造成不必要的性能浪费

## LinkedBlockingQueue的阻塞移出

相对的，我们再看看take方法：

```
public E take() throws InterruptedException {
        E x;
        int c = -1;
        //获取当前队列大小
        final AtomicInteger count = this.count;
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lockInterruptibly();//可中断
        try {
            //如果队列没有数据，当前take线程到条件队列中
            while (count.get() == 0) {
                notEmpty.await();
            }
            //如果存在数据直接删除并返回该数据
            x = dequeue();
            c = count.getAndDecrement();//队列大小减1，返回之前的值
            if (c > 1)
                notEmpty.signal();//还有数据就唤醒后续的消费线程
        } finally {
            takeLock.unlock();
        }
        //满足条件（之前队列是满的，现在刚刚执行dequeue拿出了一个），
        //唤醒条件对象上等待队列中的添加线程
        if (c == capacity)
            signalNotFull();
        return x;
    }

private E dequeue() {
        Node<E> h = head;//获取头结点
        Node<E> first = h.next; //获取头结的下一个节点（要删除的节点）
        h.next = h; // help GC//自己next指向自己，即被删除
        head = first;//更新头结点
        E x = first.item;//获取删除节点的值
        first.item = null;//清空数据，因为first变成头结点是不能带数据的，这样也就删除队列的带数据的第一个节点
        return x;
    }


复制代码
```

take方法是一个可阻塞可中断的移除方法，主要做了两件事：

- 如果队列没有数据就挂起当前线程到 notEmpty条件对象的等待队列中一直等待，如果有数据就删除节点并返回数据项，同时唤醒后续消费线程；
- 尝试唤醒条件对象notFull上等待队列中的添加线程：假设之前队列中满员了，那么新来的put进程将会被阻塞进notFull条件队列，然后await挂起沉睡。这个时候有线程通过take方法拿出了一个元素，如果此时不唤醒notFull条件队列，那么之前满员时队列中的线程就会一直睡死过去

## 总结

LinkedBlockingQueue的两个队列：

- notFull条件队列（队列满时阻塞的put线程）： await的时机：队列满了 signal的时机：一是put方法放入元素后，如果队列还有空位，会singal线程继续添加；二是如果队列最开始满员，take方法移出了一个元素后，队列还有一个空位时也会唤醒它。
- notEmpty条件队列（队列空时候阻塞的take线程）： await的时机：队列空了 signal的时机：一是take方法移出元素后，如果队列还有空位，会singal线程继续移出；二是如果队列最开始空的，put方法放入了一个元素后，队列还有一个元素时也会唤醒它。

这种算法就是“two lock queue”的设计思想，这也是LinkedBlockingQueue的吞吐量较高的本质原因

# ArrayBlockingQueue和LinkedBlockingQueue的比较总结

通过上述的分析，对于LinkedBlockingQueue和ArrayBlockingQueue的基本使用以及内部实现原理我们已较为熟悉了，这里我们就对它们两间的区别来个小结

1.队列大小和构造方法有所不同，ArrayBlockingQueue是有界的初始化必须指定大小，而LinkedBlockingQueue可以是有界的也可以是无界的(Integer.MAX_VALUE)，对于后者而言，当添加速度大于移除速度时，在无界的情况下，可能会造成内存溢出等问题，有坑。

2.数据存储容器不同，ArrayBlockingQueue采用的是数组作为数据存储容器，而LinkedBlockingQueue采用的则是以Node节点作为连接对象的单向链表。

3.从GC的角度分析：由于ArrayBlockingQueue采用的是数组的存储容器，因此在插入或删除元素时不会产生或销毁任何额外的对象实例，而LinkedBlockingQueue则会生成一个额外的Node对象。这可能在长时间内需要高效并发地处理大批量数据的时，对于GC可能存在较大影响。

4.两者的实现队列添加或移除的锁不一样，ArrayBlockingQueue实现的队列中的锁是没有分离的，即添加操作和移除操作采用的同一个ReenterLock锁，而LinkedBlockingQueue实现的队列中的锁是分离的，其添加采用的是putLock，移除采用的则是takeLock，这样能大大提高队列的吞吐量，也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。

# 死磕java concurrent包系列（六）基于AQS解析信号量Semaphore

# Semaphore

之前分析AQS的时候，内部有两种模式，独占模式和共享模式，前面的ReentrantLock都是使用独占模式，而Semaphore同样作为一个基于AQS实现的并发组件，它是基于共享模式实现的，我们先看看它的使用场景

## Semaphore共享锁的基本使用

假设有20个人去银行柜面办理业务，银行只有3个柜面，同时只能办理三个人，如果基于这种有限的、我们需要控制资源的情况，使用Semaphore比较方便：

```
public class SemaphoreTest {
  //排队总人数
  private static final int COUNT =20;
  //只有三个柜台
  private static final Semaphore AVALIABLECOUNT = new Semaphore(3);

  public static void main(String[] args) {
    //创建一个线程池
    BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(COUNT);
    BasicThreadFactory.Builder builder = new BasicThreadFactory.Builder().namingPattern("线程池");
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(COUNT, COUNT, 30L, TimeUnit.SECONDS, workQueue,
        builder.build());
    for (int i = 0; i < COUNT; i++) {
      final int count = i;
      //排队的人都需要被服务，所以所有的人直接提交线程池处理
      threadPoolExecutor.execute(() -> {
        try {
          //使用acquire获取共享锁
          AVALIABLECOUNT.acquire();
          System.out.println(Thread.currentThread().getName());
          System.out.println("服务号"+count+"正在服务");
          Thread.sleep(1000);
        }catch (Exception e){
          System.out.println(e.getMessage());
        }
        finally {
          //获取完了之后释放资源
          AVALIABLECOUNT.release();
        }
      });
    }
    threadPoolExecutor.shutdown();
  }
}
复制代码
```

输出如下：我们执行代码，可以发现每隔1秒几乎同一时间出现3条线程访，如下图

![QQ20181212-142037.gif](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="420" height="216"></svg>)



# Semaphore内部原理解析

## Semaphore的内部结构

在深入分析Semaphore的内部原理前先看看一张类图结构

![image.png](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="554" height="277"></svg>)

这个结构和ReentrantLock基本上完全一致，Semaphore内部同样存在继承自AQS的内部类Sync以及继承自Sync的公平锁(FairSync)和非公平锁(NofairSync),从这点也足以说明Semaphore的内部实现原理也是基于AQS并发组件的。 在之前的文章中，我们提到过，AQS是基础组件，只负责核心并发操作，如加入或维护同步队列，控制同步状态等，而具体的加锁和解锁操作交由子类完成，因此子类Semaphore共享锁的获取与释放需要自己实现，这两个方法分别是获取锁的tryAcquireShared(int arg)方法和释放锁的tryReleaseShared(int arg)方法，这点从Semaphore的内部结构完全可以看出来。 我们在调用Semaphore的方法时，其内部则是通过间接调用其内部类或AQS执行的。下面我们就从Semaphore的源码入手分析共享锁实现原理，这里先从非公平锁入手。



## 非公平锁的共享锁

同样的，我们先看看构造方法：

```
 public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    /**
     * Creates a {@code Semaphore} with the given number of
     * permits and the given fairness setting.
     *
     * @param permits the initial number of permits available.
     *        This value may be negative, in which case releases
     *        must occur before any acquires will be granted.
     * @param fair {@code true} if this semaphore will guarantee
     *        first-in first-out granting of permits under contention,
     *        else {@code false}
     */
    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
复制代码
```

我们通过默认构造函数创建时，诞生的就是非公平锁，接下来我们看一下构造方法的入参permits的传递：

```
static final class NonfairSync extends Sync {
    NonfairSync(int permits) {
          super(permits);
    }
   //调用父类Sync的nonfairTryAcquireShared
   protected int tryAcquireShared(int acquires) {
       return nonfairTryAcquireShared(acquires);
   }
}

复制代码
```

在Sync中：

```
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;
        //直接将该值设置为AQS中的state的值
        Sync(int permits) {
            setState(permits);
        }
复制代码
```

所以Semaphore的入参permit直接传入设置到AQS中的state中。 接下来我们看看acquire()方法，我们先通俗的解释一下它的执行过程： 当一个线程请求到来时，state值代表的许可数，那么请求线程将会获得同步状态即对共享资源的访问权，并更新state的值(一般是对state值减1)，但如果请求线程过多，state值代表的许可数已减为0，则请求线程将无法获取同步状态，线程将被加入到同步队列并阻塞，直到其他线程释放同步状态(一般是对state值加1)才可能获取对共享资源的访问权。 调用Semaphore的acquire()方法后将会调用到AQS的acquireSharedInterruptibly()：

```
    //Semaphore的acquire()
    public void acquire() throws InterruptedException {
      sync.acquireSharedInterruptibly(1);
    }
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        //判断是否被中断
        if (Thread.interrupted())
            throw new InterruptedException();
        //如果tryAcquireShared(arg)不小于0，则说明当前还有permit可被使用
        if (tryAcquireShared(arg) < 0)
            //如果许可被用完了，没有剩余许可 则加入同步队列等待
            doAcquireSharedInterruptibly(arg);
    }
复制代码
```

在acquireSharedInterruptibly()方法内部先进行了线程中断的判断，那么先尝试调用tryAcquireShared(arg)方法获取同步状态，如果此时许可获取成功，那么方法执行结束，如果获取失败，则说明没有剩余许可了，那么调用doAcquireSharedInterruptibly(arg);方法加入同步队列等待。 这里的tryAcquireShared(arg)是个模板方法设计模式，AQS内部没有提供具体实现，由子类实现，也就是有Semaphore内部自己实现，该方法在Semaphore内部非公平锁的实现如下

```
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                //remaining < 0说明许可已经供不应求了，这个时候进来的线程需要被阻塞
                //否则CAS操作更新avaliable的值，它表示剩余的许可数
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
复制代码
```

nonfairTryAcquireShared(int acquires)方法内部，先获取state的值，并执行减法操作，得到remaining值，它可以理解为剩余的许可数，如果remaining<0，说明请求的许可数过大，此时直接返回一个负数的remaining；如果remaining大于0，说明还有剩余的许可数，则可以访问共享资源，后续将被加入同步队列(通过doAcquireSharedInterruptibly(arg))。 注意Semaphore的acquire()可能存在并发操作，因此nonfairTryAcquireShared()方法体内部采用死循环+无锁(CAS)并发的操作保证对state值修改的安全性。 例如：假设permit值为5，有多个线程并发accquire获取许可，线程1运行时得到的remainin是5-1=4，线程2运行时，得到的remaining同样是5-1=4，但是执行compareAndSetState时，线程2 更快一点，执行CAS操作：判断state现在是否为5，如果为5，则CAS更新为4. 这个时候线程1也执行CAS操作，判断state现在是否为5，发现不为5，所以CAS失败，这时候需要这个死循环去重试。

如果remaining大于0，说明还有剩余的许可数，则可以访问共享资源，后续将被加入同步队列，接下来看入队的操作，这一部分与ReentrantLock差不多：

```
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        //使用SHARED类型创建共享模式的Node
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                //获取前序节点
                final Node p = node.predecessor();
                //如果前序节点是头节点，说明自己的Node在队列最前端，此时可能共享资源随时被释放
                //所以需要再次尝试获取共享资源
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    //如果获取共享资源成功
                    if (r >= 0) {
                        //已经获取资源后，node已经没有意义，所以清理head节点并传播
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                //如果不是头节点
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
复制代码
```

在方法中，由于当前线程没有获取同步状态，因此创建一个共享模式类型（Node.SHARED）的结点并通过addWaiter(Node.SHARED)加入同步队列，加入完成后，当前线程进入自旋状态，首先判断前驱结点是否为head，如果是，那么尝试获取同步状态并返回r值，如果r大于0，则说明获取同步状态成功，将当前线程设置为head并传播，传播指的是，通知后续结点继续获取同步状态，到此return结束，获取到同步状态的线程将会执行原定的任务。

```
private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; 
        setHead(node);//设置为头结点
        /* 
         * 尝试去唤醒队列中的下一个节点，如果满足如下条件： 
         * 还有剩余许可(propagate > 0), 
         * 或者h.waitStatus为PROPAGATE(被上一个操作设置) 
         * 并且 
         *   下一个节点处于共享模式或者为null。 
         * 
         * 这两项检查中的保守主义可能会导致不必要的唤醒，但只有在有
         * 有在多个线程争取获得/释放同步状态时才会发生，所以大多
         * 数情况下会立马获得需要的信号
         */  
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
            //唤醒后继节点，因为是共享模式，所以允许多个线程同时获取同步状态
                doReleaseShared();
        }
    }

复制代码
```

但如果前驱结点不为head或前驱结点为head并尝试获取同步状态失败（与），那么调用shouldParkAfterFailedAcquire(p, node)方法判断前驱结点的waitStatus值是否为SIGNAL并调整同步队列中的node结点状态，如果返回true，那么执行parkAndCheckInterrupt()方法，将当前线程挂起。 shouldParkAfterFailedAcquire方法与ReentrantLock中的如出一辙：

```
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //获取当前结点的等待状态
        int ws = pred.waitStatus;
        //如果为等待唤醒（SIGNAL）状态则返回true
        if (ws == Node.SIGNAL)
            return true;
        //如果ws>0 则说明是结束状态，
        //遍历前驱结点直到找到没有结束状态的结点
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //如果ws小于0又不是SIGNAL状态，说明是node是首次加入的线程
            //则将其前驱节点设置为SIGNAL状态。下次执行shouldParkAfterFailedAcquire方法时就
            //满足ws == Node.SIGNAL条件了
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

复制代码
```

这个方法是AQS中的，如果不懂的话，可以参考之前在ReentrantLock中也分析过：[juejin.im/post/5c021b…](https://juejin.im/post/5c021b59f265da6175737f0b) 中自旋的部分。 到此，加入同步队列的整个过程完成。

## 总结

在AQS中存在一个volatile变量state，当我们创建Semaphore对象传入许可数值时，最终会赋值给state，state的数值代表可同时操作共享数据的线程数量，每当一个线程请求(如调用Semaphored的acquire()方法)获取同步状态成功，state的值将会减少1，直到state为0时，表示已没有可用的许可数，也就是对共享数据进行操作的线程数已达到最大值，其他后来线程将被阻塞，此时AQS内部会将线程封装成共享模式的Node结点，加入同步队列中等待并开启自旋操作。只有当持有对共享数据访问权限的线程执行完成任务并释放同步状态后，同步队列中的对于的结点线程才有可能获取同步状态并被唤醒执行同步操作，注意在同步队列中获取到同步状态的结点将被设置成head并清空相关线程数据(毕竟线程已在执行也就没有必要保存信息了)，AQS通过这种方式便实现共享锁，用图表示如下：

![1544670486895.jpg](https://user-gold-cdn.xitu.io/2018/12/13/167a64255da703a4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



\##非公平锁的释放锁 接下来看一下释放锁：

```
public void release() {
       sync.releaseShared(1);
}

//调用到AQS中的releaseShared(int arg) 
public final boolean releaseShared(int arg) {
       //调用子类Semaphore实现的tryReleaseShared方法尝试释放同步状态
      if (tryReleaseShared(arg)) {
          doReleaseShared();
          return true;
      }
      return false;
  }
复制代码
```

显然Semaphore间接调用了AQS中的releaseShared(int arg)方法，通过tryReleaseShared(arg)方法尝试释放同步状态，如果释放成功，那么将调用doReleaseShared()唤醒同步队列中后继结点的线程，tryReleaseShared(int releases)方法如下：

```
//在Semaphore的内部类Sync中实现的
protected final boolean tryReleaseShared(int releases) {
       for (;;) {
              //获取当前state
             int current = getState();
             //释放状态state增加releases
             int next = current + releases;
             if (next < current) // overflow
                 throw new Error("Maximum permit count exceeded");
              //通过CAS更新state的值
             if (compareAndSetState(current, next))
                 return true;
         }
        }
复制代码
```

逻辑很简单，释放同步状态，更新state的值，同样的，通过for死循环和CAS操作来保证线程安全问题，因为可能存在多个线程同时释放同步状态的场景。释放成功后通过doReleaseShared()方法唤醒后继结点。

```
private void doReleaseShared() {
    /* 
     * 如果头节点的后继节点需要唤醒，那么执行唤醒  
     * 动作；如果不需要，将头结点的等待状态设置为PROPAGATE保证   
     * 唤醒传递。另外，为了防止过程中有新节点进入(队列)，这里必  
     * 需做循环，所以，和其他unparkSuccessor方法使用方式不一样  
     * 的是，如果(头结点)等待状态设置失败，重新检测。 
     */  
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            // 获取头节点对应的线程的状态
            int ws = h.waitStatus;
            // 如果头节点对应的线程是SIGNAL状态，则意味着头
            //结点的后继结点所对应的线程需要被unpark唤醒。
            if (ws == Node.SIGNAL) {
                // 修改头结点对应的线程状态设置为0。失败的话，则继续循环。
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                // 唤醒头结点h的后继结点所对应的线程
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        // 如果头结点发生变化，则继续循环。否则，退出循环。
        if (h == head)                   // loop if head changed
            break;
    }
}


//唤醒传入结点的后继结点对应的线程
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
      if (ws < 0)
          compareAndSetWaitStatus(node, ws, 0);
       //拿到后继结点
      Node s = node.next;
      if (s == null || s.waitStatus > 0) {
          s = null;
          for (Node t = tail; t != null && t != node; t = t.prev)
              if (t.waitStatus <= 0)
                  s = t;
      }
      if (s != null)
          //唤醒该线程
          LockSupport.unpark(s.thread);
    }

复制代码
```

显然doReleaseShared()方法中通过调用unparkSuccessor(h)方法唤醒head的后继结点对应的线程。这个方法在之前获取资源时也会被调用：

```
 if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }

复制代码
```

两种情况下都是为唤醒后继节点，因为是共享模式，所以允许多个线程同时获取同步状态。释放操作的过程还是相对简单些的，即尝试更新state值，更新成功调用doReleaseShared()方法唤醒后继结点对应的线程。

## 公平锁的共享锁

公平锁的中的共享模式实现除了在获取同步状态时与非公平锁不同外，其他基本一样：

```
static final class FairSync extends Sync {
        FairSync(int permits) {
            super(permits);
        }

        protected int tryAcquireShared(int acquires) {
            for (;;) {
                //这里是重点，先判断队列中是否有结点再执行
                //同步状态获取。
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
    }

相比之下,对于非公平锁：
    final int nonfairTryAcquireShared(int acquires) {
         //使用死循环
         for (;;) {
             //每当有线程获取共享资源时，就直接尝试CAS操作
             int available = getState();
             int remaining = available - acquires;
             //判断信号量是否已小于0或者CAS执行是否成功
             if (remaining < 0 ||
                 compareAndSetState(available, remaining))
                 return remaining;
         }
     }

复制代码
```

从代码中可以看出，与非公平锁tryAcquireShared(int acquires)方法实现的唯一不同是，在尝试获取同步状态前，先调用了hasQueuedPredecessors()方法判断同步队列中是否存在结点，如果存在则返回-1，即将线程加入同步队列等待，后续通过Node结构保证唤醒的顺序。从而保证先到来的线程请求一定会先执行，也就是所谓的公平锁。其他操作，与前面分析的非公平锁一样。

## 总结

AQS作为核心并发组件，它通过state值来控制对共享资源访问的线程数，内部的Node有独占模式（EXCLUSIVE）和共享模式（SHARED）：

- 对于ReenTrantLock：state默认为0，每次加锁后state更新为1，更新为1之后如果还有线程尝试获取锁，则加入同步队列等待；每当线程释放锁时，再更新为0并唤醒队列中的线程
- 对于Semaphore：State默认为许可数，每当线程请求同步状态成功，state值将会减1，如果超过限制数量的线程将被封装共享模式的Node结点加入同步队列封装成独占模式（EXCLUSIVE）等待，直到其他执行线程释放同步状态，才有机会获得执行权，而每个线程执行完成任务释放同步状态后，state值将会增加1，这就是共享锁的基本实现模型。

AQS是采用模板方法的设计模式构建的，它作为基础组件，封装的是核心并发操作，但是实现上分为两种模式，即共享模式(如Semaphore)与独占模式（如ReetrantLock，这两个模式的本质区别在于多个线程能不能共享一把锁），而这两种模式的加锁与解锁实现方式是不一样的，但AQS只关注内部公共方法实现并不关心外部不同模式的实现，所以提供了模板方法给子类使用：也就是说实现独占锁，如ReentrantLock需要自己实现tryAcquire()方法和tryRelease()方法，而实现共享模式的Semaphore，则需要实现tryAcquireShared()方法和tryReleaseShared()方法，这样做的好处是显而易见的，无论是共享模式还是独占模式，其基础的实现都是同一套组件(AQS)，只不过是加锁解锁的逻辑不同罢了，更重要的是如果我们需要自定义锁的话，也变得非常简单，只需要选择不同的模式实现不同的加锁和解锁的模板方法即可。 不管是ReentrantLock还是Semaphore，公平锁与非公平锁的不同之处在于公平锁会在线程请求同步状态前，判断同步队列是否存在Node，如果存在就将请求线程封装成Node结点加入同步队列，从而保证每个线程获取同步状态都是先到先得的顺序执行的。非公平锁则是通过竞争的方式获取，不管同步队列是否存在Node结点，只有通过竞争获取就可以获取线程执行权。