### 题干还原

```python
from Flag import FLAG
from Crypto.Util.number import bytes_to_long, getPrime

m = bytes_to_long(FLAG)

PrimeBits = 512
p, q = getPrime(PrimeBits), getPrime(PrimeBits)
n = p * q

assert m < n

def encrypt_RSA(m, e, n):
    return pow(m, e, n)

def decrypt_RSA(c, p, q):
    d = pow(e, -1, (p - 1)*(q - 1))
    return pow(c, d, n)


e = 65537
c = encrypt_RSA(m, e, n)
hint = (p + q) // e

with open(r"./crypto2-EzPrime/SourceCode/output.txt", "w") as f:
    print("Ok, Here is your public key:")
    print(f"{(n, e) = }", file=f)
    print("Ok, Here is your ciphertext:")
    print(f"{c = }", file=f)

    print("Well, well, I will give you some hints:")
    print(f"{hint = }", file=f)
```

### 解题思路

知道 `n = p * q`，从而我们只要知道 `p + q` 就可以解出 `p` 和 `q` 了。注意到 `hint` 只是抹去了 `p + q` 的末尾，因此只要遍历 `p + q` 的所有可能取值，尝试解出 `p` 和 `q` ，判断是否满足 `n = p * q` 即可。

### 解题代码

```python
import gmpy2
from Crypto.Util.number import long_to_bytes

with open(r'./crypto2-EzPrime/SourceCode/output2.txt', 'r') as f:
    exec(f.read().strip())

def decrypt_RSA(c, p, q):
    d = pow(e, -1, (p - 1)*(q - 1))
    return pow(c, d, n)

for r in range(e):
    p_plus_q = hint * e + r
    delta = p_plus_q ** 2 - 4 * n
    if delta < 0:
        continue
    p = (gmpy2.isqrt(delta) + p_plus_q) // 2
    if n % p == 0 and p > 1 and p < n:
        q = n // p
        m = decrypt_RSA(c, p, q)
        print(long_to_bytes(m))
        break
```