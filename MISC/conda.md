在终端下使用conda,这样会是**cmd启动变慢，谨慎添加**

 >conda环境变量
 D:\miniconda3
D:\miniconda3\Scripts
D:\miniconda3\Library\bin


1. 查看系统执行策略
 get-executionpolicy
Restricted
2. 修改系统执行策略
 set-executionpolicy remotesigned
在交互中选择是，Y
3. 终端重启conda
   conda init powershell
会在user/用户名/.bash_profile下
```shell
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
if [ -f '/d/conda/Scripts/conda.exe' ]; then
    eval "$('/d/conda/Scripts/conda.exe' 'shell.bash' 'hook')"
fi
# <<< conda initialize <<<
```
### 1.管理conda自身
#### 1.1查看conda版本
```shell
conda --version
conda -V
```
#### 1.2查看conda环境配置
```shell
conda config --show
```
#### 1.3设置镜像源
```shell
#设置清华镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
#设置bioconda
conda config --add channels bioconda
conda config --add channels conda-forge
#设置搜索时显示通道地址
conda config --set show_channel_urls yes
```
#### 1.4更新conda
```shell
conda update conda
```
#### 1.5更新anaconda整体
```shell
conda update Anaconda
```
#### 1.6查询某个命令的帮助
```shell
conda <cmd> --help
```
### 2.管理环境
#### 2.0修改环境默认位置
**方法一：**
```shell
#添加新路径
conda config --add envs_dirs D:\ProgramData\anaconda3\envs
#查看配置是否生效
conda condif --show envs_dirs
#删除环境路径
conda config --remove envs_dirs <path>

#更改包存放路径
conda config --show pkgs_dirs  # 显示包缓存目录
conda config --add pkgs_dirs D:\ProgramData\anaconda3\pkgs
```
**方法二：**
修改`C:\Users\Username\.condarc`内容
#### 2.1创建虚拟环境
```shell
conda create -n <env_name> python=3.8 #python版本号

#创建虚拟环境同时安装必要的包
conda create -n <env_name> numpy matplotlib python=3.8

#指定环境路径
conda create --prefix=C:/ProgramData/Anaconda3/envs/pytorch python=3.8 #pytorch就是创建的环境名

```
#### 2.2查看虚拟环境
```shell
conda env list
conda info -e
conda info --envs
```
#### 2.3激活虚拟环境
```shell
conda activate <env_name>
```
#### 2.4退出虚拟环境
```shell
conda activate
conda deactivate
#以上两条命令只中任一条都会让你回到base environment，它们从不同的角度出发到达了同一个目的地。可以这样理解，activate的缺省值是base，deactivate的缺省值是当前环境，因此它们最终的结果都是回到base
```
#### 2.5删除虚拟环境
```shell
#删除虚拟环境及所有安装的包
conda remove --name <env_name> --all

#删除虚拟环境中的某个或者某些包
conda remove --name <env_name> <package_name>
```
#### 2.6导出环境
```shell
#获得环境中的所有配置
conda env export -name myrnv > myenv.yml
#重新还原环境
conda env create -f myenv.yml
```
### 3.包管理
#### 3.1查询包的安装情况
```shell
conda list
#查询当前Anaconda repository中是否有你想要安装的包
conda search <package_name>
```
#### 3.2查询是否安装某个包
```shell
conda list <pkgname>
conda list <pkgname*>#支持通配符
```
#### 3.3包的安装与更新
```shell
#安装
conda install <package_name>
#指定版本
conda install <package_name>=<版本号>
conda update numpy=0.20.3
#更新包
conda update <package_name>
#指定通道安装
conda install pkg_name -c <conda_forge>
```
#### 3.4卸载包
```shell
#删除指定包 以及依赖这个包的所有包
conda uninstall <package_name>
#仅删除指定包
conda uninstall <package_name> --force
```
#### 3.5清理缓存
```shell
conda clean -p      # 删除没有用的包 --packages
conda clean -t      # 删除tar打包 --tarballs
conda clean -y -all # 删除所有的安装包及cache(索引缓存、锁定文件、未使用过的包和tar包)
```
### 4.python版本管理
>除了在创建环境时指定Python版本也可以在环境中再次更改

#### 5.1将版本变更到指定版本
```shell
conda install python=3.5

#变更后查看是否符合预期
python --version
```
#### 5.2将python版本变更到最新版本
```shell
conda update python
```
### 6.channel管理
```shell
#追加conda-forge channel
conda config --add channels <conda-forge>
#移除conda-forge channel
conda config --remove channels <conda-force>
#查询当前配置中包含哪些channels
conda config --get channels
```
### 7. conda install 和 pip install
1. conda可以管理非python包，pip只能管理python包。
2. conda自己可以用来创建环境，pip不能，需要依赖virtualenv之类的。
3. conda安装的包是编译好的二进制文件，安装包文件过程中会自动安装依赖包；pip安装的包是wheel或源码，装过程中不会去支持python语言之外的依赖项。
4. conda安装的包会统一下载到一个目录文件中，当环境B需要下载的包，之前其他环境安装过，就只需要把之间下载的文件复制到环境B中，下载一次多次安装。pip是直接下载到对应环境中。
5. conda只能在conda管理的环境中使用，例如比如conda所创建的虚环境中使用。pip可以在任何环境中使用，在conda创建的环境 中使用pip命令，需要先安装pip（conda install pip ），然后可以 环境A 中使用pip 。conda 安装的包，pip可以卸载，但不能卸载依赖包，pip安装的包，只能用pip卸载。

> pip和conda在安装软件包时，在依赖关系方面的处理机制不同。
> 
> pip在递归的串行循环中安装依赖项，不会确保同时满足所有软件包的依赖关系，如果按顺序较早安装的软件包相对于按顺序较晚安装的软件包具有不兼容的依赖项版本，则可能导致环境以微妙的方式被破坏掉；
> 
> conda使用SAT（satisfiability）solver来验证是否满足环境中安装的所有软件包的所有要求，只要有关依赖项的软件包元数据正确，conda就会按预期产生可用的环境。

