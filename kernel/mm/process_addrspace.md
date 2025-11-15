# 进程地址空间

## API
```C
find_vma();
get_unmapped_area();
insert_vm_struct();

do_mmap();
```

## tools
**kmap**可以看进程地址空间map情况


## 内存描述符
```C
struct mm_struct {
    struct vm_area_struct *mmap;  // 指向线性区对象的链表头
    struct rb_root mm_rb;  // 维护线性区红黑树的根
    struct vm_area_struct *mmap_cache; // 最后一个引用的线性区对象-->提高查找速度
    unsigned long (*)() get_unmapped_area;  // 在进程地址空间搜索有效线性区间的方法
    void (*)() unmap_area;  // 释放线性地址区间时调用的方法
    // 内核从这个地址开始搜索进程地址空间中线性地址的空闲区间；初始化为user space地址空间的1/3(为预定义vma保留：.text, .data, .bss)
    unsigned long free_area_cache;
};
```
- 保存与进程地址空间有关的全部信息

## 线性区
```C
struct vm_area_struct {
    struct mm_struct *vm_mm;  // 指向内存描述符
    unsigned long vm_start;  // 线性区内第一个线性地址
    unsigned long vm_end;  // 线性区之后的第一个线性地址
    struct vm_area_struct *vm_next;  // 进程拥有的线性区链表中的下一个线性区
    pgprot_t vm_page_prot;  // 线性区中页框的访问权限
    unsigned long vm_flags;  // 线性区的标志
    struct rb_node vm_rb;  // 维护进程的线性区链表
    struct vm_operations_struct *vm_ops;  // 指向线性区的方法
    // 在映射文件中的偏移量；对匿名页，等于0或者vm_start/PAGE_SIZE
    unsigned long vm_pgoff;
};
```

线性区
- 表示一个线性地址区间
- 进程的线性区不会重叠
- 内核会把新分配的线性区和相邻的现有线性区合并(需要有相同的访问权限)
- 进程的所有线性区通过一个链表连接，按**内存地址升序**排列
- 两个线性区可以由未用的内存地址区隔开

### 线性区访问权限

```vm_flags```
- VM_READ/WRITE/EXEC/SHARED
- 页表标志的初值存放在```vm_page_prot```字段
- 不能把线性区访问权限直接转换成页保护位

### 线性区的处理

查找给定地址的**已分配的**最邻近区
```struct vm_area_struct *find_vma(struct mm_struct *mm, unsigned long addr);```
- 参数：进程内存描述符的地址；线性地址addr
-  查找线性区的vm_end大于addr的第一个线性区位置，返回线性区描述符的地址

**查找一个空闲的地址区间**
```unsigned long get_unmapped_area(struct file *file, unsigned long addr, unsigned long len, unsigned long pgoff, unsigned long flags)```

- 查找进程的地址空间，找到一个可以使用的vma
- 参数
    - len：指定区间的长度
    - 非空addr：指定必须从哪个地址开始进行查找；会检查addr是否在user space并且与页边界对齐
- 查找成功，返回新vma的起始地址；否则返回-ENOMEM
- 根据文件内存映射/匿名内存映射调用不同的实现
    - 调用文件实现的操作
    - 调用内存描述符的get_unmapped_area方法
        - arch_get_unmapped_area：从user space起始地址开始向高地址增长
        - arch_get_unmapped_area_topdown：从user space堆开始向低地址增长
- 搜索从最近被分配的线性区后面的线性地址开始（**free list分配**）
    - 顺序遍历所有的vma，直到满足**vma->start >= (addr + len)**

向内存描述符链表中插入一个线性区
```insert_vm_struct()```
- 通过get_unmapped_area找到空闲vma之后需要插入到mm_struct->mmap链表中管理；并插入红黑树
- 先找到插入位置，再调用```vma_link()```执行插入操作

**分配线性地址区间**
```unsigned long do_mmap(struct file *file, unsigned long addr, unsigned long len, unsigned long prot, unsigned long flags, unsigned long pgoff, unsigned long *populate, struct list_head *uf)```

- 为当前进程创建并初始化一个新的vma
- 分配成功后，可以把新的vma和进程已有的其他vma进行合并
- 参数：
    - file, offset：指定映射文件的位置
    - 非空addr：指定从何处开始查找空闲的区间
    - len：vma的长度
    - prot：指定线性区**包含的页的访问权限**(PROT_READ/WRITE/EXEC/NONE)
    - flag：指定线性区的其他标志(MAP_SHARED/PRIVATE/FIXED/ANON...)
        - MAP_FIXED规定vma的起始地址必须由参数addr指定
- 大致流程：
    - 检查参数
    - 调用get_unmapped_area获得新线性区的vma
    - 通过prot和flag计算新vma的标志
    - 调用vma_merge()检查是否有可合并的vma
    - 调用slab分配器分配vm_area_struct管理信息
    - 填充vma管理信息
    - 调用vma_link()把新线性区插入到vma链表和红黑树中
  
## page fault handler
