# game2

### 逆向

```c
undefined8 FUN_004019d6(void)

{
  int iVar1;
  long lVar2;
  long in_FS_OFFSET;
  undefined local_78 [32];
  undefined8 local_58;
  undefined6 local_50;
  undefined2 uStack_4a;
  undefined6 uStack_48;
  undefined8 local_42;
  undefined local_38 [40];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_58 = 0x6d3d6a21712b793d;
  local_50 = 0x6b6e796e252c;
  uStack_4a = 0x2811;
  uStack_48 = 0x3d3e142e6827;
  local_42 = 0x373c6e7b13777b;
  FUN_004057a0(
              "Little CTFer!\nYou seemed to pass the first challenge easily!\nWell done!\nBut can yo u pass the second challenge?\nI beleive this is 7! piece of cake for you!\nGood appeti te!\n"
              );
  FUN_00405410(local_78,0x1e,PTR_DAT_004ad6d8);
  lVar2 = thunk_FUN_00412e70(local_78);
  if (lVar2 == 0x1d) {
    FUN_004018e5(local_78,local_38,7);
    iVar1 = thunk_FUN_00412ee0(local_38,&local_58,0x1e);
    if (iVar1 == 0) {
      FUN_004057a0("Success!");
      FUN_00405180("/bin/cat flag2.txt");
    }
    else {
      FUN_004057a0("Fit But Wrong!");
    }
  }
  else {
    FUN_004057a0("Not Fit!\n");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    FUN_00420d30();
  }
  return 0;
}
```

main函数中调用了 `FUN_004018e5`函数，该函数对输入的字符串 `local_38`进行了异或和移位运算，然后与 `local_58`进行比较，比较结果为0则输出flag

```c
void FUN_004018e5(long param_1,long param_2,char param_3)

{
  ulong uVar1;
  long in_FS_OFFSET;
  int local_2c;
  undefined4 local_27;
  undefined3 uStack_23;
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  local_27 = 0x53414355;
  uStack_23 = 0x535245;
  local_2c = 0;
  while( true ) {
    uVar1 = thunk_FUN_00412e70(param_1);
    if (uVar1 <= (ulong)(long)local_2c) break;
    uVar1 = (ulong)local_2c;
    *(byte *)(param_2 + local_2c) =
         (*(byte *)((long)&local_27 + uVar1 + ((uVar1 - uVar1 / 7 >> 1) + uVar1 / 7 >> 2) * -7) ^
         *(byte *)(param_1 + local_2c)) + param_3;
    local_2c = local_2c + 1;
  }
  *(undefined *)(param_2 + local_2c) = 0;
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    FUN_00420d30();
  }
  return;
}
```

异或运算的key为 `local_27`，`local_27`的值为 `0x53414355535245`，考虑小端序，则可知 `key`为 `UCASERS`

移位运算体现在

```c
uVar1 = (ulong)local_2c;
*(byte *)(param_2 + local_2c) =
        (*(byte *)((long)&local_27 + uVar1 + ((uVar1 - uVar1 / 7 >> 1) + uVar1 / 7 >> 2) * -7) ^
        *(byte *)(param_1 + local_2c)) + param_3;
```

的 `+param_3`上，`param_3`的值为 `7`，所以移位运算为 `+7`

### 解题

首先要获取预定义的字符数组 `local_58`，然后对其作逆向的移位与异或运算，得到正确输入

```c
void decrypt(char *input, char *output, char key) {
  int i;
  char xor_key[] = {'U', 'C', 'A', 'S', 'E', 'R', 'S'};
  for (i = 0; i < strlen(input); i++) {
    output[i] = (input[i] - key) ^ xor_key[i % sizeof(xor_key)];
    printf("%c\n",output[i]);
  }
  output[i] = '\0';
}
```



`solve.c`

```c
#include <stdio.h>
#include <string.h>

#define PASSCODELEN 30

void decrypt(char *input, char *output, char key) {
  int i;
  char xor_key[] = {'U', 'C', 'A', 'S', 'E', 'R', 'S'};
  for (i = 0; i < strlen(input); i++) {
    output[i] = (input[i] - key) ^ xor_key[i % sizeof(xor_key)];
  }
  output[i] = '\0';
}

int main() {
  char passcode[PASSCODELEN] = {0x3d, 0x79, 0x2b, 0x71, 0x21, 0x6a, 0x3d, 0x6d,
                                0x2c, 0x25, 0x6e, 0x79, 0x6e, 0x6b, 0x11, 0x28,
                                0x27, 0x68, 0x2e, 0x14, 0x3e, 0x3d, 0x7b, 0x77,
                                0x13, 0x7b, 0x6e, 0x3c, 0x37, 0x0};
  char str_decrypted[PASSCODELEN];
  decrypt(passcode, str_decrypted, 7);
  printf("Decrypted passcode: %s\n", str_decrypted);

}

```

```python
from pwn import *

# Connect to the server (change the IP and port to the correct values)

r = remote('127.0.0.1', 31049)

r.sendline(b"c1e9_1e3f_4757_ba2b_dc71_15fe")

r.interactive()

```
