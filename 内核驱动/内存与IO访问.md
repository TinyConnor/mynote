# 1.内存管理单元MMU
1. `TLB`:即转换旁路缓存，TLB是MMU的核心部件，它缓存少量的虚拟地址与物理地址的转换关系，是转换表的Cache，因此也经常被称为“快表”。
2. `TTW`:即转换表漫游，当TLB中没有缓冲对应的地址转换关系时，需要通过对内存中转换表（大多数处理器的转换表为多级页表，如图11.2所示）的访问来获得虚拟地址和物理地址的对应关系。TTW成功后，结果应写入TLB中。
![[ARM处理器的MMU.png]]
MMU具有虚拟地址和物理地址转换、内存访问权限保护等功能，这将使得Linux操作系统能单独为系统的每个用户进程分配独立的内存空间并保证用户空间不能访问内核空间的地址，为操作系统的虚拟内存管理模块提供硬件基础。
# 2.Linux内存管理
在Linux系统中，进程的4GB内存空间被分为两个部分——用户空间与内核空间。用户空间的地址一般分布为0~3GB（即PAGE_OFFSET，在0x86中它等于0xC0000000），这样，剩下的3~4GB为内核空间，如图11.5所示。用户进程通常只能访问用户空间的虚拟地址，不能访问内核空间的虚拟地址。用户进程只有通过系统调用（代表用户进程在内核态执行）等方式才可以访问到内核空间。
## 2.1ARM Linux的内存映射情况
1. 0xffff0000~0xffff0fff是“CPU vector page”，即向量表的地址。
2. 0xffc00000~0xffefffff是DMA内存映射区域，dma_alloc_xxx族函数把DMA缓冲区映射在这一段，
	*  VMALLOC_START~VMALLOC_END-1是vmalloc和ioremap区域（在vmalloc区域的大小可以配置，通过“vmalloc=”这个启动参数可以指定），
	* PAGE_OFFSET~high_memory-1是**DMA和正常区域的映射区域**，
	* MODULES_VADDR~MODULES_END-1是**内核模块区域**，
	* PKMAP_BASE~PAGE_OFFSET-1是**高端内存映射区**。

 >假设我们把PAGE_OFFSET定义为3GB，实际上Linux内核模块位于3GB-16MB~3GB-2MB，高端内存映射区则通常位于3GB-2MB~3GB。
