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



