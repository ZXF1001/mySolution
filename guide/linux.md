[< 返回主页](../README.md)
# Linux相关操作记录(以CentOS-7为例)
---
## 1. 服务如何常驻、终止
1. 通过screen指令可以让服务在后端长时间运行：
    ```bash
    screen -S name # 以name为代号开一个进程
    screen -r name # 重新打开name为代号的进程
    ```
1. 或者通过nohup指令后台运行
    ```bash
    nohup command > log.command 2>&1 &
    ```
2. 终止后台运行的nodemon进程
    ```bash
    ps -ef | grep nodemon #查找名为nodemon的进程,记下进程号
    kill 12345 #杀掉进程
    ```

## 2. 本地客户端免密登录远程服务器
[参考这篇文章](https://blog.csdn.net/qq_40451749/article/details/89348799)

## 3. tar打包和解压
- 打包：`tar -czvf name.tar.gz dir`，其中`c`表示打包，`z`表示使用gzip压缩，`v`表示显示打包过程，`f`表示指定打包后的文件名
- 解压tar.gz文件：`tar -xzvf name.tar.gz`，其中`x`表示解压，`z`表示使用gzip解压，`v`表示显示解压过程，`f`表示指定解压的文件名