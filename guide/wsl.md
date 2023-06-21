[< 返回主页](../README.md)
# WSL2相关操作指南
---
## 1. WSL2不通过微软商店安装Linux发行版

## 2. WSL2导入/导出tar包
1. 导出tar包
    ```powershell
    wsl --export <distro_name> <path_to_save_tar>
    ```
2. 导入tar包
    ```powershell
    wsl --import <distro_name> <path_to_install> <path_to_tar>
    ```
## 3. WSL2移动Linux的安装位置
通过第二点的方法移动到新的位置即可

## 4. WSL2修改默认登陆用户
1. 在wsl的linux系统中创建`/etc/wsl.conf`文件，内容如下:
    ```bash
    [user]
    default=your_username
    ```
2. 退出wsl子系统，停止该子系统
    ```powershell
    wsl -t your_distro_name
    ```
3. 重新启动该子系统，即可使用新的默认用户登陆
    ```powershell
    wsl -d your_distro_name
    ```
## 4. WSL2配置代理
1. 在wsl的linux系统home目录下创建`.proxy_rc`文件，内容如下:
    ```bash
    #!/bin/bash
    host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
    # 11223为windows宿主机的代理软件端口
    export ALL_PROXY="http://$host_ip:11223"
    ```
2. 需要使用代理的时候，执行`source ~/.proxy_rc`即可