![[32位ARM系统中Linux内核的地址空间.png]]
# 3.内核空间内存动态申请
## 3.1kmalloc()
```c
void *kmalloc(size_t size, int flags);
@brief 申请指定大小的位于DMA或常规区域的连续内存
@param size 要分配的块的大小
@param flags 分配标志
		GFP_KERNEL 在内核空间的进程中申请内存，若暂时不能满足，则进程会睡眠等待页，即会引起阻塞
		GFP_ATOMIC 不存在空闲页，则不等待，直接返回
		GFP_USER 为用户空间页分配内存，可能阻塞
		GFP_HIGHUSER 类似GFP_USER，但是它从高端内存分配
		GFP_DMA 从DMA区域分配内存
		GFP_NOIO 不允许任何I/O初始化
		GFP_NOFS 不允许进行任何文件系统调用
		__GFP_HIGHMEM 指示分配的内存可以位于高端内存
		__GFP_COLD 请求一个较长时间不访问的页
		__GFP_NOWARN 当一个分配无法满足时，阻止内核发出警告
		__GFP_HIGH 高优先级请求，允许获得被内核保留给紧急状况使用的最后的内存页
		__GFP_REPEAT 分配失败，则尽力重复尝试
		__GFP_NOFAIL 标志只许申请成功，不推荐
		__GFP_NORETRY 若申请不到，则立即放弃
@return 申请内存首地址
```
## 3.2__get_free_pages()
```c
get_zeroed_page(unsigned int flags);
@brief 返回一个指向新页的指针并且将该页清零

__get_free_page(unsigned int flags);
@brief 返回一个指向新页的指针但是该页不清零

__get_free_pages(unsigned int flags, unsigned int order);
@brief 可分配多个页并返回分配内存的首地址，分配的页数为2order，分配的页也不清零。order允许的最大值是10（即1024页）或者11（即2048页）

struct page * alloc_pages(int gfp_mask, unsigned long order);
@brief 最底层调用方法，既可在内核空间也可以在用户空间，返回第一个页的描述符

void free_page(unsigned long addr);
void free_pages(unsigned long addr, unsigned long order);
@brief 释放函数
```
## 3.3vmalloc()
vmalloc（）一般只为存在于软件中（没有对应的硬件意义）的较大的顺序缓冲区分配内存
```c
void *vmalloc(unsigned long size);
void vfree(void * addr);
```
vmalloc（）在申请内存时，会进行内存的映射，改变页表项，不像kmalloc（）实际用的是开机过程中就映射好的DMA和常规区域的页表项。因此vmalloc（）的虚拟地址和物理地址不是一个简单的线性映射。
## 3.4slab
slab是建立在buddy算法之上的，它从buddy算法拿到2n页面后再次进行二次管理，这一点和用户空间的C库很像
### 3.4.1创建slab缓存
```c
struct kmem_cache *kmem_cache_create(const char *name, size_t size,
size_t align, unsigned long flags,
void (*ctor)(void*, struct kmem_cache *, unsigned long),
void (*dtor)(void*, struct kmem_cache *, unsigned long));
@brief 创建一个slab缓存，它是一个可以保留任意数目且全部同样大小的后备缓存
@param size 要分配的每个数据结构的大小
@param flags 控制如何进行分配的位掩码
		SLAB_HWCACHE_ALIGN 每个数据对象被对齐到一个缓存行
		SLAB_CACHE_DMA 要求数据对象在DMA区域中分配
```
### 3.4.2分配slab缓存
```c
void *kmem_cache_alloc(struct kmem_cache *cachep, gfp_t flags);
@brief 在创建的slab后备缓存中分配一块并返回首地址指针
```
### 3.4.3 释放slab缓存
```c
void kmem_cache_free(struct kmem_cache *cachep, void *objp);
```
### 3.4.4 收回slab缓存
```c
int kmem_cache_destroy(struct kmem_cache *cachep);
```
## 3.5 内存池
除了slab以外，在Linux内核中还包含对内存池的支持，内存池技术也是一种非常经典的用于分配大量小对象的后备缓存技术
### 3.5.1 创建内存池
```c
mempool_t *mempool_create(int min_nr, mempool_alloc_t *alloc_fn,mempool_free_t *free_fn, void *pool_data);
@brief 创建一个内存池
@param min_nr 需要预分配对象的数目
@param alloc_fn和free_fn 指向内存池机制提供的标准对象分配和回收函数的指针
@param data 分配和回收函数用到的指针，gfp_mask是分配标记。只有当__GFP_WAIT标记被指定时，分配函数才会休眠。

typedef void *(mempool_alloc_t)(int gfp_mask, void *pool_data);
typedef void (mempool_free_t)(void *element, void *pool_data);
```
### 3.5.2 分配和回收对象
```c
void *mempool_alloc(mempool_t *pool, int gfp_mask);
void mempool_free(void *element, mempool_t *pool);
```
### 3.5.3回收内存池
```c
void mempool_destroy(mempool_t *pool);
```
# 4.设备IO内存
## 4.1设备IO内存访问
寄存器可能位于I/O空间中，也可能位于内存空间中。当位于I/O空间时，通常被称为I/O端口；当位于内存空间时，对应的内存空间被称为I/O内存。
```c
void *ioremap(unsigned long offset, unsigned long size);
@brief 将设备所处的物理地址映射到虚拟地址上
@details 需要建立新的页表，但是它并不进行vmalloc（）中所执行的内存分配行为
@return ioremap（）返回一个特殊的虚拟地址，该地址可用来存取特定的物理地址范围，这个虚拟地址位于vmalloc映射区域

void iounmap(void * addr);
@brief ioremap（）获得的虚拟地址应该被iounmap（）函数释放

void __iomem *devm_ioremap(struct device *dev, resource_size_t offset,unsigned long size);
@brief 不需要释放

读写寄存器
#define readb(c) ({ u8 __v = readb_relaxed(c); __iormb(); __v; })
#define readw(c) ({ u16__v = readw_relaxed(c); __iormb(); __v; })
#define readl(c) ({ u32 __v = readl_relaxed(c); __iormb(); __v; })

#define writeb(v,c) ({ __iowmb(); writeb_relaxed(v,c); })
#define writew(v,c) ({ __iowmb(); writew_relaxed(v,c); })
#define writel(v,c) ({ __iowmb(); writel_relaxed(v,c); })
```
## 4.2 申请与释放设备的IO内存
```c
struct resource *request_mem_region(unsigned long start, unsigned long len, char *name);
@brief 申请一段IO内存
@param start 申请的地址从start开始
@param len 申请len个内存地址
@param name 申请设备的名字
@return 成功 不是NULL

void release_mem_region(unsigned long start, unsigned long len);
@brief 归还IO内存给系统

变体：
devm_request_region（）和devm_request_mem_region（）
```
## 4.3设备IO内存访问流程
* 调用request_mem_region（）申请资源，
* 将寄存器地址通过ioremap（）映射到内核空间虚拟地址，
* 通过Linux设备访问编程接口访问这些设备的寄存器了。
* 访问完成后，应对ioremap（）申请的虚拟地址进行释放，
* 释放release_mem_region（）申请的I/O内存资源。

有时候，驱动在访问寄存器或I/O端口前，会省去request_mem_region（）、request_region（）这样的调用。
![[IO内存访问流程.png]]
## 4.4 将设备地址映射到用户空间
### 4.4.1 内存映射与VMA
设备驱动程序中可实现mmap函数，这个函数可使得用户空间能直接访问设备的物理地址
mmap（）实现了这样的一个**映射过程**：它将用户空间的一段内存与设备内存关联，当用户访问用户空间的这段地址范围时，实际上会转化为对设备的访问。

mmap（）必须以PAGE_SIZE为单位进行映射，实际上，内存只能以页为单位进行映射，若要映射非PAGE_SIZE整数倍的地址范围，要先进行页对齐，强行以PAGE_SIZE的倍数大小进行映射。
```c
内核驱动中的mmap
int(*mmap)(struct file *, struct vm_area_struct*);

用户空前的mmap
caddr_t mmap (caddr_t addr, size_t len, int prot, int flags, int fd, off_t offset);
@param addr 映射地址
@param len 映射到用户空间的字节数
@param prot 访问权限
			PROT_READ 可读
			PROT_WRITE 可写
			PROT_EXEC 可执行
			PROT_NONE 不可访问
@param flags 如果fd为-1需指定MAP_ANON表明匿名映射
@param fd 文件描述符
@param offset 映射到地址开头offset个字节开始算起
@return 映射到用户空间的地址。其类型caddr_t实际上就是void*
```
用户调用mmap（）的时候，内核会进行如下处理
1. 在进程的虚拟空间查找一块VMA *(file_operations中mmap实现)*
2. 将这块VMA进行映射
3. 如果设备驱动程序或者文件系统的file_operations定义了mmap（）操作，则调用它
4. 将这个VMA插入进程的VMA链表中
```c
int munmap(caddr_t addr, size_t len );
@brief 解除映射
```


 