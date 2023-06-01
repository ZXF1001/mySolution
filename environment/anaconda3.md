[< 返回主页](../README.md)
# Anaconda3配置
---
## 1. 下载安装包
由于服务器连接的是内网，可以在浙大镜像源下载Anaconda，下载地址：[https://mirrors.zju.edu.cn/docs/anaconda/](https://mirrors.zju.edu.cn/docs/anaconda/)，在**安装映像**里选取对应的架构版本（架构查看方法：`uname -a`，192小服务器和13大集群是x86_64，所以选择`Linux_x86_64.sh`后缀的版本），右键复制下载链接
```bash
mkdir ~/Programs && cd ~/Programs
# 下面这个链接替换为复制的最新版的anaconda的链接
wget https://mirrors.zju.edu.cn/anaconda/archive/Anaconda3-2023.03-Linux-x86_64.sh
```
## 2. 安装
```bash
# 下面这个文件名要替换为下载的文件名
chmod +x Anaconda3-2023.03-Linux-x86_64.sh
./Anaconda3-2023.03-Linux-x86_64.sh
# 安装过程中要选择安装位置，可以选在你的用户目录下的Program文件夹下
# 安装完成后，问你要不要init，选yes
```
## 3. 换源
如果用户目录下没有.condarc文件的话，先要用这句命令创建.condarc文件
```bash
conda config --set show_channel_urls yes
```
修改~/.condarc文件为：
```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.zju.edu.cn/anaconda/pkgs/main
  - https://mirrors.zju.edu.cn/anaconda/pkgs/r
  - https://mirrors.zju.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.zju.edu.cn/anaconda/cloud
  msys2: https://mirrors.zju.edu.cn/anaconda/cloud
  bioconda: https://mirrors.zju.edu.cn/anaconda/cloud
  menpo: https://mirrors.zju.edu.cn/anaconda/cloud
  pytorch: https://mirrors.zju.edu.cn/anaconda/cloud
  pytorch-lts: https://mirrors.zju.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.zju.edu.cn/anaconda/cloud
```
清除缓存并更新（可能耗时较久）
```bash
conda clean -i
conda update --all
```
修改pip源
```bash
pip config set global.index-url https://mirrors.zju.edu.cn/pypi/web/simple
```
