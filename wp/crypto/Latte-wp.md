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


### 题目源码

`Latte.py`
```python
import os
from Flag import FLAG
from MyLatteRepo import CUP, SPOON
from Crypto.Util.number import bytes_to_long, getRandomNBitInteger as RandBitInt

Amount = 96
Milk = os.urandom(16) * 8
Espresso = FLAG + os.urandom(Amount - len(FLAG))

with open(r"output.txt", "w") as f:
    print(f"Secret Recipe = {[tuple(RandBitInt(Amount) * TableWare + RandBitInt(Amount) for _ in range(2)) for TableWare in (CUP, SPOON)]}", file=f)
    print(f"Thick Latte = {pow(SPOON, bytes_to_long(Espresso + Milk), CUP ** 2)}", file=f)
```

`MyLatteRepo.py`
```python
from Crypto.Util.number import getPrime
CUP, SPOON = getPrime(1024), getPrime(1023)
```

### 思路分析

#### ***Point 1***：AGCD问题

首先对于 `Secret Recipe`：

```python
print(f"Secret Recipe = {[tuple(RandBitInt(Amount) * TableWare + RandBitInt(Amount) for _ in range(2)) for TableWare in (CUP, SPOON)]}", file=f)
```

设 `CUP` 为 $u$，`SPOON` 为 $s$，那么给到的信息就是：

$$
\begin{cases}
a_1 u + b_1 = h_1\\
a_2 u + b_2 = h_2\\
\end{cases}
$$

以及

$$
\begin{cases}
a_1' s + b_1' = h_1'\\
a_2' s + b_2' = h_2'\\
\end{cases}
$$

其中 $a_i$ 、 $b_i$ 、$a_i'$ 和 $b_i'$ 为**小的随机数**，$h_i$ 和 $h_i'$ 是已知的。

这是**典型的 AGCD 问题**，比如我们考虑针对 $u$ 的这组等式

注意到 

$$
a_2 h_1 - a_1 h_2 = a_2 b_1 - b_2 a_1
$$

等式右边是一个小数据

那么可以构造如下格子：

$$
\begin{pmatrix}
a_2 &-a_1
\end{pmatrix}
\cdot 
\begin{pmatrix}
h_1 & 1 & 0 \\
h_2 & 0 & 1
\end{pmatrix}
=
\begin{pmatrix}
a_2 b_1 - b_2 a_1 & a_2 & -a_1
\end{pmatrix}
$$

右边是一个短向量，所以对左边的格子 $\begin{pmatrix}h_1 & 1 & 0 \\\\ h_2 & 0 & 1\end{pmatrix}$ 作 `LLL` 格基约化即可。

这一层的解题代码如下：

```python

with open("./output.txt", "r") as f:
    s1, s2 = f.readlines()
    sp, sg = eval(s1.split("= ")[1])
    secret = int(s2.split("= ")[1])

def getSec(h1, h2, bits):
    mat = matrix(ZZ, [[h1, 1, 0], [h2, 0, 1]])
    mat = mat.LLL()
    t, a2, a1 = mat[0]
    a2, a1 = abs(a2), abs(a1)
    pre_p = h1 // a1
    for i in range(pre_p >> bits, pre_p >> (bits - 1)):
        q = pre_p // (i + 1)
        for bias in range(-4, 5):
            r = q + bias
            if is_prime(r) and r.bit_length() == bits: return r
    print("Failed")

p, g = getSec(*sp, 1024), getSec(*sg, 1023)
print(p)
print(g)
```

#### ***Point 2***：格分析、 $p^2$ 下的离散对数

由于

```python
Milk = os.urandom(16) * 8
```
所以可以设
设随机产生的部分为 $x$， `A = bytes_to_long((b"\x00"*15 + b"\x01") * 8)` 则  

```python
assert bytes_to_long(Milk) == x * A
``` 

简记为数学公式： $ \text{Milk} = A\cdot x $    

其中 $ x \sim 2^{128} $

进一步观察第二个式子，根据 `len(Milk) == 128` ，同时设 `y = bytes_to_long(FLAG)`，那么有数学关系：    

$ s = \text{Thick Latte} = g^{2^{128\times 8}y + x\cdot A}\pmod{p^2} $    

记 $ B = 2^{128\times 8} $ 则化为：    

$$
 \begin{aligned}
s & = g^{Ax+By}\pmod{p^2} \\
  & = \left(g^A\right)^x \cdot \left(g^B\right)^y\pmod{p^2}
\end{aligned}
$$    

其中 $ x\sim 2^{128} $ ， $ y\sim 2^{768} $ ， $ p \sim 2^{1024} $    

考虑如下推导：    

$$
s' = s^{p-1} = \left(g^{A(p-1)}\right)^x\cdot \left(g^{B(p-1)}\right)^y\pmod{p^2} 
$$    

