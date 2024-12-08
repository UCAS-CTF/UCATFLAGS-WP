# game3

### 逆向

```c
undefined8 FUN_00101727(void)

{
  char cVar1;
  size_t sVar2;
  long in_FS_OFFSET;
  char local_38 [40];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setbuf(stdout,(char *)0x0);
  setbuf(stdin,(char *)0x0);
  puts(
      "L~i~t~t~l~e C~T~F~e~r~~~~\nYou beat the second challenge!~\nI start to afraid of you!~\nSo I raised the fog for you!!!~~~\nSEE! SEE! SEE!!!!\nI! CAN! SEE! FOREVER!\n"
      );
  fgets(local_38,0x1e,stdin);
  sVar2 = strlen(local_38);
  if (sVar2 == 0x1d) {
    cVar1 = FUN_00101578(local_38);
    if (cVar1 == '\0') {
      puts("Fit But Wrong!");
    }
    else {
      puts("Success!");
      system("/bin/cat ../flag");
    }
  }
  else {
    puts("Not Fit!\n");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}

```c
undefined8 FUN_00101578(char *param_1)

{
  char cVar1;
  int iVar2;
  size_t sVar3;
  undefined8 uVar4;
  int iVar5;
  int iVar6;
  int local_20;
  
  sVar3 = strlen(param_1);
  local_20 = 0;
  while( true ) {
    if ((int)sVar3 / 2 <= local_20) {
      uVar4 = FUN_00101209(param_1,"ucatflags");
      return uVar4;
    }
    if (param_1[local_20] != param_1[(long)((int)sVar3 - local_20) + -1]) break;
    local_20 = local_20 + 1;
  }
  cVar1 = *param_1;
  *param_1 = -9;
  iVar5 = (*param_1 + 2) % 3;
  iVar2 = (iVar5 * 3) / (int)*param_1;
  iVar6 = (*param_1 + iVar5 + iVar2 + 4) % 3;
  if (*param_1 * iVar5 * iVar2 * iVar6 + *param_1 + iVar5 + iVar2 + iVar6 == 0) {
    *param_1 = '\0';
  }
  *param_1 = cVar1;
  return 0;
}


```c
undefined8 FUN_00101209(char *param_1,char *param_2)

{
  char cVar1;
  int iVar2;
  int iVar3;
  int iVar4;
  int iVar5;
  int iVar6;
  size_t __n;
  undefined4 extraout_var;
  int iVar7;
  int iVar8;
  int iVar9;
  
  cVar1 = *param_1;
  *param_1 = '8';
  iVar2 = (*param_1 + 2) % 3 + -0x23;
  iVar3 = (iVar2 * 3) / (int)*param_1 + -0x2e;
  iVar7 = (*param_1 + iVar2 + iVar3 + 0x27) % 3;
  iVar8 = *param_1 * iVar2 * iVar3 * iVar7 + -0x47;
  iVar4 = (((int)*param_1 + iVar2 / iVar3) - iVar7 * iVar8) + -0x2a;
  iVar9 = (((iVar3 * iVar7) / iVar8 + (*param_1 - iVar2)) - iVar4) + -0x42;
  iVar5 = (((*param_1 * iVar2 - iVar3) + iVar7 / iVar8) - iVar4 * iVar9) + -2;
  iVar6 = ((((*param_1 + iVar2) - iVar3 * iVar7) + iVar8 / iVar4) - iVar9 * iVar5) + 8;
  if ((((*param_1 - iVar2) + iVar3 / iVar7) - iVar8 * iVar4) + iVar9 % iVar5 + iVar6 * -9 +
      *param_1 + iVar2 + iVar3 + iVar7 + iVar8 + iVar4 + iVar9 + iVar5 + iVar6 == 0) {
    *param_1 = '\0';
  }
  *param_1 = cVar1;
  __n = strlen(param_2);
  iVar2 = strncmp(param_1,param_2,__n);
  return CONCAT71((int7)(CONCAT44(extraout_var,iVar2) >> 8),iVar2 == 0);
}

```

### 分析

这个程序的主要功能是读取用户输入的字符串，并根据特定条件判断输入是否正确。以下是详细的分析步骤：

1. **主函数 `FUN_00101727`**：

   - 设置标准输出和标准输入的缓冲区为无缓冲。
   - 打印一段提示信息。
   - 使用 `fgets` 函数读取最多30个字符的输入到 `local_38` 数组中。
   - 计算输入字符串的长度，如果长度为29（0x1d），则调用 `FUN_00101578` 函数进行进一步验证。
   - 如果 `FUN_00101578` 返回非零值，表示验证成功，打印 "Success!" 并执行 `system("/bin/cat ../flag")` 命令显示 flag；否则打印 "Fit But Wrong!"。
   - 如果输入长度不为29，打印 "Not Fit!"。
2. **函数 `FUN_00101578`**：

   - 计算输入字符串的长度，并初始化 `local_20` 为0。
   - 进入一个循环，检查输入字符串是否是回文（即正读和反读相同）。
   - 如果是回文，调用 `FUN_00101209` 函数进行进一步验证。
   - 如果不是回文，最终返回0。（中间的复杂计算对结果无影响）
3. **函数 `FUN_00101209`**：

   - 通过分析可知，中间一系列复杂的计算并不会影响最终的结果，因此可以忽略这部分计算。
   - 比较输入字符串是否以 "ucatflags" 是否开头，返回比较结果。

### 解题思路

则可知其接受一个以 "ucatflags" 开头的回文字符串，即可得到 flag。


```python
from pwn import *

# Connect to the server (change the IP and port to the correct values)

r = remote('127.0.0.1', 31050)

r.sendline(b"ucatflags11111111111sgalftacu")

r.interactive()

```
