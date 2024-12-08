# game1

### 逆向

```c
undefined8 main(void)

{
  int iVar1;
  size_t sVar2;
  long in_FS_OFFSET;
  char local_58 [32];
  undefined8 local_38;
  undefined6 local_30;
  undefined2 uStack_2a;
  undefined6 uStack_28;
  undefined8 local_22;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_38 = 0x3163612d65373365;
  local_30 = 0x376338632d35;
  uStack_2a = 0x342d;
  uStack_28 = 0x30382d383464;
  local_22 = 0x306231362d3264;
  puts(
      "Welcome! Little CTFer!\nI know you want Flag!\nBut you need to pass the challenge first!\nJus t crack it!\n"
      );
  fgets(local_58,0x1e,(FILE *)stdin);
  sVar2 = strlen(local_58);
  if (sVar2 == 0x1d) {
    iVar1 = strncmp(local_58,(char *)&local_38,0x1e);
    if (iVar1 == 0) {
      puts("Success!");
      system("/bin/cat flag1.txt");
    }
    else {
      puts("Fit But Wrong!");
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
```

可知其为一个简单的字符串比较题目，只需要将输入的字符串与 `local_38`进行比较即可

更改 `local_38`的变量类型即可看到flag

```c
undefined8 main(void)
{
  long lVar1;
  int iVar2;
  size_t sVar3;
  long in_FS_OFFSET;
  char local_58 [32];
  char local_38 [30];
  
  lVar1 = *(long *)(in_FS_OFFSET + 0x28);
  local_38[0] = 'e';
  local_38[1] = '3';
  local_38[2] = '7';
  local_38[3] = 'e';
  local_38[4] = '-';
  local_38[5] = 'a';
  local_38[6] = 'c';
  local_38[7] = '1';
  local_38[8] = '5';
  local_38[9] = '-';
  local_38[10] = 'c';
  local_38[11] = '8';
  local_38[12] = 'c';
  local_38[13] = '7';
  local_38[14] = '-';
  local_38[15] = '4';
  local_38[16] = 'd';
  local_38[17] = '4';
  local_38[18] = '8';
  local_38[19] = '-';
  local_38[20] = '8';
  local_38[21] = '0';
  local_38[22] = 'd';
  local_38[23] = '2';
  local_38[24] = '-';
  local_38[25] = '6';
  local_38[26] = '1';
  local_38[27] = 'b';
  local_38[28] = '0';
  local_38[29] = '\0';
  puts(
      "Welcome! Little CTFer!\nI know you want Flag!\nBut you need to pass the challenge first!\nJus t crack it!\n"
      );
  fgets(local_58,0x1e,(FILE *)stdin);
  sVar3 = strlen(local_58);
  if (sVar3 == 0x1d) {
    iVar2 = strncmp(local_58,local_38,0x1e);
    if (iVar2 == 0) {
      puts("Success!");
      system("/bin/cat flag1.txt");
    }
    else {
      puts("Fit But Wrong!");
    }
  }
  else {
    puts("Not Fit!\n");
  }
  if (lVar1 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```


脚本：

```python
from pwn import *
# Connect to the server (change the IP and port to the correct values)
r = remote('127.0.0.1', 31048)
r.sendline(b"e37e-ac15-c8c7-4d48-80d2-61b0")
r.interactive()
```
