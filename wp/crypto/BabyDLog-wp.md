<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


### 题干还原

```python
from Flag import FLAG
from Crypto.Util.number import bytes_to_long, isPrime, getPrime
from random import randint

assert len(FLAG) == 20 + 24
flag = bytes_to_long(FLAG)

def get_prime(n = 32, RANGE = 16):
    a, p = 1, 1
    for _ in range(n): p *= randint(1 << RANGE, 1 << (RANGE + 1))
    while not isPrime(a * p + 1): a = getPrime(RANGE * RANGE)
    return a * p + 1 

p = get_prime()
g = bytes_to_long(b"20" + b"24")

with open(r"./crypto3-BabyDLog/SourceCode/output.txt", 'w') as f: 
    print(f"{p = }", file=f)
    print(f"{g = }", file=f)
    print(f"{pow(g, flag, p) = }", file=f)
```

### 解题思路

典型的 **Pohlig-Hellman** 算法：

1. 找到 $p-1$ 的因数分解，记为 $p = p_1^{e_1} \cdots p_k^{e_k}$
2. P-H算法的原理可参考 [这里](https://www.ruanx.net/pohlig-hellman/)
3. 因为限制了 `flag` 的大小，所以其实我们不必要把 `flag` 对每个 $p_i^{e_i}$ 的模数求出来就可以做 CRT。

### 解题脚本

```python
with open(r"./crypto3-BabyDLog/SourceCode/output.txt", "r") as f:
    p, G, H = [int(line.split("=")[1]) for line in f.readlines()]

print(f"{p = }")
print(f"{G = }")
print(f"{H = }")

from sympy import factorint, prod
from Crypto.Util.number import long_to_bytes
from itertools import product

def get_order(m, factorList, p):
    newfactorList = {}
    for q, alpha in factorList.items():
        b = pow(m, (p - 1) // pow(q, alpha), p)
        for j in range(alpha):
            if pow(b, pow(q, j + 1), p) == 1: newfactorList[q] = j + 1; break
    return newfactorList

def naive_dlp(g, h, p, r):
    if r.bit_length() > 32: print(r.bit_length()); return [0]
    res = []
    for i in range(r):
        if pow(g, i, p) == h % p:
            res.append(i)
    return res
        
def partial_dlp(g, h, q, alpha, order):
    e, base = order, 1
    new_g = pow(g, order // q, p)
    res = [0]
    while alpha:
        assert e % q == 0
        e = e // q
        tmp_res = []
        for ri in res:
            new_h = pow(h * pow(g, - ri, p), e, p)
            for pd in naive_dlp(new_g, new_h, p, q):
                tmp_res.append(ri + pd * base)
        base *= q
        alpha -= 1
        res = tmp_res.copy()
    return res

def crt_solve(modList, primeExpList):
    mod = prod(modList)
    return sum(x * pow(mod // pa, -1, pa) * (mod // pa) for x, pa in zip(primeExpList, modList)) % mod

factorList = factorint(p - 1)
newfactorList = get_order(G, factorList, p)
neworder = prod(q ** alpha for q, alpha in newfactorList.items())

primeExpList = product(*[
    partial_dlp(G, H, q, alpha, neworder) for q, alpha in newfactorList.items()
])

modList = [
    pow(p, alpha) for p, alpha in newfactorList.items() if p.bit_length() < 33
]

primeExpList = list(map(lambda l: l[:len(modList)], primeExpList))

for pl in primeExpList:
    msg = long_to_bytes(crt_solve(modList, pl))
    if b"ucatflags" in msg: print(msg); print(primeExpList.index(pl))
```

### 备注

这题出拐了，，，
- 模数出长了
- 原根能够被求出来
- 本来想的是直接dlp不能全求出来，结果还是出拐了，选一部分小素数还是能求出来