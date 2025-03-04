## 1.内核模块编译
1. 编写好驱动模块后在同级目录下Kconfig问价驱动信息
	```c
conf ig TTY_PRINTK

tristate "TTY driver to output user messages via printk"

depends on EXPERT && TTY

default n

---help---

If you say Y here, the support for writing user messages (i.e.

console messages) via printk is available.

The feature is useful to inline user messages with kernel

messages.

In order to use this feature, you should output user messages

to /dev/ttyprintk or redirect console to this TTY.

If unsure, say N.
```
2. Kconfig将影响menuconfig菜单选项
3. memuconfig菜单配置好后重新生成.config文件
	![[Pasted image 20250304103302.png]]
4. makefile文件中添加相关编译选项，从.config中判断是否编译
	```c
obj-$(CONF iG_TTY_PRINTK) += ttyprintk.o
```