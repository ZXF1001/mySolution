[< 返回主页](../README.md)
# 网站服务搭建（以CentOS7为例）
---
## 1. 安装LNMP（Linux+Nginx+MySQL+PHP）
1. lnmp官网装lnmp环境
2. 查找nginx.conf路径：`find / -name nginx.conf`
3. 更改查找到的nginx配置文件：`vim /usr/local/nginx/conf/nginx.conf`
4. 把server大括号内里的默认网页根目录改成想要的根目录
5. 把网页拷进新的目录，访问域名，如果不成功，要加权限：`chmod -R 777 目录（如果涉及到上传文件的功能，必须赋予所有用户写入权限）`

## 2. 更改跨域访问设置
需求场景：本地写的vue项目想要访问远程服务器的资源，要设置允许跨域访问（CORS）：
1. nginx.conf下的server括号下加入以下内容：
    ```NGINX Conf
    add_header 'Access-Control-Allow-Methods' 'GET,OPTIONS,POST' always;
    add_header 'Access-Control-Allow-Credentials' 'true' always;
    add_header 'Access-Control-Allow-Origin' $http_origin always;
    add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type, X-Requested-With, Cache-Control' always;
    if ($request_method = OPTIONS ) { return 200; }
    ```
1. 重启ngxin：`service nginx reload`

## 3. 开放端口
1. 修改本机防火墙策略表：`vim /etc/sysconfig/iptables`
2. 插入：`-A INPUT -p tcp -m tcp --dport 1234 -j ACCEPT`，其中1234为要开放的端口
3. 重启防火墙：`service iptables restart`
4. 如果是云服务器，要开启云服务器供应商的防火墙
> 尽量通过nginx转发来访问端口上的服务，而不是直接开放端口

## 4. Nginx设置虚拟目录托管静态文件
需求场景：比如在浏览器输入：`http://ip地址/static/xxx.jpg`，nginx自动代理到资源文件夹`/home/Website/static/xxx.jpg`下
1. 在nginx.conf中添加location设置：
    ```NGINX Conf
    location ~ /static/ {
        root  /home/Website/;
        #或者alias /home/Website/geotiff也可以
        # autoindex on;#如果取消注释这行，就是开启文件目录形式
        }
    ```
2. 如果是远程调试，要记得加上[跨域访问请求头](##2-更改跨域访问设置)
3. 如果还是不成功，试试给目录添加权限：`chmod -R 777 /home/Website/static/`
4. 重启nginx：`service nginx reload`
  
## 5. Nginx托管api接口
需求场景:比如在浏览器输入：`http://ip地址/api/xxx`，nginx自动代理到api服务端口`http://ip地址:3000/api/xxx`
1. 在nginx.conf中添加location设置：
    ```NGINX Conf
    location ~ /api/ {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      #proxy_set_header X-Nginx-Proxy true;
      proxy_set_header Connection "";
      proxy_pass http://127.0.0.1:3000;
    }
    ```
1. 重启nginx:`service nginx reload`

## 6. Nginx托管websocket接口
需求场景:比如前端使用websocket服务：`ws://ip地址/ws/`，nginx自动代理到ws服务端口`http://ip地址:8082/`
1. 在nginx.conf中添加http设置：
    ```NGINX Conf
    #My WebSocket Proxy Config
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }
    ```
2. 添加server设置：
    ```NGINX Conf
    location /ws/ {
        proxy_pass http://127.0.0.1:8082;
        proxy_http_version 1.1;
        #以下配置添加代理头部：
        proxy_set_header Host $host; # 保留源信息
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
    ```
3. 重启nginx:`service nginx reload`