#一、 Rxjava大总结 #
- Rxjava代码非常的简洁，目录下的 png 图片都加载出来并显示，需要开启线程异步加载，如果使用rxjava加载，就避免了代码的层次嵌套，方便阅读和更改。
- Rxjava采用观察者模式，RxJava 的基本实现主要有三点创建Observer 即观察者，RxJava 还内置了一个实现了 Observer 的抽象类：Subscriber，基本使用方式一样，实质上，在 RxJava 的 subscribe 过程中，Observer 也总是会先被转换成一个 Subscriber 再使用。所以如果你只想使用基本功能，选择 Observer 和 Subscriber 是完全一样的，区别是，onStart(): 这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，unsubscribe(): 这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅，调用 unsubscribe() 来解除引用关系，以避免内存泄露的发生。
- 建 Observable 即被观察者，create() 方法是 RxJava 最基本的创造事件序列的方法。基于这个方法， RxJava 还提供了一些方法用来快捷创建事件队列，例如：just(T...): 将传入的参数依次发送出来。from(T[]) / from(Iterable<? extends T>) : 将传入的数组或 Iterable 拆分成具体对象后，依次发送出来。
- 创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了
- subscriber() 做了3件事：1.调用 Subscriber.onStart() 。这个方法在前面已经介绍过，是一个可选的准备方法。2，调用 Observable 中的 OnSubscribe.call(Subscriber) 。在这里，事件发送的逻辑开始运行。从这也可以看出，在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。,3，将传入的 Subscriber 作为 Subscription 返回。这是为了方便 unsubscribe().
- 程控制Scheduler，在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。通过subscribeOn() 和 observeOn()更换，即subscribe() 所发生的线程和ubscriber 所运行在的线程。
- 个人感觉最厉害的就是其中的变换，1、map(): 事件对象的直接变换，2、flatMap(): 是变换成一组数据，不需要for循环遍历了。其中的原理是lift()，
1. lift() 创建了一个 Observable （被观察者）后，加上之前的原始 Observable，已经有两个 Observable 了
2. ，新 Observable 里的新 OnSubscribe 加上之前的原始 Observable 中的原始 OnSubscribe，也就有了两个 OnSubscribe；
3. 当用户调用经过 lift() 后的 Observable 的 subscribe() 的时候，使用的是 lift() 所返回的新的 Observable ，于是它所触发的 onSubscribe.call(subscriber)，也是用的新 Observable 中的新 OnSubscribe
4. 而这个新 OnSubscribe 的 call() 方法中的 onSubscribe ，就是指的原始 Observable 中的原始 OnSubscribe
5. 新 OnSubscribe 利用 operator.call(subscriber) 生成了一个新的 Subscriber然后利用这个新 Subscriber 向原始 Observable 进行订阅。 
6. 这样就实现了 lift() 过程，有点像一种代理机制，通过事件拦截和处理实现事件序列的变换。

----------






















#二、 Android中应用程序发送广播的过程： #
- 通过sendBroadcast把一个广播通过Binder发送给ActivityManagerService，ActivityManagerService根据这个广播的Action类型找到相应的广播接收器，然后把这个广播放进自己的消息队列中，就完成第一阶段对这个广播的异步分发。
- ActivityManagerService在消息循环中处理这个广播，并通过Binder机制把这个广播分发给注册的ReceiverDispatcher，ReceiverDispatcher把这个广播放进MainActivity所在线程的消息队列中，就完成第二阶段对这个广播的异步分发：
- ReceiverDispatcher的内部类Args在MainActivity所在的线程消息循环中处理这个广播，最终是将这个广播分发给所注册的BroadcastReceiver实例的onReceive函数进行处理：、