注意，根据费马小定理，$g^{A(p-1)}\equiv 1\pmod{p}$ ，以及 $ g^{B(p-1)}\equiv 1\pmod{p} $ ，则说明 $ g^{A(p-1)} - 1 $ 和 $ g^{B(p-1)} - 1 $ 均为 $ p $ 的倍数。

设 $ g^{A(p-1)} - 1 = ap $ ， $ g^{B(p-1)} - 1 = bp $ ，则根据二项式定理有：

$$
\begin{aligned} 
s' & = (ap+1)^x\cdot (bp+1)^y &\pmod{p^2} \\
  & = (xap+1)\cdot (ybp+1) &\pmod{p^2} \\
  & = (ax+by)\cdot p + 1 &\pmod{p^2}
\end{aligned} 
$$ 

即：

$$
S' = \dfrac{s' - 1}{p} = ax + by \pmod{p} 
$$ 

其中 $ x\sim 2^{128} $ ， $ y\sim 2^{768} $ ， $ p \sim 2^{1024} $    

进一步转化为：    

$$
-x = - a^{-1}S' + a^{-1}b\cdot y\pmod{p} 
$$

设 $ S = a^{-1}S\bmod{p} $ ， 以及 $ C = a^{-1}b\bmod{p} $ 

得到 $ -x = - S + Cy - kp $ 很小，其中  $ x\sim 2^{128} $ ， $ k\sim y\sim 2^{768} $ ， $ S \sim C \sim p \sim 2^{1024} $     

因此可以构造一个矩阵

$$
M = \begin{pmatrix} \dfrac{1}{2^{768 - 128}} & 0 & C \\ 0 & 2^{128} & S \\ 0 & 0 & p \end{pmatrix} 
$$ 

满足：

$$
\begin{pmatrix} y & -1 & k\end{pmatrix} \cdot M = \begin{pmatrix} \dfrac{y}{2^{640}} \\\\ -2^{128} \\\\ -x \end{pmatrix} 
$$  

其中 $ \dfrac{1}{2^{640}} $ 和 $2^{128}$ 是为了控制最后向量的大小，和矩阵左乘系数恰好是我们想要的，考察选手对于利用格规约解决小未知数方程的手段，对于格子构造的理解。

```python
unit = b"\x00" * 15 + b"\x01"
A = bytes_to_long(unit * 8)

unit = b"\x00" * 16
B = bytes_to_long(b"\01" + unit * 8)

ap = pow(g, A * (p - 1), p ** 2) - 1
bp = pow(g, B * (p - 1), p ** 2) - 1

assert int(ap) % p == 0 
assert int(bp) % p == 0 

a, b = int(ap) // p, int(bp) // p
C = (b * inverse(a, p)) % p
S = int(pow(int(secret), p - 1, p ** 2)) - 1
assert int(S) % p == 0
S = ((int(S) // p) * inverse(a, p)) % p

M = matrix([[1/2**(768 - 128), 0, C], [0, 2**128, S], [0, 0, p]])
ML = M.LLL()
print(ML)
for v in ML:
    the_flag = abs((v * (2 ** (768 - 128)))[0])
    print(long_to_bytes(int(the_flag)))
```

### 完整解题代码

```python
from Crypto.Util.number import *

with open("./output.txt", "r") as f:
    s1, s2 = f.readlines()
    sp, sg = eval(s1.split("= ")[1])
    secret = int(s2.split("= ")[1])

def getSec(h1, h2, bits):
    mat = matrix(ZZ, [[h1, 1, 0], [h2, 0, 1]])
    mat = mat.LLL()
    t, a2, a1 = mat[0]
    a2, a1 = abs(a2), abs(a1)
    pre_p = h1 // a1
    for i in range(pre_p >> bits, pre_p >> (bits - 1)):
        q = pre_p // (i + 1)
        for bias in range(-4, 5):
            r = q + bias
            if is_prime(r) and r.bit_length() == bits: return r
    print("Failed")

p, g = getSec(*sp, 1024), getSec(*sg, 1023)
print(p)
print(g)

unit = b"\x00" * 15 + b"\x01"
A = bytes_to_long(unit * 8)

unit = b"\x00" * 16
B = bytes_to_long(b"\01" + unit * 8)

ap = pow(g, A * (p - 1), p ** 2) - 1
bp = pow(g, B * (p - 1), p ** 2) - 1

assert int(ap) % p == 0 
assert int(bp) % p == 0 

a, b = int(ap) // p, int(bp) // p
C = (b * inverse(a, p)) % p
S = int(pow(int(secret), p - 1, p ** 2)) - 1
assert int(S) % p == 0
S = ((int(S) // p) * inverse(a, p)) % p

M = matrix([[1/2**(768 - 128), 0, C], [0, 2**128, S], [0, 0, p]])
ML = M.LLL()
print(ML)
for v in ML:
    the_flag = abs((v * (2 ** (768 - 128)))[0])
    print(long_to_bytes(int(the_flag)))
```
