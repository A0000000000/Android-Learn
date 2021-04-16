# 关于ActivityThread类
* 前言: 此类是一个普通的Java类, 它具有main函数, 但它本身不是一个Thread

## ActivityThread.main

### Tips:
1. ActivityThread的main函数是整个Android App的入口, 即执行main函数的线程是Android的主线程
2. ActivityThread本身不是一个线程类, 而这个类的对象是在主线程创建的

### 源码
```java
public class ActivityThread {
    ....
    public static main(String[] args) {
        ...
        Looper.prepareMainLooper(); // #1
        ...
        ActivityThread thread = new ActivityThread(); // #2
        thread.attach(false, startSeq); // #3
        ...
        sMainThreadHandler = thread.getHandler(); // #4
        ...
        Looper.loop(); // #5
        ...
    }
    ....
}
```
* ActivityThread的main函数中 主要是如上5行代码
1. Looper.prepareMainLooper(): 此行代码主要是初始化主线程的Looper, 内容如下
    ```java
    public static void prepareMainLooper() {
            prepare(false);
            synchronized (Looper.class) {
                if (sMainLooper != null) {
                    throw new IllegalStateException("The main Looper has already been prepared.");
                }
                sMainLooper = myLooper();
            }
        }
    ```
    * prepare(false), 是在调用类中的另一个静态方法来初始化Looper, 参数是标志这该Looper是否可以退出, 对于主线程的Looper, 我们是不应该退出的, 主线程的Looper退出意味着App被关闭, 所以这里设置为false. 
        1. prepare的实现很简单, 首先判断Looper是否已经实例化过, 如果实例化过, 则抛出异常, 也就是说每个线程至多同时只能有一个Looper, 如果没有实例化过, 就调用构造函数来创建一个新的Looper, 并将其加入到线程局部变量中保存
        2. 为什么这么做?  (猜测) 
            1. 每个线程都只能有一个Looper,也就意味着每个线程都只能操作本线程的Looper, 通过一些静态方法来代理访问Looper的实例方法避免了其他线程访问本线程的Looper的可能. 
            2. 为了方便某个线程随时随地得到本线程的Looper. 
        3. 有关于Looper的构造函数,  在这个构造函数中有两个操作.
            1. 实例化MessageQueue
            2. 记录一下当前线程
        4. 关于MessageQueue, 它最后是调用的一个JNI函数来进行初始化的, 所以这里就不在继续往下深挖了.
        ```java
            private static void prepare(boolean quitAllowed) {
                if (sThreadLocal.get() != null) {
                    throw new RuntimeException("Only one Looper may be created per thread");
                }
                sThreadLocal.set(new Looper(quitAllowed));
            }

            private Looper(boolean quitAllowed) {
                mQueue = new MessageQueue(quitAllowed);
                mThread = Thread.currentThread();
            }

            MessageQueue(boolean quitAllowed) {
                mQuitAllowed = quitAllowed;
                mPtr = nativeInit();
            }

            private native static long nativeInit();

        ```
* 以上就是Looper初始化的过程

2. ActivityThread thread = new ActivityThread(): 此行代码是在调用ActivityThread无参构造函数来实例化ActivityThread的对象
    * 构造函数主要是在获取ResourcesManager的实现类
    * ResourcesManager是一个单例类, 它是通过调用无参构造函数来实例化的
    * ResourcesManager的构造函数是一个空实现, 所以也没啥好说的
    ```java
    ActivityThread() {
        mResourcesManager = ResourcesManager.getInstance();
    }

    public static ResourcesManager getInstance() {
        synchronized (ResourcesManager.class) {
            if (sResourcesManager == null) {
                sResourcesManager = new ResourcesManager();
            }
            return sResourcesManager;
        }
    }

    public ResourcesManager() {
    }
    ```
3. thread.attach(false, startSeq): 此方法主要是向ActivityManagerService注册ActivityThread, 因为再往下的代码AndroidStudio看不到了, 所以具体怎么注册的, 这里也不再多说.
    * 为什么要向AMS注册ActivityThread?(猜测)
        1. 为了让Android管理App的调度
        2. 为实现四大组件的生命周期函数的调用(向组件发出信号)
    ```java
    private void attach(boolean system, long startSeq) {
        ...
        final IActivityManager mgr = ActivityManager.getService();
        ...
        mgr.attachApplication(mAppThread, startSeq);
        ...
    }
    ```
4. sMainThreadHandler = thread.getHandler(): 获取主线程的Handler
    * 此处并不是创建Handler对象,  Handler是ActivityThread的一个实例字段, 即在创建ActivityThread对象时, Handler对象也一并创建完成.
    * 此处的Handler是一个字类,  它在这个类中定义了许多现有的消息, 用来完成指定的操作.
    ```java
    class H extends Handler {
            public static final int BIND_APPLICATION        = 110;
            ...
            String codeToString(int code) {
              ...
            }
            public void handleMessage(Message msg) {
              ...
            }
        }

    ```
    * String codeToString(int) 是一个额外扩展的方法, 他在调试模式下获得静态字段的名字, 在非调试模式下获得静态字段的值
    * void handleMessage(Message msg) 即是重载的父类的处理方法, 用来实现处理特殊消息的操作.
5. Looper.loop(): 开启主循环, 此处调用Looper的静态方法来开启循环, 目的是保证该循环开启时, 一定是调用线程来执行
    * loop方法中有一个死循环, 在这个循环中, 它会不断的取消息(这个操作可能阻塞), 然后将消息分发给Handler执行, 主要代码如下
    ```java
    public static void loop() {
        ...
        final Looper me = myLooper();
        ...
        final MessageQueue queue = me.mQueue;
        ...
        for (;;) {
            Message msg = queue.next(); // 可能会阻塞, 如果队列里没有消息
            ...
            msg.target.dispatchMessage(msg);// target就是产生消息的Handler
            ...
        }
    }
    ```
## 参考
* [百度到的博客](https://blog.csdn.net/hzwailll/article/details/85339714)