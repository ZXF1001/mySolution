[< 返回主页](../README.md)
# Ubuntu18.04下SOWFA安装

## 1. 安装OpenFOAM（要和下面的SOWFA版本对应）
### 安装OpenFOAM-6
1. 更新与安装依赖
```bash
  sudo apt-get update
  sudo apt-get upgrade
  sudo apt-get install -y build-essential cmake git libopenmpi-dev openmpi-bin libblas-dev liblapack-dev flex bison gdb valgrind g++ libxml2-dev libhdf5-serial-dev gfortran

```
2. 下载OpenFOAM源码
```bash
  mkdir ~/OpenFOAM && cd ~/OpenFOAM
  git clone https://github.com/OpenFOAM/OpenFOAM-6.git
  git clone https://github.com/OpenFOAM/ThirdParty-6.git
  # 将环境变量添加到.bashrc文件中
  echo "source $HOME/OpenFOAM/OpenFOAM-6/etc/bashrc" >> ~/.bashrc && source ~/.bashrc
```
3. 编译安装（需要较长时间）
```bash
  cd $WM_THIRD_PARTY_DIR && ./Allwmake

  cd $WM_PROJECT_DIR && ./Allwmake
```
4. 创建run文件夹
```
mkdir -p ~/OpenFOAM/$USER-6/run
```
#### 安装OpenFOAM-2.4.0
参考[这个页面](https://openfoamwiki.net/index.php/Installation/Linux/OpenFOAM-2.4.0/Ubuntu#Ubuntu_18.04)
## 2. 安装OpenFAST-2.4.0
1. 安装gfortran-10
```bash
  sudo apt-get install -y software-properties-common
  sudo add-apt-repository ppa:ubuntu-toolchain-r/test
  sudo apt-get update
  sudo apt-get install gfortran-10
```
1. 安装高版本的cmake
```bash
  mkdir ~/Programs && cd ~/Programs
  wget https://github.com/Kitware/CMake/releases/download/v3.26.3/cmake-3.26.3-linux-x86_64.sh
  chmod +x cmake-3.26.3-linux-x86_64.sh
  ./cmake-3.26.3-linux-x86_64.sh
  vim ~/.bashrc
  # 添加以下内容并保存<-
  export PATH=~/Programs/cmake-3.26.3-linux-x86_64/bin:$PATH
  # ->

  source ~/.bashrc
  # 安装完成查看版本是否为3.26.3
  cmake --version
  rm cmake-3.26.3-linux-x86_64.sh
```
1. 安装yaml-cpp
```bash
  cd ~/Programs
  git clone https://github.com/jbeder/yaml-cpp.git && cd yaml-cpp
  mkdir build && cd build
  cmake -DBUILD_SHARED_LIBS=ON ..
  make
  sudo make install
```

1. 下载openfast源码
```bash
  cd ~/Programs
  wget https://github.com/OpenFAST/openfast/archive/refs/tags/v2.4.0.tar.gz
  tar -zxvf v2.4.0.tar.gz && cd openfast-2.4.0
```

1. 编译安装
```bash
  mkdir build && cd build
  cmake .. -DBUILD_OPENFAST_CPP_API=ON -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release -DFPE_TRAP_ENABLED=ON
  
  make help
  make -j$(nproc)
  sudo make install
```
## 3. 安装SOWFA
### SOWFA-6
1. 下载源码
```bash
  cd ~/OpenFOAM/
  git clone -b dev https://github.com/pablo-benito/SOWFA-6.git && cd SOWFA-6
  # 官方最新的dev分支可能会有错，clone这个别人fork的仓库比较保险
```
1. 在用户home目录创建.sowfa_bashrc文件，内容如下：
```bash
#!/bin/bash
SOWFA-6()
{
   # Important locations.
   export inst_loc=$HOME/OpenFOAM
   export sowfa_loc=$HOME/OpenFOAM

   # Unset OpenFOAM environment variables.
   export OPENFOAM_VERSION=6
   if [ -z "$FOAM_INST_DIR" ]; then
      echo "Nothing to unset..."
   else
      echo "     *Unsetting OpenFOAM environment variables..."
      if [ -f "$FOAM_INST_DIR/OpenFOAM-$OPENFOAM_VERSION/etc/config.sh/unset" ]; then
         . $FOAM_INST_DIR/OpenFOAM-$OPENFOAM_VERSION/etc/config.sh/unset
      else
         . $FOAM_INST_DIR/OpenFOAM-$OPENFOAM_VERSION/etc/config/unset.sh
      fi
   fi

   # Set the OpenFOAM version and installation directory
   export OPENFOAM_VERSION=6
   export OPENFOAM_NAME=OpenFOAM-$OPENFOAM_VERSION
   export FOAM_INST_DIR=$inst_loc
   export WM_PROJECT_USER_DIR=/home/$USER/OpenFOAM/SOWFA-$OPENFOAM_VERSION
   export FOAMY_HEX_MESH=true

   # Source the OpenFOAM main environment.
   foamDotFile=$FOAM_INST_DIR/$OPENFOAM_NAME/etc/bashrc
   if [ -f $foamDotFile ] ; then
      echo "Sourcing $foamDotFile..."
      source $foamDotFile
   fi

   # For wmake compiling.
   export WM_NCOMPPROCS=12
   export WM_COLOURS="white blue green cyan red magenta yellow"

   # Alias to tutorials.
   alias tut='cd /home/$USER/OpenFOAM/SOWFA-$OPENFOAM_VERSION/exampleCases'

   # Set the SOWFA installation directory.
   export SOWFA_DIR=$sowfa_loc/SOWFA-$OPENFOAM_VERSION
   export SOWFA_APPBIN=$SOWFA_DIR/applications/bin/$WM_OPTIONS
   export SOWFA_LIBBIN=$SOWFA_DIR/lib/$WM_OPTIONS
   export HDF5_DIR=/usr/lib/x86_64-linux-gnu/hdf5/serial
   export OPENFAST_DIR=$HOME/Programs/openfast-2.4.0/install
   export LD_LIBRARY_PATH=$SOWFA_LIBBIN:$OPENFAST_DIR/lib:$LD_LIBRARY_PATH
   export PATH=$SOWFA_APPBIN:$OPENFAST_DIR/bin:$PATH
}
```

1. 写入.bashrc中
```bash
  echo "alias SOWFA='source ~/.sowfa_bashrc; SOWFA-6'" >> ~/.bashrc
  source ~/.bashrc
```

1. 执行SOWFA脚本来配置安装需要的环境变量
```bash
  SOWFA
  ./Allwmake
```

### SOWFA-2.4.0 (老版本，能装SOWFA-6就不用看这部分)
1. 下载源码

2. 在用户home目录创建.sowfa_bashrc文件，内容如下：
```bash
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# SOWFA-2.4.0
SOWFA-2.4.0()
{
  export inst_loc=$HOME/OpenFOAM
  export sowfa_loc=$HOME/OpenFOAM

   # Unset OpenFOAM environment variables.
   if [ -z "$FOAM_INST_DIR" ]; then
      echo "Nothing to unset..."
   else
      echo "     *Unsetting OpenFOAM environment variables..."
      . $FOAM_INST_DIR/OpenFOAM-2.4.0/etc/config/unset.sh
   fi

   # Set the OpenFOAM version and installation directory
   export OPENFOAM_VERSION=2.4.0
   export OPENFOAM_NAME=OpenFOAM-$OPENFOAM_VERSION
   export FOAM_INST_DIR=$inst_loc
   export WM_PROJECT_USER_DIR=$HOME/OpenFOAM/SOWFA-$OPENFOAM_VERSION

   foamDotFile=$FOAM_INST_DIR/$OPENFOAM_NAME/etc/bashrc
   if [ -f $foamDotFile ] ; then
      echo "Sourcing $foamDotFile..."
      source $foamDotFile
   fi

   export WM_NCOMPPROCS=12
   export WM_COLOURS="white blue green cyan red magenta yellow"

   alias tut='cd /$HOME/OpenFOAM/$USER-$OPENFOAM_VERSION/SOWFA-$OPENFOAM_VERSION/exampleCases'

   #export SOWFA_DIR=$FOAM_INST_DIR/$USER-$OPENFOAM_VERSION/SOWFA-$OPENFOAM_VERSION
   export SOWFA_DIR=$sowfa_loc/SOWFA-$OPENFOAM_VERSION
   export SOWFA_APPBIN=$SOWFA_DIR/applications/bin/$WM_OPTIONS
   export SOWFA_LIBBIN=$SOWFA_DIR/lib/$WM_OPTIONS
   export OPENFAST_DIR=$HOME/Programs/openfast-2.4.0/install
   export HDF5_DIR=/usr/lib/x86_64-linux-gnu/hdf5/serial
   export LD_LIBRARY_PATH=$SOWFA_LIBBIN:$OPENFAST_DIR/lib:$LD_LIBRARY_PATH
   export PATH=$SOWFA_APPBIN:$OPENFAST_DIR/bin:$PATH
}
```
3. 写入.bashrc中

4. 修改源码中的错误

    参考[原文](https://github.com/pablo-benito/SOWFA-installation#sowfa-compilation)
5. 执行SOWFA脚本来配置安装需要的环境变量

# CentOS-7下SOWFA的安装
## 一. OpenFOAM2.4.x安装(参考[此网站](https://openfoamwiki.net/index.php/Installation/Linux/OpenFOAM-2.4.x/CentOS_SL_RHEL#CentOS_7.1))
### 1. 安装依赖包
如果有sudo权限，直接sudo安装
```bash
# 切换到浙大镜像源
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.zju.edu.cn|g' \
-i.bak \
/etc/yum.repos.d/CentOS-*.repo

sudo yum makecache
sudo yum update
sudo yum upgrade

# 安装依赖
sudo -s
yum groupinstall 'Development Tools' 
yum install openmpi openmpi-devel zlib-devel gstreamer-plugins-base-devel libXext-devel libGLU-devel libXt-devel libXrender-devel libXinerama-devel libpng-devel libXrandr-devel libXi-devel libXft-devel libjpeg-turbo-devel libXcursor-devel readline-devel ncurses-devel python python-devel cmake qt-devel qt-assistant mpfr-devel gmp-devel  build-essential git ca-certificates flex libfl-dev bison zlib1g-dev libboost-system-dev libboost-thread-dev libopenmpi-dev openmpi-bin gnuplot libreadline-dev libncurses-dev libxt-dev

yum upgrade
# 退出sudo模式
exit
```
### 2. 下载OpenFOAM2.4.x和ThirdParty-2.4.x
打开一个新的终端窗口
```bash
cd ~
mkdir OpenFOAM
cd OpenFOAM
# 如果是在没有网的服务器上下载，需要先下载到有网的linux系统上（不下到windows是因为windows的换行标记和linux不一样，会报错），然后再拷贝过来（拷贝文件夹或者tar压缩后再拷贝）
git clone https://github.com/OpenFOAM/OpenFOAM-2.4.x.git
git clone https://github.com/OpenFOAM/ThirdParty-2.4.x.git
cd ThirdParty-2.4.x
mkdir download
# 将/files/OpenFOAM/目录下的三个压缩包通过sftp工具拷贝到这里的download文件夹
tar -xzf download/scotch_6.0.3.tar.gz
tar -xzf download/cgal-releases-CGAL-4.6.tar.gz && mv cgal-releases-CGAL-4.6 CGAL-4.6
tar -xjf download/boost_1_55_0.tar.bz2
```
### 3. 编译安装第三方库
```bash
# 修改源码
cd ~/OpenFOAM
sed -i -e 's=boost-system=boost_1_55_0=' OpenFOAM-2.4.x/etc/config/CGAL.sh
# 设置环境变量（以下两句二选一！）
## 如果是自己的虚拟机，一般通过yum安装的openmpi在/usr/lib64/openmpi/bin/目录下
module load mpi/openmpi-x86_64 || export PATH=$PATH:/usr/lib64/openmpi/bin
## 如果是大集群，openmpi在/usr/local/openmpi3.1.2_ucx/bin/目录下
export PATH=$PATH:/usr/local/openmpi3.1.2_ucx/bin/

# 这里的WM_NCOMPPROCS设置为你编译要用的核数（大集群使用手册说不要在主节点上并行，故设为单核） 
source $HOME/OpenFOAM/OpenFOAM-2.4.x/etc/bashrc WM_NCOMPPROCS=1 WM_MPLIB=SYSTEMOPENMPI
# 设置of24x别名来加载环境变量
## 如果是自己的虚拟机
echo "alias of24x='module load mpi/openmpi-x86_64; source \$HOME/OpenFOAM/OpenFOAM-2.4.x/etc/bashrc $FOAM_SETTINGS'" >> $HOME/.bashrc
## 如果是大集群
echo "alias of24x='export PATH=\$PATH:/usr/local/openmpi3.1.2_ucx/bin/; source \$HOME/OpenFOAM/OpenFOAM-2.4.x/etc/bashrc $FOAM_SETTINGS'" >> $HOME/.bashrc

source ~/.bashrc
of24x
# 编译安装gmp
# 将本文档根目录的gmp.tar上传到服务器~/Packages/目录下
cd ~/Packages
tar -xvf gmp-5.1.2.tar
cd gmp-5.1.2
./configure --prefix=$HOME/Packages/gmp-5.1.2/install --enable-cxx 
make
make check
make install

echo "export LD_LIBRARY_PATH=$HOME/Packages/gmp-5.1.2/install/lib:\$LD_LIBRARY_PATH" >> $HOME/.bashrc
source ~/.bashrc

# 编译安装CGAL
cd $WM_THIRD_PARTY_DIR
sed -i -e 's=boost-system=boost_1_55_0=' makeCGAL
./makeCGAL > log.mkcgal 2>&1
# 检查log.mkcgal文件是否有错误
```
### 4. 编译安装OpenFOAM
```bash
# 加载环境变量
wmSET $FOAM_SETTINGS
# 编译安装
cd $WM_PROJECT_DIR
./Allwmake > log.make 2>&1
# 再次运行，查看log.make内各个库是否编译成功
./Allwmake > log.make 2>&1
# paraview出错也没关系
```
### 5. （可选）修改paraFoam脚本
```bash
which paraFoam
```
修改paraFoam脚本为以下内容
```bash
pre_para=`basename $PWD`
para_file=${pre_para}.foam
>${para_file}
powershell.exe /c "paraview ${para_file}"
rm ${para_file}
```
<!-- 
## 二. OpenFAST-2.4.0安装
### 1. 安装依赖包
```bash
sudo yum install build-essential flex bison gfortran git cmake python python-dev  \
zlib1g-dev libreadline-dev libncurses-dev libyaml-cpp-dev libgmp-dev libmpfr-dev \
libboost-system-dev libboost-thread-dev libopenmpi-dev openmpi-bin \
libhdf5-dev libxml2-dev libcgal-dev libptscotch-dev libscotch-dev libopenblas-dev
```
### 2. 安装cmake
```bash
mkdir ~/package && cd ~/package
wget https://github.com/Kitware/CMake/releases/download/v3.26.3/cmake-3.26.3-linux-x86_64.sh
chmod +x cmake-3.26.3-linux-x86_64.sh
./cmake-3.26.3-linux-x86_64.sh
vim ~/.bashrc
# 添加以下内容并保存<-
export PATH=~/package/cmake-3.26.3-linux-x86_64/bin:$PATH
# ->
source ~/.bashrc
cmake --version
rm cmake-3.26.3-linux-x86_64.sh
```

### 3. 安装yaml-cpp
```bash
cd ~/package
git clone https://github.com/jbeder/yaml-cpp.git && cd yaml-cpp
mkdir build && cd build
cmake -DBUILD_SHARED_LIBS=ON ..
make
sudo make install
export HDF5_ROOT="/usr/lib/x86_64-linux-gnu/hdf5/serial"
export YAML_ROOT="/usr/"
```

### 3. 安装OpenFAST
```bash
mkdir ~/Program && cd ~/Program
wget https://github.com/OpenFAST/openfast/archive/refs/tags/v2.4.0.tar.gz
tar -zxvf v2.4.0.tar.gz && cd openfast-2.4.0

mkdir build && cd build
cmake .. -DBUILD_OPENFAST_CPP_API=ON -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release -DFPE_TRAP_ENABLED=ON

make help
make -j$(nproc)
sudo make install
``` -->

## 三. SOWFA安装
### 1. 下载源码
```bash
cd ~/OpenFOAM
git clone https://github.com/NREL/SOWFA.git
```
### 2. 配置编译和运行环境
在用户home目录创建.sowfarc文件，内容如下：
```bash
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# SOWFA-2.4.x
SOWFA-2.4.x()
{
  export inst_loc=$HOME/OpenFOAM
  export sowfa_loc=$HOME/OpenFOAM

   # Unset OpenFOAM environment variables.
   if [ -z "$FOAM_INST_DIR" ]; then
      echo "Nothing to unset..."
   else
      echo "     *Unsetting OpenFOAM environment variables..."
      . $FOAM_INST_DIR/OpenFOAM-2.4.x/etc/config/unset.sh
   fi

   # Set the OpenFOAM version and installation directory
   export OPENFOAM_VERSION=2.4.x
   export OPENFOAM_NAME=OpenFOAM-$OPENFOAM_VERSION
   export FOAM_INST_DIR=$inst_loc
   export WM_PROJECT_USER_DIR=$HOME/OpenFOAM/SOWFA

   foamDotFile=$FOAM_INST_DIR/$OPENFOAM_NAME/etc/bashrc
   if [ -f $foamDotFile ] ; then
      echo "Sourcing $foamDotFile..."
      source $foamDotFile
   fi

   export WM_NCOMPPROCS=12
   export WM_COLOURS="white blue green cyan red magenta yellow"

   alias tut='cd /$HOME/OpenFOAM/SOWFA/exampleCases'

   export SOWFA_DIR=$sowfa_loc/SOWFA
   export SOWFA_APPBIN=$SOWFA_DIR/applications/bin/$WM_OPTIONS
   export SOWFA_LIBBIN=$SOWFA_DIR/lib/$WM_OPTIONS

   export LD_LIBRARY_PATH=$SOWFA_LIBBIN:$LD_LIBRARY_PATH
   export PATH=$SOWFA_APPBIN:$PATH
}
```
然后在.bashrc文件中添加以下内容：
```bash
alias SOWFA='of24x && source ~/.sowfarc && SOWFA-2.4.x'
```
最后运行以下命令使配置生效：
```bash
source ~/.bashrc
SOWFA
```

### 3. 编译安装
```bash
cd $SOWFA_DIR
./Allwmake
# 再编译一遍检查有没有错误
./Allwmake > log.make 2>&1
```
### 4. 检查案例是否能运行
#### 1）复制大气边界层案例到运行目录下
```bash
cp -r $SOWFA_DIR/exampleCases/example.ABL.flatTerrain.neutral $FOAM_RUN/
run
cd example.ABL.flatTerrain.neutral
```
#### 2）修改runscript.preprocess文件
注释掉第13行：
```bash
# OpenFOAMversion=2.4.x-central
```
注释掉第71-80行：
```bash
# if [ $parallel -eq 1 ]
#    then
#    cd $PBS_O_WORKDIR
# fi


# # Source the bash profile and then call the appropriate OpenFOAM version function
# # so that all the modules and environment variables get set.
# source $HOME/.bash_profile
# OpenFOAM-$OpenFOAMversion
```
#### 3）修改runscript.solve.1文件
注释掉第8-12行：
```bash
# source $HOME/.bash_profile
# OpenFOAM-2.4.x-central
# module list

# cd $PBS_O_WORKDIR
```
第14行`core`改为小于等于电脑核数：
```bash
cores=8
```
#### 4）修改setUp文件
nx改为30，ny改为30，nz改为10

nCores改为你有的核数，比如8，decompOrder改为(2,2,2)，要保证乘起来要等与nCores

#### 5）修改system/controlDict.1文件
将endTime改为1000，writeInterval改为100

#### 6）预处理网格
```bash
./runscript.preprocess
```
#### 7）运行
```bash
./runscript.solve.1
```
#### 8）查看结果
```bash
reconstructPar
paraFoam &
```
