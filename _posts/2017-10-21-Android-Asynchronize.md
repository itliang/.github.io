---
layout: post
title:  "Android异步消息处理机制"
date:   2017-10-21 10:50:05
categories: Android
excerpt: Android开发实践笔记。
---

* content
{:toc}

Handler, Looper, Message都是与Android异步消息处理线程相关的概念。 异步消息处理线程启动后会进入一个无限的循环体之中，每循环一次，从其内部的消息队列中取出一个消息，然后回调相应的消息处理函数，执行完成一个消息后则继续循环。若消息队列为空，线程则会阻塞等待。
（参照http://www.cnblogs.com/smyhvae/p/4003922.html 添加图）

Looper主要负责创建一个MessageQueue然后不断地从该MessageQueue中读取消息。

Handler直接继承自Object,主要负责在子线程中发送Message/Runnable对象到MessageQueue中。在UI线程中接收，处理MessageQueue分发出来的Message/Runnable对象。

---
### Looper

Looper是每个线程中的MessageQueue管家，调用Looper的loop()方法后，就会进入到一个无限循环中不断地从MessageQueue读取消息，并调用Handler的handlerMessage()方法。每个线程中也只会有一个Looper对象。

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


### Handler

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
在Ubuntu的的开发环境中：

- curl -s "https://get.sdkman.io" | bash
- source ~/.bashrc
- gvm install groovy
 
创建test.groovy文件，执行groovy test.groovy

