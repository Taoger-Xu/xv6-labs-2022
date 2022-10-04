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

主要是实现系统调用，关键难点是用户va和内核地址空间的不同的理解，以及指针的理解

1.`user/pgtbltest.c`文件下的测试函数如下:
发现在用户空间通过buf = malloc(32 * PGSIZE)分配得到了32个page，buf这里是用户空间的va，abits是用户空间存储答案的va。
```c
void
pgaccess_test()
{
  char *buf;
  unsigned int abits;
  printf("pgaccess_test starting\n");
  testname = "pgaccess_test";
  buf = malloc(32 * PGSIZE);
  if (pgaccess(buf, 32, &abits) < 0)
    err("pgaccess failed");
  buf[PGSIZE * 1] += 1;
  buf[PGSIZE * 2] += 1;
  buf[PGSIZE * 30] += 1;
  // 调试信息
  printf("adits is:%x\n",abits);
  if (pgaccess(buf, 32, &abits) < 0)
    err("pgaccess failed");
  if (abits != ((1 << 1) | (1 << 2) | (1 << 30)))
    err("incorrect access bits set");
  free(buf);
  printf("pgaccess_test: OK\n");
}

```
2. 系统调用处理函数在`sysproc.c`文件中`sys_pgaccess`函数处理:
关键是在内核空间定义`uva_addr，len，bitmask`接受用户空间传来的参数，
注意`argaddr`和`argint`的区别,`argaddr`表示传来的是一个地址，`argint`表示传来的是一个`int`。然后跳转到proc.c中的pgaccess中进行处理，注意
pgaccess((void*)uva_addr, len, (void*)bitmask)这里参数的都是内核空间中的变量，但是值和用户空间中相同。
```c
int
sys_pgaccess(void)
{
  // lab pgtbl: your code here.

  uint64 uva_addr; //参数一，需要检查用户空间va的addrress
  int len; // 参数二，需要检查的page页数
  uint64 bitmask; // 参数三，最后检查结果需要存放的地址，也是用户的va的address

  //接受参数并且检查
  argaddr(0, &uva_addr);
  argint(1, &len);
  argaddr(2, &bitmask);
  
  return pgaccess((void*)uva_addr, len, (void*)bitmask);
}
```
3. 注意在`defs.h`声明新函数和`risv.h`定义`PTE_A`
4. `pgaccess`的实现：首先获取当前`process`的`page table`，然后用传来的与用户空间uva_address值相同的start作为va开始查找，一共`len`个页面，将答案暂时存在内核空间的ans。使用`walk()`找到va对应的pte，读取pte中的flag并且记录。最后的关键是使用`copyout`将内核中暂存的ans中的内容拷贝到传进来的用户空间位于bitmask的位置，注意类型转换。

```c
int
pgaccess(void *start, int len, void *bitmask)
{
  struct proc *p = myproc();
  if (p == 0){
    return -1;
  }
  pagetable_t pagetable = p->pagetable;
  int ans = 0;
  // 依次检查每一个page
  for(int i = 0; i < len; i++){
    pte_t *pte;
    //((uint64)pg) + (uint64)PGSIZE * i
    pte = walk(pagetable, (uint64)start + (uint64)PGSIZE * i, 0);
    if (pte !=0 &&((*pte) & PTE_A)){
      ans |= (1 << i);
      *pte ^= PTE_A;
    }
  }
   // copyout
  return copyout(pagetable, (uint64)bitmask, (char *)&ans, sizeof(ans));
}
```
