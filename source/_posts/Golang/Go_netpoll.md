---
title: Go网络轮询器netpoll
date: 2025-02-27 00:17:00
categories:
    - [Go,原理]
tags: 
    - Go
---

### 前言

Go内部使用IO多路复用结合NIO实现了一个异步IO模型

将监听fd的事件交由runtime来管理，当协程读取fd数据但是没数据时，park住该协程。在执行协程调度时会检查fd是否就绪，如果就绪，调度器再通知该park住的协程处理fd，在用户层面实现了一个异步IO模型。

Go netpoll在不同操作系统，其底层使用的IO多路复用技术也不一样

linux下使用epoll实现

darwin下使用kqueue实现

在Unix/Linux系统下，一切皆文件，每条TCP连接对应了一个socket句柄，这个句柄也可以看成是一个文件，在socket上收发数据相当于对文件进行读写。可以进入/proc/PID/fd 查看进程占用的fd。

系统内核会为每个socket句柄分配读写缓冲区，当程序调用write/send时，仅仅是把数据拷贝到缓冲区，积累到一定数量时会将数据发送到目的端。

### Linux IO模型

**阻塞式IO：**

程序想要在缓冲区读取数据时，缓冲区并不一定会有数据，这会造成陷入系统调用，只能等待数据可以读取，没有数据读取时会阻塞住进程。

**非阻塞式IO：**

程序想要读取数据时，如果缓冲区数据不存在，则直接返回给用户程序，但是需要用户程序去频繁检查，直到有数据准备好，这同样也会造成空耗CPU。

**多路复用：**

多路复用的核心就是用一个线程来监听多个网络IO

IO多路复用使用一个线程管理多个fd，可以将多个fd加入IO多路复用函数中，每次调该函数，传入要检查的fd，如果有就绪的fd，直接返回就绪的fd，再启动线程处理

常见的IO多路复用函数有select、poll、epoll。（select、poll每次调用需要以某种数据结构传入所有要监听的fd集合，当内核发现某个fd就绪时，会修改数据结构传回给进程，高并发下，用户态于内核态的数据拷贝以及内核轮询fd会浪费很多资源）

**信号驱动IO：**

应用进程向内核注册一个信号处理程序，当内核中有数据准备好，会发送一个信号给应用进程；应用进程便可以在信号处理程序中发起IO系统调用，来完成数据读取（通过信号通知而不是轮询，避免了大量无效的数据状态轮询操作）

**异步IO：**

应用进程发起IO系统调用后，会立即返回，当内核中数据准备好并复制到用户空间后，会产生一个信号来通知应用进程只需发起一次系统调用，便可以完成对数据的读取

### Linux epoll多路复用

#### epoll API

```c
// 创建一个epoll实例，返回epfd句柄
int epoll_create(int size); 

// 用于向epoll实例添加、删除、修改要监听的fd等待的IO事件
int epoll_ctl(int epfd,int op,int fd,struct epoll_event* event); 

// 阻塞监听epoll实例上的IO事件
int epoll_wait(int epfd,struct epoll_event* events,int maxevents,int timeout); 
```



#### epoll工作原理

epoll采用红黑树来存储所有监听的fd，通过epoll_ctl将fd添加到红黑树，该fd会与相应的设备（网卡）建立回调关系，也就是在内核中断处理程序为它注册一个回调函数，该回调函数被称为ep_poll_callback（将这个fd添加到rdllist就绪链表中）

epoll_wait实际上就是去检查rdllist就绪链表是否有就绪的fd，就绪链表为空时挂起，直到就绪链表非空才被唤醒返回

### Go netpoller 核心

首先，client连接server的时候，listener通过accept接收新connection，每个新connection都启动一个goroutine处理。

accept会把该connection的fd连带所在的goroutine上下文信息封装注册到epoll的监听列表里。

当goroutine调用conn.Read或者conn.Write等需要阻塞等待的函数时，会被gopark给封存起来并使之休眠。

往后Go Scheduler会在循环调度的runtime.schedule()函数以及sysmom监控线程中调用runtime.netpoll获取可以运行的goroutine列表并执行。

#### net.Listen

调用net.Listen之后，底层会通过Linux的系统调用socket方法创建一个fd，接着调用listenStream方法完成对socket的bind&listen操作。

