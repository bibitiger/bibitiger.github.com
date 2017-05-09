---
layout: post
title: "PyQt中Thread的使用"
date: 2017-05-09 19:36:40 +0800
comments: true
categories: "Python"
---

##原文地址：[blog.bibitiger.cn/blog/2017/04/11/install-php71-mac/](http://blog.bibitiger.cn/blog/2017/05/09/use-thread-in-pyqt/)

<br/>

---

PyQt中的Thread通常使用QThread，这个个人感觉封装的很好，比如我经常喜欢这样使用C++的thread：

<!--more-->

```
template<typename O, typename I>
class MYThread
{
public:
	MYThread()
	{
	}

	int start()
	{
		return pthread_create(&_tid, NULL, _procedure, NULL);
	}

	int join()
	{
		return pthread_join(_tid, NULL);
	}

	int detach()
	{
		return pthread_detach(_tid);
	}

	static void * _procedure(void * _ptr_arg)
	{
		O::do_something_in_worker_thread();
		return NULL;
	}

private:
	pthread_t _tid;
};

```

我们再来看下，PyQt下的代码：

```
class MyThread(QThread):
    def __init__(self, do_something_in_worker_thread, *args, parent=None):
        super(MyThread, self).__init__(parent)
        self._func = do_something_in_worker_thread
        self._args = args

    def run(self):
        result = self. _func(*self._args)
```

是不是发现很相似，而且有很多的代码都不需要重复的写，加上qt的signal，可以很方便的发出信号和其他线程通信就更方便了，改一下就成了这个样子：

```
class MyThread(QThread):
    finishSignalList = pyqtSignal(list)
    finishSignalInt = pyqtSignal(int)

    def __init__(self, func, *args, parent=None):
        super(MyThread, self).__init__(parent)
        self._func = func
        self._args = args

    def run(self):
        result = self._func(*self._args)
        print(result)
        if isinstance(result, list):
            self.finishSignalList.emit(result)
        elif isinstance(result, int):
            self.finishSignalInt.emit(result)
```

在使用的时候也很方便：

```
self.myTheard = MyThread(self.func, self.beginTime.dateTime().toPyDateTime(), self.endTime.dateTime().toPyDateTime())
self.myTheard.finishSignalList.connect(self.returnFunc)
self.myTheard.start()

def returnFunc(self, result):
	print(result)
```

如果在我们的func上加上输入输出的校验感觉就更完美了，这个在[Python中的注解“@”](http://blog.bibitiger.cn/blog/2017/04/17/pythondecoratorsforfunctions/)里已经说过了