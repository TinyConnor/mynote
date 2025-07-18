问题：
在WSL2中学习驱动开发，测试驱动安装是发现每次make都会报这个错误
```shell
make[1]: *** /lib/modules/5.15.167.4-microsoft-standard-WSL2/build: No such file or directory.  Stop. 
make: *** [Makefile:9: modules] Error 2
```

原因：
>WSL2的内核是修改过的，无法使用 ubuntu上游的内核头文件和modules文件，因此，我们需要手动编译并安装一个版本。

解决办法：
##### 1.下载对应的内核版本
1. 查看自己内核版本
```shell
wanderpurr@DESKTOP-5UHJTUE:~/WSL2-Linux-Kernel-linux-msft-wsl-5.15.167.4$ uname -r
5.15.167.4-microsoft-standard-WSL2
```

2. 去github上WSL源码上找到与自己内核对应的版本，下载并解压
```shell
wget https://github.com/microsoft/WSL2-Linux-Kernel/archive/refs/tags/linux-msft-wsl-5.15.167.4.tar.gz
tar -xvf linux-msft-wsl-5.15.167.4.tar.gz
```

##### 2.编译内核
1. 进入内核目录
```shell
cd WSL2-Linux-Kernel-linux-msft-wsl-5.15.167.4
```
2.  下载对应包(有一些包可能以前安过没有写出来)
```shell
sudo apt install build-essential flex bison libssl-dev libelf-dev binutils-dev dwarves -y
```
3. 编译内核（编译过程中如果出现什么错误肯定是有包没安装，搜索错误原因即可把缺失包安装上即可）
```shell
make KCONFIG_CONFIG=Microsoft/config-wsl -j$(nproc)
#如果出现很多是否选项可以ctrl+c截止
make menuconfig
#进入后直接退出即可，再次执行编译
```
4. 链接内核文件
```shell
sudo ln -s $(pwd) /lib/modules/$(uname -r)/build
```

- **`make clean`**：轻量级清理，保留配置，适合快速重新编译。
- **`make mrproper`**：彻底清理，不保留任何编译痕迹，适合全新编译或切换配置。


---
>以下两个问题都是WSL本身问题，禁止了模块加载，最佳解决办法就是更换自己的内核，

问题：
编译好内核后，尝试编译自己测试驱动
可以正常make,但安装时候会报错
```shell
wanderpurr@DESKTOP-5UHJTUE:~/test_proj/Chapter02$ sudo insmod helloworld.ko 
[sudo] password for wanderpurr:  
insmod: ERROR: could not insert module helloworld.ko: Invalid module format
```

解决办法：
让系统启动编译好的内核
```shell
#进入内核目录下
cp arch/x86_64/boot/bzImage /mnt/g #指定Windows盘符，任意位置
```
在Windows用户目录下创建配置文件
```shell
C:/user/{用户名}/.wslconfig #没有就创建

#填写如下内容
[wsl2]  
kernel=G:\\bzImage #上边指定的目录
```
重启即可
```shell
#powershell
wsl --shutdown
```



问题：
更换内核后，重新编译测试驱动
打印信息为：
```shell
wanderpurr@DESKTOP-5UHJTUE:~$ sudo insmod helloworld.ko 
insmod: ERROR: could not insert module helloworld.ko: Invalid parameters

wanderpurr@DESKTOP-5UHJTUE:~$dmesg
[32310.582652] BPF:[133625] Invalid name_offset:2402270 
[32310.582915] failed to validate module [helloworld] BTF: -22
```

原因：
内核强制要求 BTF 验证，而当前编译环境无法生成有效的 BTF 信息

解决办法：
```shell
objcopy --remove-section .BTF helloworld.ko
```
