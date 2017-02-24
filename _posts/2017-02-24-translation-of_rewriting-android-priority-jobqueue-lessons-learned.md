---
layout: default
title: 重写Android Priority JobQueue并总结经验(翻译)
category: 
comments: true
---

# 重写Android Priority JobQueue并总结经验(翻译)


2016年早春,我决定了要重构一个四年前我在[Path](https://path.com)工作的时候为了让app能有良好的离线体验而写的一个任务队列:[Android Priority JobQueue](https://github.com/yigit/android-priority-jobqueue)的内核.


经历了四个春夏秋冬，它已经从一个简单的任务队列管理系统成长为一个可以精准控制后台任务运行的复杂的任务队列管理系统.同时，安卓生态环境也跟随着改变了不少，尤其是[JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html)的出现.


由于在早期开发的时候做出了一些错误的决定导致了现在很难继续维护这份代码让它很好地适应安卓生态环境的发展.现在我已经完成了它的重构了，所以我觉得应该分享一下重构的经验并希望它在重构之后可以有一个更好的发展.



## 不要用共享内存来做线程间通讯;用消息传递来做共享内存


这个是这次重构里面最重要的一个任务,也是我在开发上一个版本的时候做的最错的一个决定.任务管理者天生就是多线程的，它必须能够并行运行多个任务.所以将需要共享的资源用一个线程锁来锁住是最常见的做法,这样同一时间就只有一个线程可以访问这个资源.刚开始的时候这样做没有出现任何问题,到后来就变得不可控制了，尤其是当你将访问这些共享资源的api开放出去之后.上一个版本的任务管理者也有一些关于线程锁的bug是几乎没有任何办法可以解决.同时 还有一些非常难管理的小问题:你要小心地处理内存屏障或者将某些必要的变量标记为```volatile```.

在新版本里面我将这些共享资源的方法完全改了从而让任务管理者变为单线程.任务管理者在它自己的一个线程里面运行着，这个线程是唯一一个可以访问共享资源的线程.在其他线程里面运行的任务只能通过线程间通讯来跟任务管理者沟通.而且它们并不能访问共享资源，实际上它们完全没有任何方法可以拿到共享资源的内存引用,这样就避免了以后再在这个地方出bug.就像是任务消费者一样,所有的公开
api都用这样的消息传递方法来跟任务管理者沟通.

