# Lab3 page tables

## Speed up system calls
1.在`proc.h`文件中声明`process`拥有新的成员`struct usyscall *usyscall`;

2.在`allocproc()`函数中为创建的新的process分配一页物理内存用来存放`usyscall`的内
容，并且完成初始化；

3.在`proc_pagetable()`函数中为2中分配的物理内存进行映射，注意`PTE`内容的填写；

4.在`freeproc()`函数中释放2中分配的物理内存；

5.在`proc_freepagetable()`中解除页表的映射；

## Print a page table
1. 在`defs.h`中声明`vmprint()`函数原型`vmprint(pagetable_t pagetable)`；

2. 在`vm.c`中实现函数`vmprint()`

3. 主要参考`freewalk`函数以及`递归`思想的理解。

## Detect which pages have been accessed
