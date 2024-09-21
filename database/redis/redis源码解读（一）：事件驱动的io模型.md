# redis源码解读（一）：事件驱动的io模型，为什么，是什么，怎么做

引言，为什么要读redis源码：

为什么需要事件驱动的io模型

同步阻塞式io

多线程io

io多路复用

什么是事件驱动的io模型（Reactor）

事件驱动的io模型在redis中的实现

整体框架

事件注册与获取

读写事件的实现

如何使用vscode调试redis源码

引言，为什么要读redis源码：
----------------

Redis作为一个高性能的[内存数据库](https://zhida.zhihu.com/search?q=%E5%86%85%E5%AD%98%E6%95%B0%E6%8D%AE%E5%BA%93&zhida_source=entity&is_preview=1)，因其出色的读写性能和丰富的数据结构支持，已成为互联网应用不可或缺的中间件之一（个人感觉应该是最常用的中间件）。阅读其源码，可以了解其内部针对高性能和分布式做的种种设计，包括但不限于**[reactor模型](https://zhida.zhihu.com/search?q=reactor%E6%A8%A1%E5%9E%8B&zhida_source=entity&is_preview=1)（单线程处理大量网络连接），定时任务的实现（面试常问），分布式CAP BASE理论的实际应用，高效的数据结构的实现**，其次还能够通过大神的代码学习C语言的编码风格和技巧，让自己的代码更加优雅。

* * *

下面进入正题：

为什么需要事件驱动的[io模型](https://zhida.zhihu.com/search?q=io%E6%A8%A1%E5%9E%8B&zhida_source=entity&is_preview=1)
--------------------------------------------------------------------------------------------------------

我们可以简单地将一个服务端程序拆成三部分，接受请求->处理请求->返回结果，其中接收请求和处理请求便是我们常说的网络io。那么网络io如何实现呢，首先我们介绍最基础的io模型，[同步阻塞式io](https://zhida.zhihu.com/search?q=%E5%90%8C%E6%AD%A5%E9%98%BB%E5%A1%9E%E5%BC%8Fio&zhida_source=entity&is_preview=1)，也是很多同学在学校所学的“[网络编程](https://zhida.zhihu.com/search?q=%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B&zhida_source=entity&is_preview=1)”。

### 同步阻塞式io

使用同步阻塞式io的单线程服务端程序处理请求大致有以下几个步骤

1.  创建socket并绑定ip，port
2.  调用listen，监听ip，port
3.  调用accept，获取一个已经建立完成的连接
4.  调用[recv](https://zhida.zhihu.com/search?q=recv&zhida_source=entity&is_preview=1)，读取客户端发送的数据
5.  处理请求
6.  调用send返回结果
7.  调用close断开连接

其中3,4步都有可能使线程阻塞（6也会可能阻塞，这里先不讨论）

在第3步，如果没有客户端请求和服务端建立连接，那么服务端线程将会阻塞。如果redis采用这种io模型，那主线程就无法执行一些定时任务，比如过期key的清理，持久化操作，[集群操作](https://zhida.zhihu.com/search?q=%E9%9B%86%E7%BE%A4%E6%93%8D%E4%BD%9C&zhida_source=entity&is_preview=1)等。

在第4步，如果客户端已经建立连接但是没有发送数据，服务端线程会阻塞。若说第3步所提到的定时任务还可以通过多开两个线程来实现，那么第4步的阻塞就是硬伤了，如果一个客户端建立了连接但是一直不发送数据，服务端便会崩溃，无法处理其他任何请求。所以同步阻塞式io肯定是不能满足互联网领域高并发的需求的。

下面给出一个阻塞式io的服务端程序示例：

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

const int PORT = 8081;
const int BUFFER_SIZE = 1024;

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};
    const char *hello = "Hello from server";

    // 创建套接字
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 配置地址结构体
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // 绑定套接字
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 监听传入连接
    if (listen(server_fd, 3) < 0) {
        perror("listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    std::cout << "Server is listening on port " << PORT << std::endl;

    while (true) {
        // 接受传入连接
        if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
            perror("accept failed");
            close(server_fd);
            exit(EXIT_FAILURE);
        }

        std::cout << "Connection established" << std::endl;

        // 读取客户端数据
        int valread = read(new_socket, buffer, BUFFER_SIZE);
        std::cout << "Received: " << buffer << std::endl;

        // 回显数据给客户端
        send(new_socket, buffer, valread, 0);
        std::cout << "Echo message sent" << std::endl;

        // 关闭连接
        close(new_socket);
        std::cout << "Connection closed" << std::endl;
    }

    return 0;
}
```

### [多线程io](https://zhida.zhihu.com/search?q=%E5%A4%9A%E7%BA%BF%E7%A8%8Bio&zhida_source=entity&is_preview=1)

刚才提到，阻塞式io的主要问题是，调用recv接收客户端请求时会导致线程阻塞，无法处理其他客户端请求。那么我们不难想到，既然调用recv会使线程阻塞，那么我们多开几个几个线程不就好了，让那些没有阻塞的线程去处理其他客户端的请求。

我们将阻塞式io处理请求的步骤改造下：

1.  创建socket并绑定ip，port
2.  调用listen，监听ip，port
3.  调用accept，获取一个已经建立完成的连接
4.  创建新的线程，在新的线程中做：

1.  调用recv，读取客户端发送的数据
2.  处理请求
3.  调用send返回结果
4.  调用close断开连接

改造后，我们用一个线程去做accept，也就是获取已经建立的连接，我们称这个线程为主线程。然后获取到的每个连接开一个新的线程去处理，这样就能够将阻塞的部分放到新的线程，达到不阻塞主线程的目的，主线程仍然可以继续接收其他客户端的连接并开新的线程去处理。这个方案对高并发服务器来说是一个可行的方案，此外我们还可以使用线程池等手段来继续优化，减少线程建立和销毁的开销。

将阻塞式io改为多线程io：

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <thread>

const int PORT = 8081;
const int BUFFER_SIZE = 1024;

void handle_client(int client_socket) {
    char buffer[BUFFER_SIZE] = {0};
    int valread = read(client_socket, buffer, BUFFER_SIZE);
    std::cout << "Received: " << buffer << std::endl;

    // 请求处理
    sleep(5);

    // 回显数据给客户端
    send(client_socket, buffer, valread, 0);
    std::cout << "Echo message sent" << std::endl;

    // 关闭连接
    close(client_socket);
    std::cout << "Connection closed" << std::endl;
}

int main() {
    int server_fd;
    struct sockaddr_in address;
    int addrlen = sizeof(address);

    // 创建套接字
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 配置地址结构体
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // 绑定套接字
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 监听传入连接
    if (listen(server_fd, 3) < 0) {
        perror("listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    std::cout << "Server is listening on port " << PORT << std::endl;

    while (true) {
        int new_socket;
        // 接受传入连接
        if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
            perror("accept failed");
            close(server_fd);
            exit(EXIT_FAILURE);
        }

        std::cout << "Connection established" << std::endl;

        // 创建新线程处理客户端连接
        std::thread client_thread(handle_client, new_socket);
        client_thread.detach();  // 分离线程，使其在后台运行
    }

    return 0;
}
```

### io多路复用

我们刚才提到多线程可以解决并发问题，然而redis6.0之前使用的是单线程来处理，之所以用单线程，官方给的答复是redis的瓶颈不在cpu，既然不在cpu那么用单线程可以降低系统的复杂度，避免[线程同步](https://zhida.zhihu.com/search?q=%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5&zhida_source=entity&is_preview=1)等问题。如何在一个线程中非阻塞地处理多个socket，进而实现多个客户端的并发处理呢，那就要借助[io多路复用](https://zhida.zhihu.com/search?q=io%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8&zhida_source=entity&is_preview=1)了。

io多路复用是操作系统提供的另一种io机制，这种机制可以实现在一个线程中监控多个socket，返回可读或可写的socket，当一个socket可读或可写时再去操作它，这样就避免了对某个socket的阻塞等待。

将多线程io改为io多路复用：

```cpp
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <sys/epoll.h>
#include <fcntl.h>

const int PORT = 8081;
const int BUFFER_SIZE = 1024;
const int MAX_EVENTS = 10;

void set_nonblocking(int sockfd) {
    int flags = fcntl(sockfd, F_GETFL, 0);
    fcntl(sockfd, F_SETFL, flags | O_NONBLOCK);
}

int main() {
    int server_fd, new_socket, epoll_fd;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE];

    // 创建套接字
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // 设置为非阻塞模式
    set_nonblocking(server_fd);

    // 配置地址结构体
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // 绑定套接字
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 监听传入连接
    if (listen(server_fd, 3) < 0) {
        perror("listen failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    // 创建 epoll 实例
    epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1 failed");
        close(server_fd);
        exit(EXIT_FAILURE);
    }

    struct epoll_event ev, events[MAX_EVENTS];
    ev.events = EPOLLIN;
    ev.data.fd = server_fd;
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, &ev) == -1) {
        perror("epoll_ctl failed");
        close(server_fd);
        close(epoll_fd);
        exit(EXIT_FAILURE);
    }

    std::cout << "Server is listening on port " << PORT << std::endl;

    while (true) {
        int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            perror("epoll_wait failed");
            close(server_fd);
            close(epoll_fd);
            exit(EXIT_FAILURE);
        }

        for (int n = 0; n < nfds; ++n) {
            if (events[n].data.fd == server_fd) {
                // 处理新的连接
                while ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) != -1) {
                    set_nonblocking(new_socket);
                    ev.events = EPOLLIN | EPOLLET; // 使用边缘触发模式
                    ev.data.fd = new_socket;
                    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, new_socket, &ev) == -1) {
                        perror("epoll_ctl: new_socket failed");
                        close(new_socket);
                    }
                }
                if (new_socket == -1 && errno != EAGAIN && errno != EWOULDBLOCK) {
                    perror("accept failed");
                }
            } else {
                // 处理客户端请求
                int client_socket = events[n].data.fd;
                int valread = read(client_socket, buffer, BUFFER_SIZE);
                if (valread == -1) {
                    if (errno != EAGAIN && errno != EWOULDBLOCK) {
                        perror("read failed");
                        close(client_socket);
                    }
                } else if (valread == 0) {
                    // 客户端关闭连接
                    close(client_socket);
                } else {
                    // 回显数据给客户端
                    send(client_socket, buffer, valread, 0);
                }
            }
        }
    }

    close(server_fd);
    close(epoll_fd);
    return 0;
}
```

### 什么是事件驱动的io模型（Reactor）

这里只讨论redis用到的单线程Reactor模型

事件驱动的io模型并不是一个具体的调用，而是高并发服务器的一种抽象的编程模式。

在Reactor模型中，有三种事件：

1.  [连接事件](https://zhida.zhihu.com/search?q=%E8%BF%9E%E6%8E%A5%E4%BA%8B%E4%BB%B6&zhida_source=entity&is_preview=1)：客户端请求与服务端建立连接
2.  读事件：服务端读取客户端请求内容
3.  写事件：服务端将结果写回给客户端

与这三种事件对应的，有三种[handler](https://zhida.zhihu.com/search?q=handler&zhida_source=entity&is_preview=1)，负责处理对应的事件。我们在一个主循环中不断判断是否有事件到来（一般通过io多路复用获取事件），有事件到来就调用对应的handler去处理时间。

听着玄乎，实际上也就这一张图：

<img src="https://pic1.zhimg.com/v2-f29591d59830cf8e7e266203f7b5ac72\_b.jpg" data-size="normal" data-rawwidth="1082" data-rawheight="496" class="origin\_image zh-lightbox-thumb" width="1082" data-original="https://pic1.zhimg.com/v2-f29591d59830cf8e7e266203f7b5ac72\_r.jpg"/>

![](https://pic1.zhimg.com/80/v2-f29591d59830cf8e7e266203f7b5ac72_720w.webp)

单线程Reactor模型示意图

事件驱动的io模型在redis中的实现
-------------------

以下提及的源码版本为 5.0.8

文字的苍白的，建议参照本文最后的方法下载代码，自己调试下

### 整体框架

redis-server的main方法在 src/server.c 最后，在main方法中，首先进行一系列的初始化操作，最后进入进入Reactor模型的主循环中：

```cpp
int main(int argc, char **argv) {

    ......  // 一坨初始化

    // 设置一些回调函数
    aeSetBeforeSleepProc(server.el,beforeSleep);
    aeSetAfterSleepProc(server.el,afterSleep);

    // 进入主循环，后续会一直执行这个函数
    aeMain(server.el);

    // 退出redis-server，主循环结束，回收资源
    aeDeleteEventLoop(server.el);
    return 0;
}
```

主循环在aeMain函数中，aeMain函数传入的参数 server.el ，是一个 aeEventLoop 类型的全局变量，保存了主循环的一些状态信息，包括需要处理的读写事件、时间事件列表，epoll相关信息，回调函数等。

```cpp
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        if (eventLoop->beforesleep != NULL)
            eventLoop->beforesleep(eventLoop);
        aeProcessEvents(eventLoop, AE_ALL_EVENTS|AE_CALL_AFTER_SLEEP);  // 处理时间事件、读写事件
    }
}
```

aeMain函数中，我们可以看到当 eventLoop->stop 标志位为0时，while循环中的内容会被重复执行，每次循环首先会调用beforesleep回调函数，然后处理时间。beforesleep函数在main函数中被注册，会进行集群状态更新、AOF落盘等任务。

之所以叫beforesleep，是因为aeProcessEvents函数中包含了获取事件和处理事件的逻辑，其中获取读写事件时通过epoll\_wait实现，会将线程阻塞。

```cpp
int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
    int processed = 0, numevents;

    /* 不处理任何事件 */
    if (!(flags & AE_TIME_EVENTS) && !(flags & AE_FILE_EVENTS)) return 0;

    /* 获取距离最近的时间事件的时间 */
    if (eventLoop->maxfd != -1 ||
        ((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {
        int j;
        aeTimeEvent *shortest = NULL;
        struct timeval tv, *tvp;

        if (flags & AE_TIME_EVENTS && !(flags & AE_DONT_WAIT))
            shortest = aeSearchNearestTimer(eventLoop);
        if (shortest) {
            long now_sec, now_ms;

            aeGetTime(&now_sec, &now_ms);
            tvp = &tv;

            /* How many milliseconds we need to wait for the next
             * time event to fire? */
            long long ms =
                (shortest->when_sec - now_sec)*1000 +
                shortest->when_ms - now_ms;

            if (ms > 0) {
                tvp->tv_sec = ms/1000;
                tvp->tv_usec = (ms % 1000)*1000;
            } else {
                tvp->tv_sec = 0;
                tvp->tv_usec = 0;
            }
        } else {
            /* If we have to check for events but need to return
             * ASAP because of AE_DONT_WAIT we need to set the timeout
             * to zero */
            if (flags & AE_DONT_WAIT) {
                tv.tv_sec = tv.tv_usec = 0;
                tvp = &tv;
            } else {
                /* Otherwise we can block */
                tvp = NULL; /* wait forever */
            }
        }

        /* 底层调用epoll_wait，获取读写事件。如果没有读写事件到来，会在超时后返回（超时时间为最近的时间事件到来的时间） */
        numevents = aeApiPoll(eventLoop, tvp);

        /* After sleep 回调函数. */
        if (eventLoop->aftersleep != NULL && flags & AE_CALL_AFTER_SLEEP)
            eventLoop->aftersleep(eventLoop);

        // 处理读写事件
        for (j = 0; j < numevents; j++) {
            aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
            int mask = eventLoop->fired[j].mask;
            int fd = eventLoop->fired[j].fd;
            int fired = 0; /* Number of events fired for current fd. */

            int invert = fe->mask & AE_BARRIER;

            // 处理读事件
            if (!invert && fe->mask & mask & AE_READABLE) {
                fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                fired++;
            }

            /* 处理读事件. */
            if (fe->mask & mask & AE_WRITABLE) {
                if (!fired || fe->wfileProc != fe->rfileProc) {
                    fe->wfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }

            /* invert定义了读写事件的处理顺序，如果配置了读事件在写事件后处理，那么此时处理读事件 */
            if (invert && fe->mask & mask & AE_READABLE) {
                if (!fired || fe->wfileProc != fe->rfileProc) {
                    fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }

            processed++;
        }
    }
    /* 处理时间事件 */
    if (flags & AE_TIME_EVENTS)
        processed += processTimeEvents(eventLoop);

    return processed; /* return the number of processed file/time events */
}
```

在aeProcessEvents函数中，处理读写事件和时间事件，参数flags定义了需要处理的事件类型，我们可以暂时忽略这个参数，认为读写时间都需要处理。

aeProcessEvents函数的逻辑可以分为三个部分，首先获取距离最近的时间事件，这一步的目的是为了确定epoll\_wait的超时时间，并不是实际处理时间事件。

第二个部分为获取读写事件并处理，首先调用epoll\_wait，获取需要处理的读写事件，超时时间为第一步确定的时间，也就是说，如果在超时时间内有读写事件到来，那么处理读写时间，如果没有读写时间就阻塞到下一个时间事件到来，去处理时间事件。

第三个部分为处理时间事件。

### 事件注册与获取

上面我们讲了整体框架，了解了主循环的大致流程。接下来我们来看其中的细节，首先是读写时间的注册与获取。

redis将读、写、连接事件用结构aeFileEvent表示，因为这些事件都是通过epoll\_wait获取的。

```cpp
/* File event structure */
typedef struct aeFileEvent {
    int mask; /* one of AE_(READABLE|WRITABLE|BARRIER) */
    aeFileProc *rfileProc;
    aeFileProc *wfileProc;
    void *clientData;
} aeFileEvent;
```

事件的具体类型通过mask标志位来区分。aeFileEvent还保存了事件处理的回调函数指针（rfileProc、wfileProc）和需要读写的数据指针（clientData）。

既然读写事件是通过epoll io多路复用实现，那么就避不开epoll的[三部曲](https://zhida.zhihu.com/search?q=%E4%B8%89%E9%83%A8%E6%9B%B2&zhida_source=entity&is_preview=1) epoll\_create epoll\_ctrl epoll\_wait，接下来我们看下redis对epoll接口的封装。

我们之前提到aeMain函数的参数是一个 aeEventLoop 类型的全局变量，aeEventLoop中保存了epoll[文件描述符](https://zhida.zhihu.com/search?q=%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6&zhida_source=entity&is_preview=1)和epoll事件。在aeApiCreate函数（src/ae\_epoll.c）中，会**调用epoll\_create来创建初始化epoll文件描述符和epoll事件**，调用关系为 main -> initServer -> aeCreateEventLoop -> aeApiCreate

```cpp
static int aeApiCreate(aeEventLoop *eventLoop) {
    aeApiState *state = zmalloc(sizeof(aeApiState));

    if (!state) return -1;
    state->events = zmalloc(sizeof(struct epoll_event)*eventLoop->setsize);
    if (!state->events) {
        zfree(state);
        return -1;
    }
    state->epfd = epoll_create(1024); /* 1024 is just a hint for the kernel */
    if (state->epfd == -1) {
        zfree(state->events);
        zfree(state);
        return -1;
    }
    eventLoop->apidata = state;
    return 0;
}
```

调用epoll\_create创建epoll后，就可以**添加需要监控的文件描述符**了，需要监控的情形有三个，一是监控新的客户端连接连接请求，二是监控客户端发送指令，也就是读事件，三是监控客户端写事件，也就是处理完了请求写回结果。

这三种情形在redis中被抽象为文件事件，文件事件通过函数aeCreateFileEvent（src/ae.c）添加，添加一个文件事件主要包含三个步骤，通过epoll\_ctl添加监控的文件描述符，指定回调函数和指定读写缓冲区。

```cpp
// src/ae.c
int aeCreateFileEvent(aeEventLoop *eventLoop, int fd, int mask,
        aeFileProc *proc, void *clientData)
{
    if (fd >= eventLoop->setsize) {
        errno = ERANGE;
        return AE_ERR;
    }
    aeFileEvent *fe = &eventLoop->events[fd];
    
    // epoll_ctl添加监控的文件描述符
    if (aeApiAddEvent(eventLoop, fd, mask) == -1)
        return AE_ERR;

    // 指定回调函数
    fe->mask |= mask;
    if (mask & AE_READABLE) fe->rfileProc = proc;
    if (mask & AE_WRITABLE) fe->wfileProc = proc;

    // 指定读写缓冲区
    fe->clientData = clientData;
    if (fd > eventLoop->maxfd)
        eventLoop->maxfd = fd;
    return AE_OK;
}

// src/ae_epoll.c
static int aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask) {
    aeApiState *state = eventLoop->apidata;
    struct epoll_event ee = {0}; /* avoid valgrind warning */
    /* If the fd was already monitored for some event, we need a MOD
     * operation. Otherwise we need an ADD operation. */
    int op = eventLoop->events[fd].mask == AE_NONE ?
            EPOLL_CTL_ADD : EPOLL_CTL_MOD;

    ee.events = 0;
    mask |= eventLoop->events[fd].mask; /* Merge old events */
    if (mask & AE_READABLE) ee.events |= EPOLLIN;
    if (mask & AE_WRITABLE) ee.events |= EPOLLOUT;
    ee.data.fd = fd;
    if (epoll_ctl(state->epfd,op,fd,&ee) == -1) return -1;
    return 0;
}
```

最后是通过epoll\_wait来获取事件，上文我们提到，在每次主循环中，首先根据最近到达的时间事件来计算epoll\_wait的超时时间，然后调用epoll\_wait获取事件，再处理事件，其中获取事件在函数aeApiPoll（src/ae\_epoll.c）中。

```cpp
static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp) {
    aeApiState *state = eventLoop->apidata;
    int retval, numevents = 0;

    retval = epoll_wait(state->epfd,state->events,eventLoop->setsize,
            tvp ? (tvp->tv_sec*1000 + tvp->tv_usec/1000) : -1);
    if (retval > 0) {
        int j;

        numevents = retval;
        for (j = 0; j < numevents; j++) {
            int mask = 0;
            struct epoll_event *e = state->events+j;

            if (e->events & EPOLLIN) mask |= AE_READABLE;
            if (e->events & EPOLLOUT) mask |= AE_WRITABLE;
            if (e->events & EPOLLERR) mask |= AE_WRITABLE;
            if (e->events & EPOLLHUP) mask |= AE_WRITABLE;
            eventLoop->fired[j].fd = e->data.fd;
            eventLoop->fired[j].mask = mask;
        }
    }
    return numevents;
}
```

获取到事件后，主循环中会逐个调用事件的回调函数来处理事件。

### 读写事件的实现

写累了，有空补上……

如何使用vscode调试redis源码
-------------------

1.  首先进入redis项目根目录，执行

```text
make
```

编译出二进制程序

这一步有可能报错：

```text
Makefile:79: recipe for target 'jemalloc' failed
make[2]: *** [jemalloc] Error 126
make[2]: Leaving directory '/root/redis-5.0.8/deps'
Makefile:199: recipe for target 'persist-settings' failed
make[1]: [persist-settings] Error 2 (ignored)
    CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:10: fatal error: jemalloc/jemalloc.h: No such file or directory
 #include <jemalloc/jemalloc.h>
          ^~~~~~~~~~~~~~~~~~~~~
compilation terminated.
Makefile:257: recipe for target 'adlist.o' failed
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory '/root/redis-5.0.8/src'
Makefile:6: recipe for target 'all' failed
make: *** [all] Error 2
```

jemalloc是内存分配的一种更高效的实现，用于代替libc的默认实现。这里报错找不到jemalloc，我们只需要将其替换成libc默认实现就好：

```text
make MALLOC=libc
```

如果报错：

```text
release.c:36:10: fatal error: release.h: No such file or directory
 #include "release.h"
          ^~~~~~~~~~~
compilation terminated.
Makefile:257: recipe for target 'release.o' failed
make[1]: *** [release.o] Error 1
make[1]: Leaving directory '/root/redis-5.0.8/src'
Makefile:6: recipe for target 'all' failed
make: *** [all] Error 2
```

我们可以在src目录找到一个脚本名为mkreleasehdr.sh，其中包含创建release.h的逻辑，将报错信息网上翻可以发现有一行：

```text
sh: 1: ./mkreleasehdr.sh: Permission denied
```

看来是这个脚本没有执行权限，导致release.h没有成功创建，我们需要给这个脚本添加执行权限然后重新编译：

```text
chmod +x src/mkreleasehdr.sh
make MALLOC=libc
```

2\. 创建调试配置（vscode）

创建文件 .vscode/launch.json，并填入以下内容：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "redis-server",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/src/redis-server",
            "args": [
                "./redis.conf"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb"
        },
    ]
}
```

然后就可以进入调试页面打断点调试了，main函数在 src/server.c

<img src="https://pic3.zhimg.com/v2-0d1169046bc6c7b3c4fc1565f6220426\_b.jpg" data-caption="" data-size="normal" data-rawwidth="1840" data-rawheight="704" class="origin\_image zh-lightbox-thumb" width="1840" data-original="https://pic3.zhimg.com/v2-0d1169046bc6c7b3c4fc1565f6220426\_r.jpg"/>

![](https://pic3.zhimg.com/80/v2-0d1169046bc6c7b3c4fc1565f6220426_720w.webp)

本文转自 <https://zhuanlan.zhihu.com/p/701182318>，如有侵权，请联系删除。