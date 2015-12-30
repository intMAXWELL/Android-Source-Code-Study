# AsyncTask #

## 概述 ##

- AsyncTask被设计成助手类,处理`Handler`和`Thread`,但是它并未形成一般的线程框架。

- AsyncTasks最好运用在短操作场景(最多几秒种。)

- 如果你需要使线程运行保持比较长的时间周期，强烈推荐由`java.util.concurrent`提供的其他各种API例如`Executor`,`ThreadPoolExecutor`和`FutureTask`。

- AsyncTask运行于非主线程，并将结果更新在UI线程。 AsyncTask定义了三个泛型参数`Params`,`Progess`,`Result`,任务执行主要分为四步`onPreExecute`,`doInBackground`,`onProgressUpdate`和`onPostExecute`。

## 用法 ##

Async必须继承使用。子类必须重写`doInBackground`方法,通常也会重写`onPostExecute`。

下面是一个例子:

     private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
     	protected Long doInBackground(URL... urls) {
         	int count = urls.length;
         	long totalSize = 0;
         	for (int i = 0; i < count; i++) {
             	totalSize += Downloader.downloadFile(urls[i]);
             	publishProgress((int) ((i / (float) count) * 100));
             	// Escape early if cancel() is called
             	if (isCancelled()) break;
        	 }
         	return totalSize;
     	}

     	protected void onProgressUpdate(Integer... progress) {
         	setProgressPercent(progress[0]);
     	}
	
     	protected void onPostExecute(Long result) {
         	showDialog("Downloaded " + result + " bytes");
     	}
	}

子类一旦创建，执行起来非常简单:

     new DownloadFilesTask().execute(url1, url2, url3);

## AsyncTask的三个泛型参数 ##
1. `Params`, 任务执行所需要的参数
2. `Progress`,后台任务运行时进度更新的参数
3. `Result`,后台任务结束后的结果参数

并非所有的参数都有作用，如果不需要，刻意简单的设置为Void

    private class MyTask extends AsyncTask<Void, Void, Void> { ... }

## 四个步骤 ##

1. `onPreExecute()`,在主线程中执行，执行前任务的初始化，可以更新UI。
2. `doInBackground(Params...)`,在非主线程中执行耗时逻辑。在本步中，可以调用`publishProgress(Progress...)`更新UI。
3. `onProgressUpdate`,在调用`publishProgress(Progress...)`方法后在主线程中更新UI。
4. `onPostExecute(Result)`,`doInBackground(Params...)`执行后在主线程执行。

## 取消任务 ##
在任何时候，可以通过调用cancel(boolean)取消任务。在调用cancel(boolean) 方法后，调用isCancelled()将返回true，而且在执行完`doInBackground(Param...)`后将不执行`onPostExecute(Object)`,取而代之的是`onCancelled(Object)`。

## Threading rules ##
- AsyncTask**必须**在UI线程中加载。在`JELLY_BEAN`自动完成了此步骤。
- AsyncTask实例**必须**在UI线程中创建。
- execute(Params...)**必须**在UI线程中调用。
- 不要人为调用 `onPreExecute()`,`onPostExecute(Result)`,`doInBackground(Params...)`,`onProgressUpdate(Progress...)`。
- 该任务只能执行一次。

## Memory observability ##
AsyncTask确保所有的回调都是同步的，所以以下操作是安全的而不需要显示同步。
- 在Constructor或者`onPreExecute()`设置成员变量，然后在`doInBackground()`中引用相关。
- 在`doInBackground()`中设置成员变量，然后在`onProgressUpdate()`和`onPostExecute()`中引用相关。

## 执行顺序 ##
在最初提出，AsyncTask在单后台线程顺序执行。自从DONUT,改为在一个线程池中多任务并行执行。自从HONEYCOMB,为了避免由共同应用造成的并行执行错误，AsyncTasks在单线程中运行。
如果你确实想要并行执行，你可以调用executeOnExecutor(Java.util.concurrent.Executor,Object[])。






 




