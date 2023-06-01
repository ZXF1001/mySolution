[< 返回主页](../README.md)
# 大集群上OpenFOAM-2.4.x和SOWFA的安装（无网络、无sudo权限）
## 〇. 准备工作
根据以下下载链接下载安装包和依赖库的源码（如果链接失效，可以网上找对应版本的安装包下载），使用sftp工具上传到服务器的`~/upload`文件夹（注意：后面的步骤可能有的地方用到的tar路径不是这里的，注意修改）

[gcc-4.8.5](http://mirror.linux-ia64.org/gnu/gcc/releases/gcc-4.8.5/gcc-4.8.5.tar.gz)、
[yaml-cpp-0.6.0](https://codeload.github.com/jbeder/yaml-cpp/tar.gz/refs/tags/yaml-cpp-0.6.0)、
[hdf5-1.8.15](https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.8/hdf5-1.8.15/src/hdf5-1.8.15.tar.gz)、
[libxml2-2.8.0](https://gitlab.gnome.org/GNOME/libxml2/-/archive/v2.8.0/libxml2-v2.8.0.tar.gz)、
[lapack-3.10.1](https://codeload.github.com/Reference-LAPACK/lapack/tar.gz/refs/tags/v3.10.1)、
[openfast-2.4.0](https://codeload.github.com/OpenFAST/openfast/tar.gz/refs/tags/v2.4.0)、
[mpc-1.0.1](https://mirrors.sjtug.sjtu.edu.cn/gnu/mpc/mpc-1.0.1.tar.gz)、
[mpfr-3.1.2](https://www.mpfr.org/mpfr-3.1.2/mpfr-3.1.2.tar.gz)、[gmp-5.1.2](https://mirrors.aliyun.com/gnu/gmp/gmp-5.1.2.tar.xz?spm=a2c6h.25603864.0.0.34897154FANPT7)、
[CGAL-4.6](https://codeload.github.com/CGAL/cgal/tar.gz/refs/tags/releases/CGAL-4.6)、
[boost-1.55.0](https://udomain.dl.sourceforge.net/project/boost/boost/1.55.0/boost_1_55_0.tar.bz2)、
[scotch_6.0.3](https://www.labri.fr/perso/pelegrin/scotch/distrib/scotch_6.0.3.tar.gz)

另外，OpenFOAM和SOWFA源码的压缩包保存在了[这个github仓库](https://github.com/ZXF1001/SOWFA_Installation_Files)中，下载下来，也上传到服务器的`~/upload`文件夹
## 一. OpenFOAM2.4.x安装(基于[此网站](https://openfoamwiki.net/index.php/Installation/Linux/OpenFOAM-2.4.x/CentOS_SL_RHEL#CentOS_7.1)部分修改)
### 1. 下载OpenFOAM2.4.x和ThirdParty-2.4.x
打开一个新的终端窗口
```bash
mkdir ~/OpenFOAM && cd ~/OpenFOAM
# 将/file目录下OpenFOAM和ThirdParty的压缩包拷贝到这里
tar -zxf OpenFOAM-2.4.x.tar.gz
tar -zxf ThirdParty-2.4.x.tar.gz
```
### 2. 下载第三方依赖库
```bash
cd ThirdParty-2.4.x
mkdir download
```
将/files/目录下的几个压缩包通过sftp工具拷贝到这里的download文件夹，再解压到ThirdParty-2.4.x目录下
```bash
tar -zxf download/scotch_6.0.3.tar.gz
tar -zxf download/cgal-releases-CGAL-4.6.tar.gz && mv cgal-releases-CGAL-4.6 CGAL-4.6
tar -jxf download/boost_1_55_0.tar.bz2
tar -zxf download/gcc-4.8.5.tar.gz
tar -xf download/gmp-5.1.2.tar
tar -zxf download/mpfr-3.1.2.tar.gz
tar -zxf download/mpc-1.0.1.tar.gz
```
### 3. 设置环境变量
修改源码中的boost版本
```bash
cd ~/OpenFOAM
sed -i -e 's=boost-system=boost_1_55_0=' OpenFOAM-2.4.x/etc/config/CGAL.sh
```
设置调用的mpi库为openmpi3.1.2_ucx
```bash
export PATH=$PATH:/usr/local/openmpi3.1.2_ucx/bin/
# 这里的WM_NCOMPPROCS设置为你编译要用的核数（大集群使用手册说不要在主节点上并行，故设为单核） 
```
添加环境变量
```bash
source $HOME/OpenFOAM/OpenFOAM-2.4.x/etc/bashrc WM_NCOMPPROCS=1

echo "alias of24x='export PATH=\$PATH:/usr/local/openmpi3.1.2_ucx/bin/; source \$HOME/OpenFOAM/OpenFOAM-2.4.x/etc/bashrc $FOAM_SETTINGS'" >> $HOME/.bashrc
echo 'of24x' >> $HOME/.bashrc
source ~/.bashrc 
```

### 4. 安装第三方依赖库
通过makeGcc脚本编译安装CGAL需要的gmp和mpfr库
```bash
cd $WM_THIRD_PARTY_DIR
./makeGcc gcc-4.8.5
# 可能在最后编译安装gcc的时候会报错，可以不用管，因为需要的是前面的gmp和mpfr库
```
修改编译cgal要用到的库版本
```bash
sed -i -e 's=boost-system=boost_1_55_0=' makeCGAL
sed -i -e 's=gmp-system=gmp-5.1.2=' makeCGAL
sed -i -e 's=mpfr-system=mpfr-3.1.2=' makeCGAL
```
编译第三方依赖库
```bash
./Allwmake
# 再次安装输出安装报告，检查安装情况
./Allwmake > log.make 2>&1
```
### 5. 安装OpenFOAM
```bash
cd $WM_PROJECT_DIR
./Allwmake
# 再次安装输出安装报告，检查安装情况
./Allwmake > log.make 2>&1
```
可能会出现`fatal error: gmp.h: No such file or directory`，只会影响多孔介质网格的生成工具`foamyHexMesh`和`foamyQuadMesh`，影响不大

创建run文件夹并进入
```bash
mkdir -p $FOAM_RUN
run
```
### 6. 测试
#### icoFoam单核算例
拷贝测试算例并生成网格
```bash
cp -r $FOAM_TUTORIALS/incompressible/icoFoam/cavity $FOAM_RUN
cd cavity
blockMesh
```
当前目录下创建提交脚本script.sh，内容如下（将`#$ -N zxf`中的`zxf`修改为你的任务名）：
```bash
#!/bin/sh 
#___INFO__MARK_BEGIN__
# Welcome to use  EasyCluster V1.6 All Rights Reserved.
#
#___INFO__MARK_END__
#
#$ -S /bin/sh 
#$ -N zxf 
#$ -j y 
#$ -o ./ 
#$ -e ./ 
#$ -cwd  
#$ -q zone2.q  
#$ -pe thread 1

source ~/.bashrc
hash -r
export path=$TMPDIR:$path

icoFoam > log.icoFoam 2>&1
```
提交、查询任务
```bash
# 提交任务
ssub script.sh

#查询任务状态
qstat
```
#### 并行测试，damBreak算例
拷贝测试算例并生成网格
```bash
cp -r $FOAM_TUTORIALS/multiphase/interFoam/laminar/damBreak $FOAM_RUN
cd damBreak
blockMesh
cp -r 0/alpha.water.org 0/alpha.water
setFields
decomposePar
```
在当前目录下创建提交脚本script.sh，内容如下：
```bash
#!/bin/sh 
#___INFO__MARK_BEGIN__
# Welcome to use  EasyCluster V1.6 All Rights Reserved.
#
#___INFO__MARK_END__
#
#$ -S /bin/sh 
#$ -N zxf_openfoam_test 
#$ -j y 
#$ -o ./ 
#$ -e ./ 
#$ -cwd  
#$ -q zone2.q  
#$ -pe mpi 4-4
source ~/.bashrc
hash -r
export path=$TMPDIR:$path

mpirun -np $NSLOTS interFoam -parallel > log.interFoam 2>&1
```
提交、查询任务
```bash
# 提交任务
ssub script.sh

#查询任务状态
qstat
```
计算完成后，使用reconstructPar命令重建数据
```bash
reconstructPar
# 将数据文件传到本地，用paraview查看
```

## 二. OpenFAST安装
### 1. 安装依赖
如果还没有创建所有依赖库的安装目录，可以运行下面的命令创建
```bash
mkdir -p $HOME/Packages
```
进入依赖库的安装目录
```bash
cd $HOME/Packages/
```
将本文档下`files/OpenFAST/`目录下的安装包手动拷贝到`$HOME/Packages/`目录下
#### BLAS&LAPACK
解压安装包
```bash
tar -zxf lapack-3.10.1.tgz && cd lapack-3.10.1
```
编译安装
```bash
cp make.inc.example make.inc
make blaslib
make lapacklib
```
#### LibXml2
解压安装包
```bash
tar -zxf libxml2-2.8.0.tar.gz && cd libxml2-2.8.0
```
编译安装
```bash
mkdir build && cd build
../configure --prefix=$HOME/Packages/libxml2-2.8.0/install
make
make install
```
#### HDF5
解压安装包
```bash
tar -zxf hdf5-1.8.15.tar.gz && cd hdf5-1.8.15
```
编译安装
```bash
mkdir build && cd build
../configure --prefix=$HOME/Packages/hdf5-1.8.15/install
make
make check
make install
```

#### ymal-cpp
解压安装包
```bash
tar -zxf yaml-cpp-yaml-cpp-0.6.0.tar.gz
mv yaml-cpp-yaml-cpp-0.6.0 yaml-cpp-0.6.0
```
编译安装
```bash
cd yaml-cpp-0.6.0
mkdir build && cd build
cmake .. \
-DCMAKE_INSTALL_PREFIX="$HOME/Packages/yaml-cpp-0.6.0/install" \
-DBUILD_SHARED_LIBS=ON 
make
make install
```

### 2. 安装OpenFAST-2.4.0
将文档下的`files/OpenFAST/openfast-2.4.0.tar.gz`拷贝到服务器的安装目录下，这里以`$HOME/Programs`为例
```bash
cd ~/Programs
```
解压安装包
```bash
tar -zxf openfast-2.4.0.tar.gz
cd openfast-2.4.0
```
激活intel编译器环境变量
```bash
source /opt/intel/parallel_studio_xe_2020/bin/psxevars.sh intel64
```
进入编译文件夹
```bash
mkdir build && cd build
```
配置编译选项（注意这里的依赖库路径要和你安装的路径对应）
```bash
cmake .. \
-DBLAS_LIBRARIES="$HOME/Packages/lapack-3.10.1" \
-DLAPACK_LIBRARIES="$HOME/Packages/lapack-3.10.1" \
-DLIBXML2_LIBRARY="$HOME/Packages/libxml2-2.8.0/install/lib" \
-DLIBXML2_INCLUDE_DIR="$HOME/Packages/libxml2-2.8.0/install/include/libxml2/libxml" \
-DBUILD_OPENFAST_CPP_API=ON \
-DHDF5_ROOT="$HOME/Packages/hdf5-1.8.15/install" \
-DYAML_ROOT="$HOME/Packages/yaml-cpp-0.6.0/install" \
-DCMAKE_C_COMPILER=icc \
-DCMAKE_CXX_COMPILER=icpc \
-DCMAKE_Fortran_COMPILER=ifort \
-DBUILD_SHARED_LIBS=ON \
-DCMAKE_BUILD_TYPE=Release \
-DFPE_TRAP_ENABLED=ON \
-DCMAKE_INSTALL_PREFIX="$HOME/Programs/openfast-2.4.0/install"
```
安装
```bash
make help
make
make install
```
没有报error即为安装成功

## 三. SOWFA的安装
### 1. 拷贝源码
将本文档`/files`目录下的SOWFA.tar.gz拷贝到服务器的`$HOME/OpenFOAM/`目录下，并进入目录
```bash
cd ~/OpenFOAM
tar -zxf SOWFA.tar.gz
cd SOWFA
```
### 2. 修改源码错误
详见[这篇文档](https://github.com/pablo-benito/SOWFA-installation#sowfa-compilation)，修改源码的部分错误再编译，要修改的三个文件分别为：
1. `applications/solvers/incompressible/windEnergy/pisoFoamTurbine.ALMAdvancedOpenFAST/Make/options`
2. `applications/solvers/incompressible/windEnergy/windPlantSolver.ALMAdvancedOpenFAST/Make/options`
3. `src/turbineModels/turbineModelsOpenFAST/Make/options`


### 3. 创建.bashrc中的启动命令
在`$HOME`目录下创建`.sowfarc`文件，内容如下:
```bash
export inst_loc=$HOME/OpenFOAM
export sowfa_loc=$HOME/OpenFOAM

# Unset OpenFOAM environment variables.
if [ -z "$FOAM_INST_DIR" ]; then
   echo "Nothing to unset..."
else
   echo "*Unsetting OpenFOAM environment variables..."
   . $FOAM_INST_DIR/OpenFOAM-2.4.x/etc/config/unset.sh
fi

# Set the OpenFOAM version and installation directory
export OPENFOAM_VERSION=2.4.x
export OPENFOAM_NAME=OpenFOAM-$OPENFOAM_VERSION
export FOAM_INST_DIR=$inst_loc
export WM_PROJECT_USER_DIR=$sowfa_loc/SOWFA

foamDotFile=$FOAM_INST_DIR/$OPENFOAM_NAME/etc/bashrc
if [ -f $foamDotFile ] ; then
   echo "Sourcing $foamDotFile..."
   source $foamDotFile
fi

export WM_NCOMPPROCS=1
export WM_COLOURS="white blue green cyan red magenta yellow"

alias tut='cd $sowfa_loc/SOWFA/exampleCases'

export SOWFA_DIR=$sowfa_loc/SOWFA
export SOWFA_APPBIN=$SOWFA_DIR/applications/bin/$WM_OPTIONS
export SOWFA_LIBBIN=$SOWFA_DIR/lib/$WM_OPTIONS
export OPENFAST_DIR=$HOME/Programs/openfast-2.4.0/install
export HDF5_DIR=$HOME/Packages/hdf5-1.8.15/install

export LD_LIBRARY_PATH=$SOWFA_LIBBIN:$OPENFAST_DIR/lib:$LD_LIBRARY_PATH
export PATH=$SOWFA_APPBIN:$OPENFAST_DIR/bin:$PATH
```
把这个启动命令加入到`~/.bashrc`文件中
```bash
echo "alias SOWFA = 'of24x && source ~/.sowfarc'" >> ~/.bashrc
```
更新`~/.bashrc`文件
```bash
source ~/.bashrc
```
### 4. 执行安装
设置环境变量
```bash
SOWFA
```
编译安装
```bash
cd $SOWFA_DIR
./Allwmake
```
再次执行来查看各个求解器安装情况
```bash
./Allwmake > log.make 2>&1
```
未来每次需要使用SOWFA的时候都要输入`SOWFA`来激活环境
### 5. 测试案例
#### 复制大气边界层案例到运行目录下
```bash
cp -r $SOWFA_DIR/exampleCases/example.ABL.flatTerrain.neutral $FOAM_RUN/
run
cd example.ABL.flatTerrain.neutral
```
#### 修改runscript.preprocess文件
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
#### 修改runscript.solve.1文件
为了在集群上并行运行，改成了如下模板（可以把`#$ -N zxf`中的`zxf`改成你要的任务名，把`#$ -pe mpi 8-8`中的两个8改成你的核数）：
```bash
#!/bin/sh 
#___INFO__MARK_BEGIN__
# Welcome to use  EasyCluster V1.6 All Rights Reserved.
#
#___INFO__MARK_END__
#
#$ -S /bin/sh 
#$ -N zxf 
#$ -j y 
#$ -o ./ 
#$ -e ./ 
#$ -cwd  
#$ -q zone2.q  
#$ -pe mpi 8-8

source ~/.bashrc
source ~/.sowfarc
hash -r
export path=$TMPDIR:$path

initializer=setFieldsABL
solver=ABLSolver
runNumber=1
startTime=0

cp system/controlDict.$runNumber system/controlDict

echo "Starting OpenFOAM job at: " $(date)
echo "using " $NSLOTS " cores"

# Run the flow field initializer (parallel)
if [ $runNumber -eq 1 ] 
   then
   mpirun -np $NSLOTS $initializer -parallel > log.$runNumber.$initializer 2>&1
fi
# Run the solver (parallel)
mpirun -np $NSLOTS $solver -parallel > log.$runNumber.$solver 2>&1

echo "Ending OpenFOAM job at: " $(date)
```
#### 修改setUp文件
nx改为30，ny改为30，nz改为10

nCores改为你有的核数，比如8，decompOrder改为(2,2,2)，要保证乘起来要等与nCores

#### 修改system/controlDict.1文件
将endTime改为1000，writeInterval改为100
#### 运行计算
网格预处理
```bash
./runscript.preprocess
```
运行并行计算
```bash
# 提交任务
ssub runscript.solve.1

# 查看任务
qstat
```
可以通过`log.n.setFieldsABL`和`log.n.ABLSolver`文件来查看结果

合并多核结果
```bash
reconstructPar
```