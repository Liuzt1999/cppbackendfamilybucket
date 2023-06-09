# select、poll和epoll
## select  
通过线性表描述文件描述符集合，文件描述符上限一般为1024，通过修改源码重新编译内核可修改，不推荐  
通过文件描述符集合拷贝进内核，让内核通过遍历的方式检查是否有事件发生，并将发生事件的socket标记  
有事件发生后将文件描述符集合拷贝到用户态，用户态通过遍历找到对应的socket  
## poll
通过链表描述突破了文件描述符上限，但仍需要频繁在用户态和内核态之间进行拷贝和遍历  
## epoll
在内核使用红黑树来跟踪文件描述符，把需要监控的socket通过epoll_ctl函数加入内核中的红黑树中  
使用事件驱动机制，在内核里维护一个链表记录就绪事件，调用epoll_wait时只返回有事件发生的文件描述符的结构体数组，结构体包含socket的信息  
在活跃socket较少的情况下使用epoll可以提高性能，所有socket都很活跃时可能有性能问题  
  
# 工作模式
select和poll都只能工作在LT模式下，epoll可以使用ET模式  
## 水平触发LT
服务端会不断从epoll_wait中苏醒直到内核缓冲区被read函数读完，持续通知到读完  
## 边缘触发ET
服务端只会从epoll_wait中苏醒一次，即使没有读完也不会再次通知，因此要保证一次将内核缓冲区读完。可以  
使用EPOLLONESHOT，使一个socket连接在任一时刻都只被一个线程处理，进一步减少可读、可写和异常事件被触发的次数 

  
#  应用场景
当所有fd都是活跃连接，epoll由于需要建立文件系统、红黑树和链表，此时效率不如select和poll  
当检测fd数目较小，且各个fd都比较活跃，建议使用select或poll  
当检测fd数目很大，且单位时间只有一小部分fd处于就绪状态，此时使用epoll能明显提升性能  

