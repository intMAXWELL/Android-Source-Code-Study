# Memory Consistency Properties #
[Chapter 17 of the Java Language Specification](http://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5 )在内存操作上定义了*happens-before*关系，例如读和写同一个共享变量。如果写操作*happens-before*读操作，那么写操作的结果对另一线程的读操作的可见性能够得到保证。`Synchronized`和`Volatile`的设计，以及`Thread.start()`和`Thread.join()`方法，都能形成*happens-before*关系。特别地:
- 某线程的任一操作*happens-before*在同一线程位于该操作之后的任何操作。
- 对monitor的解锁(`synchronized`块或者方法的出口) *happens-before* 在这之后的对同一个monitor的上锁(`synchronized`块或者方法入口)。而且因为*happens-before*关系是可传递的，一个线程中所有优先于解锁的动作*happens-before*所有任一线程对monitor上锁之后的操作。
- 对`volatile`域的写操作*happens-before*所有在这之后对该域的读操作。对`volatile`域的读写操作和进入、离开monitor锁区域具有相似的内存一致效应，但是不需要互斥锁。
- 调用线程的`start`*happens-before*所有线程开始后的操作。
- 线程的所有操作*happens-before*从这个线程的` Thread.join()` 成功返回的所有其他线程。