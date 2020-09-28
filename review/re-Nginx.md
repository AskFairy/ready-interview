Nginx ，是一个 

- 基于事件的Web 服务器
- 处理请求是异步非阻塞的，低消耗，支撑高并发
- 反向代理服务器，实现负载均衡

cgi ：web 服务器会fork 一个新进程

fastcgi：web 服务器不会一个新进程，直接tcp交给启动就开启的进程。

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

🦅 **Nginx 日志格式中的 `$time_local` 表示的是什么时间？请求开始的时间？请求结束的时间？其次，当我们从前到后观察日志中的 `$time_local` 时间时，有时候会发现时间顺序前后错乱的现象，请说明原因？**

`$time_local` ：在服务器里请求开始写入本地的时间。

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

- api服务：数据服务应用场景简单，并发性能，tps高于应用服务。js、lua、openResty
  - web防火墙

#### 请解释 Nginx 如何处理 HTTP 请求？

- 首先，Nginx 在启动时，会解析配置文件，得到需要监听的端口与 IP 地址，然后在 Nginx 的 Master 进程里面先初始化好这个监控的Socket(创建 S ocket，设置 addr、reuse 等选项，绑定到指定的 ip 地址端口，再 listen 监听)。

- 然后，再 fork(一个现有进程可以调用 fork 函数创建一个新进程。由 fork 创建的新进程被称为子进程 )出多个子进程出来。

- 之后，子进程会竞争 accept 新的连接。此时，客户端就可以向 nginx 发起连接了。当客户端与nginx进行三次握手，与 nginx 建立好一个连接后。此时，某一个子进程会 accept 成功，得到这个建立好的连接的 Socket ，然后创建 nginx 对连接的封装，即 ngx_connection_t 结构体。

- 接着，设置读写事件处理函数，并添加读写事件来与客户端进行数据的交换。

  > 这里，还是有一些逻辑，继续在 [「Nginx 是如何实现高并发的？」](http://svip.iocoder.cn/Nginx/Interview/#) 问题中来看。

- 最后，Nginx 或客户端来主动关掉连接，到此，一个连接就寿终正寝了。

🦅 **Nginx 是如何实现高并发的？**

如果一个 server 采用一个进程(或者线程)负责一个request的方式，那么进程数就是并发数。那么显而易见的，就是会有很多进程在等待中。等什么？最多的应该是等待网络传输。其缺点胖友应该也感觉到了，此处不述。

> 思考下，Java 的 NIO 和 BIO 的对比哟。

而 Nginx 的异步非阻塞工作方式正是利用了这点等待的时间。在需要等待的时候，这些进程就空闲出来待命了。因此表现为少数几个进程就解决了大量的并发问题。

Nginx是如何利用的呢，简单来说：同样的 4 个进程，如果采用一个进程负责一个 request 的方式，那么，同时进来 4 个 request 之后，每个进程就负责其中一个，直至会话关闭。期间，如果有第 5 个request进来了。就无法及时反应了，因为 4 个进程都没干完活呢，因此，一般有个调度进程，每当新进来了一个 request ，就新开个进程来处理。

> 回想下，BIO 是不是存在酱紫的问题？嘻嘻。

Nginx 不这样，每进来一个 request ，会有一个 worker 进程去处理。但不是全程的处理，处理到什么程度呢？处理到可能发生阻塞的地方，比如向上游（后端）服务器转发 request ，并等待请求返回。那么，这个处理的 worker 不会这么傻等着，他会在发送完请求后，注册一个事件：“如果 upstream 返回了，告诉我一声，我再接着干”。于是他就休息去了。此时，如果再有 request 进来，他就可以很快再按这种方式处理。而一旦上游服务器返回了，就会触发这个事件，worker 才会来接手，这个 request 才会接着往下走。

> 这就是为什么说，Nginx 基于事件模型。

由于 web server 的工作性质决定了每个 request 的大部份生命都是在网络传输中，实际上花费在 server 机器上的时间片不多。这是几个进程就解决高并发的秘密所在。即：

> webserver 刚好属于网络 IO 密集型应用，不算是计算密集型。

而正如叔度所说的

> 异步，非阻塞，使用 epoll ，和大量细节处的优化。

也正是 Nginx 之所以然的技术基石。

🦅 **为什么 Nginx 不使用多线程？**

Apache: 创建多个进程或线程，而每个进程或线程都会为其分配 cpu 和内存（线程要比进程小的多，所以 worker 支持比 perfork 高的并发），并发过大会榨干服务器资源。

Nginx: 采用单线程来异步非阻塞处理请求（管理员可以配置 Nginx 主进程的工作进程的数量）(epoll)，不会为每个请求分配 cpu 和内存资源，节省了大量资源，同时也减少了大量的 CPU 的上下文切换。所以才使得 Nginx 支持更高的并发。

> Netty、Redis 基本采用相同思路。