Nginx ，是一个 

- **基于事件的Web 服务器**
- 处理请求是异步非阻塞的，低消耗，支撑高并发
- 反向代理服务器，实现负载均衡

cgi ：web 服务器会fork 一个新进程

`fastcgi`：web 服务器不会一个新进程，直接tcp交给启动就开启的进程。

#### Nginx 常用命令？

- 启动 `nginx` 。
- 停止 `nginx -s stop`  。
- 重载配置 `./sbin/nginx -s reload(平滑重启)` 或 `service nginx reload` 。
- 重载指定配置文件 `.nginx -c /usr/local/nginx/conf/nginx.conf` 。
- 查看 nginx 版本 `nginx -v` 。
- 检查配置文件是否正确 `nginx -t` 。
- 显示帮助信息 `nginx -h` 。

#### Nginx 常用配置？

```java
worker_processes  8; # 工作进程个数
worker_connections  65535; # 每个工作进程能并发处理（发起）的最大连接数（包含所有连接数）
error_log         /data/logs/nginx/error.log; # 错误日志打印地址
access_log      /data/logs/nginx/access.log; # 进入日志打印地址
log_format  main  '$remote_addr"$request" ''$status $upstream_addr "$request_time"'; # 进入日志格式

listen       80; # 监听端口
server_name  rrc.test.jiedaibao.com; # 允许域名
root  /data/release/rrc/web; # 项目根目录
index  index.php index.html index.htm; # 访问根文件
```

🦅 **Nginx 日志格式中的 `$time_local` 表示的是什么时间？其次，当我们从前到后观察日志中的 `$time_local` 时间时，有时候会发现时间顺序前后错乱的现象，请说明原因？**

`$time_local` ：在服务器里**请求开始写入本地的时间**。

因为请求发生时间有前有后，所以会时间顺序前后错乱。

#### 优点

非阻塞、高并发、高性能、高吞吐

BSD许可证 灵活性

#### **反向代理服务器**的优点

当中间层，隐藏源服务器的存在和特征

#### 正向代理：代理端代理的是客户端，VPN

#### 反向代理：代理端代理的是服务端

#### 场景

![image-20200928123210804](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928123220.png)

- api服务：数据服务应用场景简单，并发性能，tps（系统吞吐量）高于应用服务。js、lua、openResty
  - web防火墙
  - openresty解决的痛点是，nginx模块开发使用lua规范方便，不用C去开发，不需要特别熟悉nginx源码，可以很方便的开发出适合自己业务的高性能web Server模块，他也自带MySQL，redis，memcached等模块，再说到你说的数据上，那你把MySQL查询的数据放memcached，浏览器获取数据不就快了么

#### 请解释 Nginx 如何处理 HTTP 请求？

- 首先，Nginx 在启动时，会**解析配置文件**，得到需要监听的端口与 IP 地址，然后在 Nginx 的 **Master 进程**里面先初始化好这个**监控的Socket**。
- 然后，再 **fork出多个子进程**出来。之后，子进程会竞争新的连接。
- 此时，客户端就可以向 nginx 发起连接了。当客户端与nginx进行三次握手，与 nginx 建立好一个连接后。
- 此时，某一个子进程会 accept 成功，得到这个建立好的连接的 Socket ，然后创建 nginx 对连接的封装。
- 接着，设置**读写事件处理函数**，并添加读写事件来与客户端进行数据的交换。

- 最后，Nginx 或客户端来主动关掉连接，到此，一个连接就寿终正寝了。

### Nginx 是如何实现高并发的？

Nginx 不这样，每进来一个 request ，会有一个 worker 进程去处理。但不是全程的处理，**处理到可能发生阻塞的地方**，比如向上游（后端）服务器转发 request ，并等待请求返回。

那么，这个处理的 worker 不会这么傻等着，他会在发送完请求后，**注册一个事件**：“如果 upstream 返回了，告诉我一声，我再接着干”。

于是他就休息去了。此时，如果再有 request 进来，他就可以很快再按这种方式处理。而一旦**上游服务器返回了，就会触发这个事件**，worker 才会来接手，这个 request 才会接着往下走。

> 这就是为什么说，Nginx 基于事件模型。

由于 web server 的工作性质属于网络 IO 密集型应用，不算是计算密集型。

> 异步，非阻塞，使用 epoll ，和大量细节处的优化。也正是 Nginx 之所以然的技术基石。

### 请解释Nginx服务器上的Master和Worker进程分别是什么?

- nginx在启动后，会有一个master进程和多个worker进程。
- master进程主要用来**管理worker进程**，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程异常退出后，，会自动重新启动新的worker进程。
- 而基本的网络事件，则是放在worker进程中来处理了
- worker进程的个数是可以设置的，一般我们会设置与**机器cpu核数一致**，这里面的原因与nginx的进程模型以及事件处理模型是分不开的。nginx的进程模型，

### 为什么 Nginx 不使用多线程？

Apache: 创建多个进程或线程，而每个进程或**线程都会为其分配 cpu 和内存**（线程要比进程小的多，所以 worker 支持比 perfork 高的并发），并发过大会榨干服务器资源。

Nginx: 采用单线程来异步非阻塞处理请求(epoll)，不会为每个请求分配 cpu 和内存资源，**节省了大量资源**，同时也**减少了大量的 CPU 的上下文切换**。所以才使得 Nginx 支持更高的并发。

> Netty、Redis 基本采用相同思路。

#### 多进程

![image-20200928144356362](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928144400.png)

为了保持高可用，高可靠性。多线程共享地址空间，第三方引发地址空间段错误，地址越界，进程挂掉。

**进程间通信采用共享内存解决的。**

一个worker绑定一个cpu核，更好使用cpu核中的缓存，减少缓存失效的命中率。

![image-20200928145020276](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928145022.png)

reload   ==   kill -SIGHUB 9170 

kill -SIGTERM 16882

命令行的子命令，就是向`Master`进程**发送信号**

![image-20200928145530397](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928145533.png)

hub 重载日志文件

usr1做日志文件的切割

![image-20200928150532520](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928150534.png)

![image-20200928150614260](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928150615.png)

- worker_shutdown_TimeOut

升级Nginx，使用

![image-20200928151105436](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928151107.png)

优雅的关闭：针对HTTP请求

![image-20200928151448502](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928151449.png)

![image-20200928152111397](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928152112.png)

![image-20200928152347122](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928152349.png)

三次握手后，**才通知唤醒Nginx，Nginx从操作系统kernel中获取事件**

#### [epoll](https://zhuanlan.zhihu.com/p/63179839)

![image-20200928152830807](https://gitee.com//chenchong0817/picture/raw/master/Aaron/20200928152832.png)

- 每次取活跃链接，只会遍历**链表**，链表中**只存活跃**的链接，内核态和用户态只读取活跃的链表
- 建立链接成功后，在红黑树中建立事件。添加、删除、修改、都是logN的时间复杂度