```go
func socket(ctx context.Context, net string, family, sotype, proto int, ipv6only bool, laddr, raddr sockaddr, ctrlFn func(string, string, syscall.RawConn) error) (fd *netFD, err error) {
   s, err := sysSocket(family, sotype, proto)
   if err != nil {
      return nil, err
   }
  ...

   if laddr != nil && raddr == nil {
      switch sotype {
      case syscall.SOCK_STREAM, syscall.SOCK_SEQPACKET:
          // 对listener fd 进行 bind&listen 操作
         if err := fd.listenStream(laddr, listenerBacklog(), ctrlFn); err != nil {
            fd.Close()
            return nil, err
         }
         return fd, nil
...
   }
...
}

func (fd *netFD) listenStream(laddr sockaddr, backlog int, ctrlFn func(string, string, syscall.RawConn) error) error {
   ...
   // 绑定
   if err = syscall.Bind(fd.pfd.Sysfd, lsa); err != nil {
      return os.NewSyscallError("bind", err)
   }
   // 监听
   if err = listenFunc(fd.pfd.Sysfd, backlog); err != nil {
      return os.NewSyscallError("listen", err)
   }
   // 初始化epoll实例，将lintener fd 注册到实例
   if err = fd.init(); err != nil {
      return err
   }
   lsa, _ = syscall.Getsockname(fd.pfd.Sysfd)
   fd.setAddr(fd.addrFunc()(lsa), nil)
   return nil
}
```

#### net.Accept

```go
func (fd *FD) Accept() (int, syscall.Sockaddr, string, error) {
   if err := fd.readLock(); err != nil {
      return -1, nil, "", err
   }
   defer fd.readUnlock()

   if err := fd.pd.prepareRead(fd.isFile); err != nil {
      return -1, nil, "", err
   }
   for {
       // 使用linux系统调用accept接收新连接， 非阻塞
      s, rsa, errcall, err := accept(fd.Sysfd)
      if err == nil {
         return s, rsa, "", err
      }
      switch err {
      case syscall.EINTR:
         continue
      case syscall.EAGAIN:
         if fd.pd.pollable() {
             // 如果没有期待的IO事件，则park goroutine 让逻辑 block 在这里
            if err = fd.pd.waitRead(fd.isFile); err == nil {
               continue
            }
         }
      case syscall.ECONNABORTED:
         // This means that a socket on the listen
         // queue was closed before we Accept()ed it;
         // it's a silly error, so try again.
         continue
      }
      return -1, nil, errcall, err
   }
}
```

#### conn.Read

调用conn.Read的时候会调用fd的read方法，当fd没有数据可以读的时候会返回EAGAIN，这时goroutine会被park住，当socket处于就绪状态的时候会把对应goroutine唤醒

```go
func (fd *FD) Read(p []byte) (int, error) {
   if err := fd.readLock(); err != nil {
      return 0, err
   }
   defer fd.readUnlock()
   if len(p) == 0 {
      // If the caller wanted a zero byte read, return immediately
      // without trying (but after acquiring the readLock).
      // Otherwise syscall.Read returns 0, nil which looks like
      // io.EOF.
      // TODO(bradfitz): make it wait for readability? (Issue 15735)
      return 0, nil
   }
   if err := fd.pd.prepareRead(fd.isFile); err != nil {
      return 0, err
   }
   if fd.IsStream && len(p) > maxRW {
      p = p[:maxRW]
   }
   for {
      n, err := ignoringEINTRIO(syscall.Read, fd.Sysfd, p)
      if err != nil {
         n = 0
         if err == syscall.EAGAIN && fd.pd.pollable() {
              // 如果没有期待的IO事件，则park goroutine 让逻辑 block 在这里
            if err = fd.pd.waitRead(fd.isFile); err == nil {
               continue
            }
         }
      }
      err = fd.eofError(n, err)
      return n, err
   }
}
```

### 参考资料

[1] [Go netpoller 原生网络模型](https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651443085&idx=3&sn=2c1ed8474bc7fed68b519ce9e5f5e0b0)

[2] [Go原生网络轮询器netpoller](https://zhuanlan.zhihu.com/p/463017601)

