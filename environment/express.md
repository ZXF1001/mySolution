[< 返回主页](../README.md)
# Express后端框架配置
---
## 1. Express框架安装

1. 服务器安装node.js和npm（不要直接用apt-get或yum，安装，要去官网下载安装）
2. 创建后端项目文件夹，`npm init`初始化项目，一路回车
3. 按照Express教程引入Express包开始开发,详细参考[B站教程](https://www.bilibili.com/video/BV1mQ4y1C7Cn)
4. 可以通过[Nginx的代理功能](./web-server.md/##5-Nginx托管api接口)把`http://your.server.ip.address:xxxx/`的请求代理到`http://your.server.ip.address/api/`下，就避免了端口暴露的隐患
5. 开发api接口的时候可以用一些接口测试管理的工具，如Apifox、postman，用来做接口管理很方便

## 2. WebSocket的实现
1. 在node.js下引入nodejs-websocket包
2. 网上很多简单的起步教程
3. 前端页面通过`ws://your.server.ip.address:xxxx/`就能和服务器通信，也可以像Express一样通过nginx代理，但是ws协议需要通过nginx升级，具体见[Nginx的代理功能](./web-server.md/##6-Nginx托管websocket接口)