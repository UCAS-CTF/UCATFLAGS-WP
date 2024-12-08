# game4

```c
undefined8 FUN_0010137d(void)

{
  long in_FS_OFFSET;
  char local_3d;
  uint local_3c;
  char local_38 [40];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setbuf(stdout,(char *)0x0);
  setbuf(stdin,(char *)0x0);
  puts(
      "OK, Finally, You are here!\nWelcome! And\nYOU WILL NEVER GET THE FLAG!\nTell me your NAME!\nB Efore you GET c!r!a!z!y!\n"
      );
  for (local_3c = 0; local_3c < 0x1e; local_3c = local_3c + 1) {
    local_38[(int)local_3c] = 'a';
  }
  fgets(local_38,0x1e,stdin);
  puts("Interesting....\nNow, GAME START!!!!");
  while ((DAT_001041cc != 9 || (DAT_001041d0 != 9))) {
    __isoc99_scanf(&DAT_001020c3,&local_3d);
    if ((int)local_3d == local_38[0] + 0x14) {
      FUN_001012f4(0,0xffffffff);
    }
    else if ((int)local_3d == local_38[1] + -0x11) {
      FUN_001012f4(0xffffffff,0);
    }
    else if ((int)local_3d == local_38[2] + 0x12) {
      FUN_001012f4(0,1);
    }
    else if ((int)local_3d == local_38[3] + -0x16) {
      FUN_001012f4(1,0);
    }
    else if ((int)local_3d == local_38[4] + -8) {
LAB_00101525:
      if (local_10 == *(long *)(in_FS_OFFSET + 0x28)) {
        return 0;
      }
                    /* WARNING: Subroutine does not return */
      __stack_chk_fail();
    }
  }
  puts("YOU CRAZY!!!!");
  system("/bin/cat ../flag");
  goto LAB_00101525;
}
```

```c
void FUN_001012f4(int param_1,int param_2)

{
  param_1 = param_1 + DAT_001041cc;
  param_2 = param_2 + DAT_001041d0;
  if ((((-1 < param_1) && (param_1 < 10)) && (-1 < param_2)) &&
     ((param_2 < 10 && (*(int *)(&DAT_00104020 + ((long)param_2 * 10 + (long)param_1) * 4) == 0))))
  {
    DAT_001041cc = param_1;
    DAT_001041d0 = param_2;
  }
  return;
}
```

逆向分析可知其为一个迷宫，且方向移动与预输入的用户名有关（用户名前五位为 `crazy`时，上下左右移动分别对应 `w`、`s`、`a`、`d`）。

然后通过逆向分析可知 `DAT_00104020`为地图，且其为int类型为一个格点，大小为10*10。

```hex
                             INT_ARRAY_00104020                              XREF[6]:     FUN_00101209:00101273(*), 
                                                                                          FUN_00101209:0010127a(*), 
                                                                                          FUN_00101209:001012b1(*), 
                                                                                          FUN_00101209:001012b8(*), 
                                                                                          FUN_001012f4:0010135a(*), 
                                                                                          FUN_001012f4:00101361(*)  
        00104020 00 00 00        int[100]
                 00 01 00 
                 00 00 01 
           00104020 [0]                      0h,           1h,           1h,           1h,
           00104030 [4]                      0h,           1h,           1h,           1h,
           00104040 [8]                      1h,           1h,           0h,           0h,
           00104050 [12]                     1h,           1h,           0h,           1h,
           00104060 [16]                     0h,           0h,           1h,           0h,
           00104070 [20]                     1h,           0h,           1h,           1h,
           00104080 [24]                     0h,           1h,           0h,           1h,
           00104090 [28]                     1h,           0h,           1h,           0h,
           001040a0 [32]                     0h,           0h,           0h,           0h,
           001040b0 [36]                     0h,           1h,           1h,           0h,
           001040c0 [40]                     1h,           0h,           1h,           0h,
           001040d0 [44]                     1h,           0h,           1h,           0h,
           001040e0 [48]                     0h,           0h,           1h,           1h,
           001040f0 [52]                     0h,           0h,           1h,           0h,
           00104100 [56]                     1h,           0h,           1h,           0h,
           00104110 [60]                     1h,           0h,           0h,           0h,
           00104120 [64]                     1h,           0h,           0h,           0h,
           00104130 [68]                     1h,           0h,           1h,           1h,
           00104140 [72]                     0h,           1h,           1h,           1h,
           00104150 [76]                     0h,           1h,           0h,           0h,
           00104160 [80]                     0h,           0h,           0h,           0h,
           00104170 [84]                     0h,           1h,           0h,           1h,
           00104180 [88]                     0h,           1h,           0h,           0h,
           00104190 [92]                     1h,           1h,           1h,           1h,
           001041a0 [96]                     1h,           1h,           0h,           0h

```

二维数组在内存中按行存储，则可以得到以下为地图

```text
0, 1, 1, 1, 0, 1, 1, 1, 1, 1,
0, 0, 1, 1, 0, 1, 0, 0, 1, 0,
1, 0, 1, 1, 0, 1, 0, 1, 1, 0,
1, 0, 0, 0, 0, 0, 0, 1, 1, 0,
1, 0, 1, 0, 1, 0, 1, 0, 0, 0,
1, 1, 0, 0, 1, 0, 1, 0, 1, 0,
1, 0, 0, 0, 1, 0, 0, 0, 1, 0,
1, 1, 0, 1, 1, 1, 0, 1, 0, 0,
0, 0, 0, 0, 0, 1, 0, 1, 0, 1,
0, 0, 1, 1, 1, 1, 1, 1, 0, 0,
```

剩下的只需走出迷宫


```python
from pwn import *

# Connect to the server (change the IP and port to the correct values)

r = remote('127.0.0.1', 31051)

r.sendline(b"crazy")

r.sendline(b"sdssddddsssddwwddsssassd")

r.interactive()

```
