#### Nginx

## Install

```shell
sudo yum install epel-release
sudo yum install nginx
```

## Start nginx

1. 关闭防火墙（如果有）

   ```shell
   sudo firewall-cmd --permanent --zone=public --add-service=http 
   sudo firewall-cmd --permanent --zone=public --add-service=https
   sudo firewall-cmd --reload
   ```

2. 设置开机启动

   ```shell
   sudo systemctl enable nginx
   ```

3. 启动nginx

   ```shell
   sudo systemctl start nginx
   ```

## Reload

```shell
nginx -s reload #stop, quit, reopen, reload
```

## Nginx 配置（config）

nginx的全局配置文件路径： `/etc/nginx/nginx.conf`

nginx配置官网样例：[配置样例](https://www.nginx.com/resources/wiki/start/topics/examples/full/)



### 	location配置

location语法规则：

```nginx
location [=|~|~*|^~] /uri/ { … }
```

其中：

```nginx
------------------------------------
[留空] | 匹配uri但是优先级低
------------------------------------
=     | 标识精确匹配
------------------------------------
^~    | 以某字符串开头
------------------------------------
~     | 区分大小写的正则匹配
------------------------------------
~*    | 不区分大小写的正则匹配
------------------------------------
!~    | ~取反
------------------------------------
!~*   | ~*取反
------------------------------------
```

优先级为：

```nginx
(location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)
```



### Proxy 配置

proxy.conf

```nginx
proxy_redirect          off;
proxy_set_header        Host            $host;
proxy_set_header        X-Real-IP       $remote_addr;
proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size    10m;
client_body_buffer_size 128k;
proxy_connect_timeout   90;
proxy_send_timeout      90;
proxy_read_timeout      90;
proxy_buffers           32 4k;
```

