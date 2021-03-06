# Nginx 配置

## Nginx配置通用语法

### 块配置项
块配置项由一个块配置项名和一对大括号组成。

### 2.2.3 配置项的注释
注释以“#”开头。

## 2.3 Nginx 服务的基本配置
配置项按照用户使用时的预取功能分成以下4类：
* 用于调试、定位问题的配置项
* 正常运行的必备配置项
* 优化性能的配置项
* 事件类配置项

### 2.3.1 用于调试、定位问题的配置项
1. 是否以守护进程方式运行
    daemon on|off;
2. 是否以master/worker方式工作
    master_process on|off;
3. error日志设置
   error_log pathfile level;
   pathfile 默认为logs/error.log

   调试时，可以设置为标准输出 `error_log  /dev/stdout  notice;`, 同时在http 块或server块设置access日志 `access_log  /dev/stdout ;`
4. 特殊调试点
    debug_points[stop|abort]

5. 仅对指定的客户端输出debug级别日志
    debug_connection[IP|CIDR]
    ```
    events {
        debug_connection 10.224.66.14;
        debug_connection 10.224.57.0/24;
    }
    ```
6. 限制coredump核心转储文件的大小
   work_rlimit_core size;
7. coredump文件生成目录
    working_directory path;

### 2.3.2 正常运行的配置项
1. 定义环境变量
   env VAR|VAR=VALUE;
    __怎么用？？？__

2. 嵌入其他配置文件
   include pathfile;

3. pid文件路径
    pid path/file;

4. Nginx worker进程运行的用户及用户组
    user uaername[groupname];

5. 指定Nginx worker进程可以打开的最大文件句柄描述符个数
    worker_rlimit_nofile limit;

6. 限制信号队列
    worker_rlimit_sigpending  limit;

### 2.3.3 优化性能的配置项
1. Nginx worker进程个数
    worker_processes number;

2. 绑定Nginx worker进程到指定的CPU
    worker_cpu_affinity cpumask [cpumask...];
    ```
    worker_processes 4;
    worker_cpu_affinity 1000 0100 0010 0001;
    ```
3. SSL硬件加速
    ssl_engine device;
    openssl engine -t 可查看是否有SSL硬件加速设备

4. gettimeofday执行频率
    timer_resolution t;
    一般不需要配置。

5. Nginx worker进程优先级设置
    worker_priority nice;

### 2.3.4 事件类配置项
1. 是否打开accept锁
    accept_mutex [on|off];

2. lock文件的路径
    lock_file path/file;

3. 使用accept锁后到真正建立连接之间的延迟时间
    accept_mutex_delay Nms;
    默认500ms
    ***如何保证任一时刻必然有一个进程监听listen fd? 当前监听进程释放锁后通知其他进程？？***
4. 批量建立新连接
    mylti_accept [on|off]
    ***代码实现？***

5. 选择时间模型
    use epoll;

6. 每个worker的最大连接数
    worker_connection number;

## 2.4 用HTTP核心模块配置一个静态Web服务器
一个典型的静态web服务器可能会包含多个server块和localtion 块。  
静态web服务器的部分功能：虚拟主机与请求分发；文件路径定义；内存及磁盘资源的分配；网络连接设置；MIME类型的设置；客户端请求限制；文件操作优化；对客户端请求的特殊处理。

### 2.4.1 虚拟主机与请求的分发
可以实现多个主机域名共同使用一个IP地址。
配置多个server块，指定不同的server_name， listen 可以相同也可不同。

location
```
location = / {
    #只有当用户请求是"/"时，才会使用该location下的配置
    …
}
```

### 2.4.2文件路径的定义
#### (1) 以root方式设置资源路径
```
location /download/ {
    root optwebhtml;
}
```
如果有一个请求的URI是/download/index/test.html, 那么web服务器将会返回服务器上PREFIX/optwebhtml/download/index/test.html文件的内容。

#### (2) 以alisa方式设置资源路径
alias 会将用户请求映射到真正的磁盘文件上。例如用户请求的URI是/conf/nginx.conf， 而用户实际想访问的文件在/usr/local/nginx-1.5.9/conf/nginx.conf, 那么可以采用如下配置：
```
location /conf {
    alias   /usr/local/nginx-1.5.9/conf;
}
```
nginx是将location path 替换为 alias path

#### (3) 访问首页
index file...;

#### (4) 根据HTTP返回码重定向页面
对于某个请求返回错误码时，如果匹配上了error_page中设置的code,则重定向到新的URI中。这里的重定向不是只代表重定向到某个url, 本地的某个文件也属于新的URI。
```
error_page 404 404.html;
error_page 502 503 504 50x.html;
error_page 403 http://example.com/forbidden.html; 
error_page 404 = @fetch;
```
可以修改返回的code.

#### (5) 是否允许递归使用error_page

#### (6) try_files

### 2.4.3 内存及磁盘资源的分配

### 2.4.4 网络连接设置

### 2.4.5

## 2.5 用HTTP proxy module 配置一个反向代理服务器
当客户端发来HTTP请求，Nginx不会立刻转发到上游服务器，而是先把用户的请求（包括HTTP包体)完整地接收到Nginx所在服务器的硬盘或者内存中，然后再向上游服务器发起连接，把缓存的客户端请求转发到上游服务器。
优点：  
缺点：  
