# Emby播放分离反代示例

1、Misakaf.conf

这是关于MisakaF Emby 公费服/机场服的反代基础配置
（补充：pilipili的结构与misakaf一致）

```
# debian
cd /etc/nginx/conf.d/
nano misakaf.conf

#将配置文件内容粘贴进去，并修改对应的`#注释`区域，保存并退出

nginx -t
systemctl reload nginx
```
2、embyplus.conf

**纸片人已于2025-01-17修改为单次重定向，与misakaF一致，以下逻辑废弃**

~~这是关于embyplus（纸片人） Emby 公益服的反代基础配置~~
~~- 由于目前纸片人有单独的推流线路，所以需要2次重定向~~
~~- 你在纸片人bot中选的每一条线路相对的推流域名都不同，但仅仅是线路推流域名不同（即第二次重定向），默认推流域名（即第一次重定向）是不变的~~
- 建议在纸片人bot中选择默认线路，cf线路不论你使用那个区域的反代vps都能获得相对较好的体验
```
# debian
cd /etc/nginx/conf.d/
nano embyplus.conf

#将配置文件内容粘贴进去，并修改对应的`#注释`区域，保存并退出

nginx -t
systemctl reload nginx
```
# 补充 by wlin
在推流的localtion(比如location /s1)里面加上 
```
proxy_cache off; 
proxy_request_buffering off; 
tcp_nodelay on; 
proxy_set_header Range $http_range; 
proxy_set_header If-Range $http_if_range;
```

在根location / 里面加上
```
sub_filter_once off; 
sub_filter_types text/html text/css application/javascript application/json application/xml;

sub_filter '推流域名:端口/' '自己反代域名:端口/s1/';
sub_filter '推流域名:端口' 'http://wlin.ddns.net:8011/s1';  # 无斜杠
```
完整配置，http的
```
map $http_upgrade $connection_upgrade {
   default upgrade;
   ''      close;
}

server {
    listen 反代端口;
    # 你的域名
    server_name 反代域名;
    # 你的证书
    # ssl_certificate /root/cert/example.com.cer;
    # ssl_certificate_key /root/cert/example.com.key;

    client_max_body_size 20M;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    location / {
        # 你需要反代的emby服务器域名
        proxy_pass http://面板域名:面板端口/;
        # 你需要反代的emby推流地址
        proxy_redirect http://推流域名:推流端口/ http://反代域名:反代端口/s1/;
        # 你需要反代的emby服务器主页
        proxy_set_header Referer "http://面板域名:面板端口/web/index.html"; 
        proxy_set_header Upgrade $http_upgrade; 
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $proxy_host; 
        # proxy_ssl_server_name on; 
        proxy_http_version 1.1;
        
        proxy_cache off; 

        sub_filter_once off; 
        sub_filter_types text/html text/css application/javascript application/json application/xml;

        sub_filter 'http://推流域名:推流端口/' 'http://反代域名:反代端口/s1/';
        sub_filter 'http://推流域名:推流端口' 'http://反代域名:反代端口/s1';  # 无斜杠
    }

    location /s1 {
        rewrite ^/s1(/.*)$ $1 break;
        # 你需要反代的emby推流地址
        proxy_pass http://推流域名:推流端口/;
        proxy_set_header Referer "http://面板域名:面板端口/web/index.html";
        proxy_set_header Host $proxy_host;
        # proxy_ssl_server_name on;
        proxy_buffering off;

        proxy_cache off; 
        proxy_request_buffering off; 
        tcp_nodelay on; 
        proxy_set_header Range $http_range; 
        proxy_set_header If-Range $http_if_range;
    }
}
```
