# UI卡顿优检测大总结 #
1. 从Looger.loop源码可以发现，在msg.target.dispathMessgae（msg）分别有开始和结束的logging日志
2. 通过loop.getMainLooper.setMessageLogging 获取日志信息，通过计算两次的时间差值
3. 利用HandlerThread，如果达到我们设置的耗时时间就会打印UI线程当前堆栈信息
，如果在设置的时间之内就会remo掉该runnable，之所以使用HandlerThread因为开发中如果多次使用类似new Thread(){}.start(); 这种方式开启一个子线程，会创建多个匿名线程，使得程序运行越来越慢，而HandlerThread自带Looper使他可以通过消息来多次重复使用当前线程，节省开支
2、Android系统提供的Handler类内部的Looper默认绑定的是UI线程的消息队列，对于非UI线程又想使得消息机制，那么HandlerThread内部的Looper是最合适的，它不会干扰或阻塞UI线程。


