# Executor #

## 概述 ##
通常我们使用`Executor`而不创建一个具体的线程。例如,你可以像下面这样而不是调用`new Thread(new RunnaleTask()).start()`:
     
	 Executor executor = anExecutor;
     executor.execute(new RunnableTask1());
     executor.execute(new RunnableTask2());
     ...
然而，接口`Executor`并非严格要求执行异步。最简单的一种情况，一个`Executor`在调用者的线程执行提交的任务:

     class DirectExecutor implements Executor {
       public void execute(Runnable r) {
     	r.run();
     }}
更加典型地，任务执行在新的线程而不是调用者的线程：

    class ThreadPerTaskExecutor implements Executor {
       public void execute(Runnable r) {
     	new Thread(r).start();
       
     }}

许多`Executor`的实现在任务**何时**及**如何**执行上施加某些限制。例如，下面的`Executor`将提交的将要执行的任务和第二个`Executor`串行化，图解了一个复合的`Executor`。

	 class SerialExecutor implements Executor {
	   final Queue tasks = new ArrayDeque<>();
	   final Executor executor;
	   Runnable active;
	
	   SerialExecutor(Executor executor) {
	     this.executor = executor;
	   
	
	   public synchronized void execute(final Runnable r) {
	     tasks.add(new Runnable() {
	       public void run() {
	         try {
	           r.run();
	         } finally {
	           scheduleNext();
	         }
	       }
	     });
	     if (active == null) {
	       scheduleNext();
	     }
	   }
	
	   protected synchronized void scheduleNext() {
	     if ((active = tasks.poll()) != null) {
	       executor.execute(active);
	     }
	   }
	 }}

Interface` ExecutorService`应用更加广泛的接口。
Class `ThreadPoolExecutor`实现广泛的线程池实现。
Class `Executors`为这些Executors提供方便的工厂方法。
内存一致性效应：线程中将 Runnable 对象提交到 Executor 之前的操作 happen-before 其执行开始（可能在另一个线程中）。