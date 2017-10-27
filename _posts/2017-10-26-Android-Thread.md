---
layout: post
title:  "Android的线程和线程池"
date:   2017-10-26 19:50:05
categories: Android
excerpt: Android开发实践笔记。
---

* content
{:toc}

Android上的线程分为主线程和子线程，主线程主要处理和界面相关的事情，子线程往往用于执行耗时操作。除了Thread之外，Android可以扮演线程角色的还有AsyncTask, IntentService, HandlerThread。不同的形式的线程具有不同的特性和使用场景。

在操作系统中，线程是操作系统调度的最小单元，也不可能无限制地产生，其线程的创建和销毁都会有相应的开销。通过在线程池中缓存一定数量的线程就可以避免频繁创建和销毁线程带来的系统开销。

---
### AyncTask

AsyncTask：允许在后台执行一个异步任务。可以将耗时的操作放在异步任务当中来执行，并随时将任务执行的结果返回给UI线程作更新。通过AsyncTask可以解决多线程之间的通信问题。

AsyncTask封装了Thread和Handler，就相当于Android提供了一个多线程编程的一个框架。如果要定义一个AsyncTask，就需要定义一个类来继承AsyncTask这个抽象类，并实现其唯一的一个 doInBackgroud 抽象方法。

	


#### Looper源码分析
Looper主要有prepare()和loop()两个方法。

	private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
	
	private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

在构造函数中创建了一个MessageQueue(消息队列)。在prepare()中将一个Looper实例放入到ThreadLocal中。首先判断sThreadLocal是否为null,否则抛出异常。这就保证prepare()方法不能被调用两次，同时保证一个线程中只有一个Looper实例。

ThreadLocal不是线程，主要作用是在每个线程中互不干扰地存储数据。

	public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            if (traceTag != 0) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }

msg.target.dispatchMessage(msg); 把msg交给msg的target的dispatchMessage方法去处理。其实就是Handler对象。	


### HandlerThread

Android UI是线程不安全的，如果在子线程中进行UI操作，程序就可能崩溃。通过创建一个Message对象，借助Handler发送出去，在Handler的handleMessage()方法中获得刚才发送的Message对象，然后再对UI进行操作。  

---

#### Handler源码分析
首先观察下Handler的构造函数，然后再了解下sendMessage()方法。

	public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

通过Looper.myLooper()获取了当前线程保存的Looper实例，然后又获取了这个Looper实例中保存的MessageQueue。

	public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
	
	private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

此方法内部直接获取MessageQueue,然后调用enqueueMessage()方法，也就是将当前的handler作为msg的target属性。

	public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

	public void handleMessage(Message msg) {
    }

handleMessage()是个空方法，消息的最终回调都需要我们自己控制。

#### Handler post
会传递一个Runnable对象到消息队列中，并在这个Runnable对象中，重写run()方法，一般在这个run()方法中写入需要在UI线程中的操作。

	Handler handler = new Handler();
	Runnable update = new Runnable() {
		public void run() {
            int currentPage = mViewPager.getCurrentItem();            
            mViewPager.setCurrentItem(currentPage + 1, true);
        }
	}
	handler.post(update);

对于post方式,它其中的Runnable对象的run()方法中的代码均运行在主线程上，所以不能在UI线程上的操作，一样不能使用post方式执行，比如网络操作。

Runnable对象被放到了消息队列当中，然后主线程中的Looper(因为Handler是在主线程中创建的，所以Looper也在主线程中)将这个Runnable对象从消息队列中取出。

	public final boolean post(Runnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }

	private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r; //将Runnable对象赋值给Message的Callback属性
        return m;
    }

再返回头看dispatchMessage()方法中的handleCallback(msg)调用，message的callback属性直接调用了run()方法，而不是开启一个新的子线程：

	private static void handleCallback(Message message) {
        message.callback.run();
    }

使用post的好处在于：避免了主线程和子线程中数据传输的麻烦。

### IntentService

对于Message对象，不推荐使用构造方法得到，建议使用Message.obtain()这个静态方法或者Handler.obtainMessage()获取。Message.obtain()会从消息池中获取一个Message对象，如果消息池中是空的，才会使用构造方法实例化一个新Message，这样有利于消息资源的利用。并不需要担心消息池中的消息过多，它是有上限的，上限为10个。Handler.obtainMessage()具有多个重载方法，在内部也是调用Message.obtain()。


#### Message源码分析

	public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }



