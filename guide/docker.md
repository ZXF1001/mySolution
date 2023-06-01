[< 返回主页](../README.md)
# Docker相关操作
---
## 1. 安装
1. 创建一个新的docker用户组，将当前用户添加到docker用户组
    ```bash
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```
2. 退出当前终端并重新登录以刷新用户组设置。
3. 检查用户是否已经添加到Docker用户组：
    ```bash
    id -nG
    ```
4. 如果有必要，更新apt软件仓库：
    ```bash
    sudo apt update
    ```
5. 安装Docker的依赖软件包：
    ```bash
    sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    ```
6. 添加Docker的GPG密钥
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```
7. 将Docker软件源添加到Ubuntu 18.04的apt软件仓库
    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```
8. 再次更新apt软件仓库
    ```bash
    sudo apt update
    ```
9. 安装Docker CE
    ```bash
    sudo apt install docker-ce
    ```
10. 启动Docker服务
    ```bash
    sudo systemctl status docker
    # 或者
    service docker status
    ```
11. 如果Docker服务没有启动，则可以使用以下命令启动：
    ```bash
    sudo systemctl start docker
    # 或者
    service docker start
    ```
现在，非root用户也可以使用Docker了。

12. 更换国内源
    ```bash
    sudo vim /etc/docker/daemon.json
    ```
    在daemon.json中写入以下内容：
    ```json
    {
        "registry-mirrors": ["http://hub-mirror.c.163.com/"]
    }
    ```
    保存退出后重启docker服务：
    ```bash
    sudo service doocker restart
    ```
    检查是否生效：
    ```bash
    docker info|grep Mirrors -A 1
    ```
## 2. 创建网络
1. 创建一个新的网络
    ```bash
    docker network create app-network
    ```
2. 查看网络
    ```bash
    docker network ls
    ```
3. 创建一个公共文件夹
    ```bash
    mkdir -p ~/Docker/public
    ```
## 3.1 手动部署nginx-1.22
1. 拉取nginx镜像
    ```bash
    docker pull nginx:1.22
    ```
2. 新建文件夹用来存放nginx配置文件
    ```bash
    mkdir -p ~/Docker/nginx/conf
    mkdir -p ~/Docker/nginx/log
    mkdir -p ~/Docker/nginx/html
    ```
3. 先建一个容器，把其中的设置文件和静态网页拷贝出来
    ```bash
    cd ~/Docker/nginx
    docker run --name c_nginx -p 80:80 -d nginx:1.22
    docker cp c_nginx:/etc/nginx/nginx.conf ~/Docker/nginx/conf
    docker cp c_nginx:/etc/nginx/conf.d ~/Docker/nginx/conf
    docker cp c_nginx:/usr/share/nginx/html ~/Docker/nginx
    ```
4. 停止并删除容器
    ```bash
    docker stop c_nginx
    docker rm c_nginx
    ```
5. 启动nginx并加入网络
    ```bash
    docker run --name c_nginx \
    -p 80:80 \
    --network app-network \
    --network-alias nginx \
    -e TZ=Asia/Shanghai \
    -v ~/Docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v ~/Docker/nginx/conf/conf.d:/etc/nginx/conf.d \
    -v ~/Docker/nginx/html:/usr/share/nginx/html \
    -v ~/Docker/nginx/log:/var/log/nginx \
    -v ~/Docker/public:/usr/share/nginx/public \
    -d nginx:1.22
    ```
6. 查看nginx是否启动成功
    ```bash
    # 查看宿主机ip
    ifconfig
    ```
    在浏览器中输入`http://宿主机ip`，如果出现`Welcome to nginx!`则说明启动成功。

7. 编辑设置文件，代理静态资源
   打开`~/Docker/nginx/conf/conf.d/default.conf`，在`location /`的大括号以下添加以下内容：
    ```bash
    location ~ (/geotiff/|/streamline/|/DEM/|/contour/) {
        root   /usr/share/nginx/public;
        autoindex on;
        # CORS config
        add_header 'Access-Control-Allow-Methods' 'GET,OPTIONS,POST' always;
        add_header 'Access-Control-Allow-Credentials' 'true' always;
        add_header 'Access-Control-Allow-Origin' $http_origin always;
        add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type, X-Requested-With, Cache-Control' always;
        if ($request_method = OPTIONS ) { return 200; }
    }
    ```
    保存退出后重启nginx服务：
    ```bash
    docker exec -it c_nginx bash
    nginx -s reload
    ```
    在~/Docker/public下创建/DEM文件夹，访问`http://宿主机ip/DEM/`，如果看见/DEM目录下的文件列表则说明成功。

## 3.2 使用Dockerfile部署nginx-1.22
1. 在`~/Docker/nginx`下新建`conf`文件夹，用来存放nginx配置文件
    ```bash
    mkdir -p ~/Docker/nginx/conf
    ```
    在conf/conf.d里放入`default.conf`文件，在conf里放入`nginx.conf`文件
3. 在`~/Docker/nginx`下新建`html`文件夹，放入你的静态网页
    ```bash
    mkdir -p ~/Docker/nginx/html
    ```
4. 在`~/Docker/nginx`下新建`Dockerfile`文件，内容如下：
    ```bash
    FROM nginx:1.22
    RUN mkdir -p /usr/share/nginx/public
    COPY ./conf/nginx.conf /etc/nginx/nginx.conf
    COPY ./conf/conf.d/ /etc/nginx/conf.d/
    COPY ./html/ /usr/share/nginx/html/
    EXPOSE 80
    ```
5. 构建镜像
    ```bash
    docker build -t my_nginx ~/Docker/nginx
    ```
6. 启动nginx并加入网络
    ```bash
    docker run --name c_nginx \
    -p 80:80 \
    --network app-network \
    --network-alias nginx \
    -e TZ=Asia/Shanghai \
    -v ~/Docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v ~/Docker/nginx/conf/conf.d:/etc/nginx/conf.d \
    -v ~/Docker/nginx/html:/usr/share/nginx/html \
    -v ~/Docker/nginx/log:/var/log/nginx \
    -v ~/Docker/public:/usr/share/nginx/public \
    -d my_nginx
    ```
7. 如果有更新设置，更新设置后，重启nginx服务
    ```bash
    docker exec -it c_nginx bash
    nginx -s reload
    ```

## 4. 部署mysql-5.7.38
1. 拉取mysql镜像
    ```bash
    docker pull mysql:5.7.38
    ```
2. 新建文件夹用来存放mysql配置文件和数据
    ```bash
    mkdir -p ~/Docker/mysql/conf
    mkdir -p ~/Docker/mysql/data
    mkdir -p ~/Docker/mysql/log
    ```
3. 先建一个容器，把其中的设置文件拷贝出来
    ```bash
    docker run --name c_mysql \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -d mysql:5.7.38
    
    docker cp c_mysql:/etc/my.cnf ~/Docker/mysql/conf
    ```
4. 停止并删除容器
    ```bash
    docker stop c_mysql
    docker rm c_mysql
    ```
5. 启动mysql，并加入网络
   ```bash
    docker run -d \
    -p 3306:3306 \
    --name=c_mysql \
    --network app-network \
    --network-alias mysql \
    -v ~/Docker/mysql/log:/var/log/mysql \
    -v ~/Docker/mysql/data:/var/lib/mysql \
    -v ~/Docker/mysql/conf/my.cnf:/etc/my.cnf \
    -e MYSQL_ROOT_PASSWORD=123456 \
    -e TZ=Asia/Shanghai \
    mysql:5.7.38
    ```
6. 查看mysql是否启动成功
    ```bash
    docker exec -it c_mysql /bin/sh
    mysql -uroot -p
    ```
    这一步第一次可能不成功，退出重新进入就能成功。如果出现`mysql>`则说明启动成功。

7. 禁止远程登录root，并创建远程用户
    ```SQL
    USE mysql;
    SELECT user,host FROM user;
    DELETE user FROM mysql.user WHERE user='root' AND host='%';

    CREATE USER 'zxf'@'%'  IDENTIFIED BY 'zxf451001';
    CREATE DATABASE `zxf` CHARACTER SET 'utf8mb4';
    GRANT ALL PRIVILEGES ON zxf.* TO 'zxf'@'%';
    FLUSH PRIVILEGES;
    ```
8. 修改mysql配置文件
    ```bash
    vim ~/Docker/mysql/conf/my.cnf
    ```
    在`[client]`、`[mysql]`、`[mysqld]`下添加以下内容：
    ```bash
    [client]
    default-character-set=utf8mb4

    [mysql]
    default-character-set=utf8mb4

    [mysqld]
    init_connect="SET collation_connection = utf8mb4_unicode_ci"
    init_connect="SET NAMES utf8mb4"
    character-set-server=utf8mb4
    collation-server=utf8mb4_unicode_ci
    skip-character-set-client-handshake
    ```
9. 重启mysql
    ```bash
    docker restart c_mysql
    ```
