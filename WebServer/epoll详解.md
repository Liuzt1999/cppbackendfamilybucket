# int epoll_create(int size)
返回一个指向epoll实例的文件描述符，后续通过这个文件描述符来对epoll进行各种操作，  
如添加、删除或修改等操作，size参数目前已经被忽略。  
  
# int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
epfd:epoll_create函数返回的文件描述符，用来标识内核中的epoll实例  
op：对fd文件描述符进行操作的类型，主要有  
  EPOLL_CTL_ADD：向epfd实例进行注册，在fd有I/O事件时获得通知  
  EPOLL_CTL_DEL：从epfd中删除fd  
  EPOLL_CTL_MOD：修改正在监视的fd事件  
fd：需要被操作的文件描述符  
event：指向epoll_event结构体的指针，存储了实际要监视fd的事件  
  
    struct epoll_event {  
      uint32_t events; //epoll事件类型，包括读写等  
      epoll_data_t data; //用户数据，可以是一个指针或文件描述符  
    };
      
   events字段表示要监听的事件类型，包括：  
      EPOLLIN：表示对应的文件描述符上有数据可读  
      EPOLLOUT：表示对应的文件描述符上可以写入数据  
      EPOLLERR：表示发生错误  
      EPOLLRDHUP：表示对端已经关闭连接，或者关闭了写操作端的写入  
      EPOLLHUP：表示文件描述符被挂起  
      EPOLLET：表示将epoll设置为边缘触发模式  
      EPOLLONTSHOT：表示将事件设置为一次性事件  
    
    epoll_data{  
      void *ptr;  
      int fd;  
      uint32_t u32;   
      uint64_t u64;   
    }epoll_data_t;  
  fd表示文件描述符
  用法：  
  
      int number = epoll_wait(epollfd, events, MAX_EVENT_NUMBER, -1);
            if ((number < 0) && (errno != EINTR))
            {
                LOG_ERROR("%s", "epoll failure");
                break;
            }

            for (int i = 0; i < number; i++)
            {
                int sockfd = events[i].data.fd;

                //处理新到的客户连接
                if (sockfd == listenfd)
                {
                    bool status = deal_client_connect();
                    if (status == false)
                        continue;
                }
  
# int epoll_wait(int epfd, struct epoll_event *evlist, int maxevents, int timeout);
epdf：epoll_create函数返回的文件描述符，用来标识内核中的epoll实例  
evlist：epoll事件结构的数组，由调用进程分配，当epoll_wait返回时，通过修改这个数组来指示就绪的 fd的信息（就绪列表）  
maxevents: evlist数组大小  
timeout：指定epoll_wait系统调用的阻塞时间， 0为非阻塞，检查完后马上返回；-1为永久阻塞，直到有事件发生或被信号处理程序中断  
返回值：发生错误返回-1；超时无事件发生返回0；有事件发生返回大于0的正数表示发生事件的总数，检查evlist以确定哪些事件发生在哪些fd上  


