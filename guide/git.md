[< 返回主页](../README.md)
# Git相关
---
## 1. Github被墙时，命令行环境下Git的代理配置
就算挂了梯子，用命令行提交到git的时候也没有用，需要设置命令行代理：
1. 挂梯子
2. 设置命令行代理：
    ```Powershell
    #端口号11223是梯子的端口号
    git config --global http.proxy http://127.0.0.1:11223
    git config --global https.proxy http://127.0.0.1:11223

    #下两行可选，我也不知道是什么
    git config --global http.proxy 'socks5://127.0.0.1:11223'
    git config --global https.proxy 'socks5://127.0.0.1:11223'

    # # 输入下面两行来取消代理
    # git config --global --unset http.proxy
    # git config --global --unset https.proxy
    ```