## 4.1 导入数据库
1. 将数据库文件拷贝到`~/Docker/mysql/data`下
## 5. 部署express后端服务（node-18.14.2）
1. 创建文件夹
    ```bash
    mkdir -p ~/Docker/express
    ```
2. 新建Dockerfile
    ```bash
    vim ~/Docker/express/Dockerfile
    ```
    写入以下内容：
    ```Dockerfile
    FROM node:18.14.2
    COPY ./app /app
    WORKDIR /app
    RUN npm install
    EXPOSE 3000
    CMD npm run start > express.log 2>&1
    ```
3. 将项目拷贝到`~/Docker/express/app`下
4. 构建镜像
    ```bash
    docker build -t my_express ~/Docker/express
    ```
5. 启动容器，并加入网络
    ```bash
    docker run -d \
    --name c_express \
    -e TZ=Asia/Shanghai \
    --network app-network \
    --network-alias express \
    my_express
    ```
    如果进去就退出，可以把-d改成-it，去容器内部命令行检查问题原因

## 6. 部署node应用（qweather）
1. clone项目到`~/Docker/service/qweather`下
    ```bash
    git clone https://gitee.com/zxf001/wind-platform-service.git
    ```
    并将文件夹重命名为app
2. 将package.json中的nodemon改成node
3. 新建Dockerfile
    ```bash
    vim ~/Docker/service/qweather/Dockerfile
    ```
    写入以下内容：
    ```Dockerfile
    FROM node:18.14.2
    COPY ./app /app
    WORKDIR /app
    RUN npm install
    CMD npm run start > qweather.log 2>&1
    ```
4. 构建镜像
    ```bash
    docker build -t my_qweather ~/Docker/service/qweather
    ```
5. 启动容器，并加入网络
    ```bash
    docker run -d \
    --name c_qweather \
    --network app-network \
    --network-alias qweather \
    -e TZ=Asia/Shanghai \
    my_qweather
    ```
    如果进去就退出，可以把-d改成-it，去容器内部命令行检查问题原因

## 7. 部署python应用
1. 项目拷贝到`~/Docker/service/realtimeData`下
    
    并将文件名重命名为app.py
2. 新建Dockerfile
    ```bash
    vim ~/Docker/service/realtimeData/Dockerfile
    ```
    写入以下内容：
    ```Dockerfile
    FROM python:3.6.9
    RUN mkdir /runtime
    ADD .  /runtime
    WORKDIR /runtime
    RUN pip3 install -r requirements.txt -i https://mirrors.zju.edu.cn/pypi/web/simple
    CMD python app.py > ./realtimeData.log 2>&1
    ```
3. 构建镜像
    ```bash
    docker build -t my_realtimedata ~/Docker/service/realtimeData
    ```
4. 启动容器，并加入网络
    ```bash
    docker run -d \
    --name c_realtimedata \
    --network app-network \
    --network-alias realtimedata \
    -e TZ=Asia/Shanghai \
    my_realtimedata
    ```
    如果进去就退出，可以把-d改成-it，去容器内部命令行检查问题原因

## 7. 部署flask框架
总体和部署python应用一样，只是导出requirements.txt的时候注意全部使用版本号的形式。
1. 在开发环境导出requirements.txt
    ```bash
    pip list --format=freeze > requirements.txt
    ```
    其中，要把distribute，pip，setuptools，wheel这几个包删除，还要把pywin32这个删除。
2. 项目拷贝到`~/Docker/flask`下
    并将文件夹重命名为app
3. 创建日志文件夹
    ```bash
    mkdir -p ~/Docker/flask/log
    ```
4. 新建Dockerfile
    ```bash
    vim ~/Docker/flask/Dockerfile
    ```
    写入以下内容：
    ```Dockerfile
    FROM python:3.10.9
    RUN mkdir -p /runtime/app
    RUN mkdir -p /runtime/log
    COPY requirements.txt /runtime
    RUN pip3 install -r /runtime/requirements.txt -i https://mirrors.zju.edu.cn/pypi/web/simple
    COPY app /runtime/app
    EXPOSE 5000
    WORKDIR /runtime/app
    CMD python /runtime/app/app.py > /runtime/log/flask.log 2>&1
    ```
5. 构建镜像
    ```bash
    docker build -t my_flask ~/Docker/flask
    ```
6. 启动容器，并加入网络
    ```bash
    docker run -d \
    --name c_flask \
    --network app-network \
    --network-alias flask \
    -e TZ=Asia/Shanghai \
    -v ~/Docker/flask/log:/runtime/log \
    my_flask
    ```
    如果进去就退出，可以把-d改成-it，去容器内部命令行检查问题原因
