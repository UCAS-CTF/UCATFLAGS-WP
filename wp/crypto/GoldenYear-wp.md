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

```python
# sagemath
from Crypto.Util.number import *
from os import urandom
from Flag import FLAG, MAXLEN

New_Year = 2024
FLAG = FLAG + urandom(MAXLEN - len(FLAG))

# Verify the Goldbach's Conjecture
Goldbach_christian = [(p, New_Year - p) for p in sieve_base if p <= New_Year // 2 and isPrime(New_Year - p)]

# 2024 is a Leap Year!
Happy_2024, Haqqy_2024 = getPrime(New_Year // 4), getPrime(New_Year // 4)
while (Happy_2024 % 3 == Haqqy_2024 % 3):
    Happy_2024 = getPrime(New_Year // 4)
Hanny_2024 = Happy_2024 * Haqqy_2024

# The exuberant new year of 2024 will bring us joy and excitement!
p, q = choice(Goldbach_christian)
Exuberant_p = EllipticCurve(GF(Happy_2024), [0, p])
Exuberant_q = EllipticCurve(GF(Haqqy_2024), [0, q])

# Happy New Year!
m = bytes_to_long(FLAG)
with open(r"./output.txt", "w") as f:
    gift1 = bytes_to_long(str(Exuberant_p.order() + q).encode()) * bytes_to_long(str(Haqqy_2024 + p).encode())
    gift2 = bytes_to_long(str(Happy_2024 + q).encode()) * bytes_to_long(str(Exuberant_q.order() + p).encode())
    print(f"I'll give the gifts to U: {Hanny_2024, gift1, gift2}", file = f)
    red_envelope1 = pow(Happy_2024 * m + Exuberant_p.order(), Haqqy_2024, Hanny_2024)
    red_envelope2 = pow(Haqqy_2024 * m + Exuberant_q.order(), Happy_2024, Hanny_2024)
    print(f"OKOK ~ I'll give U both my red envelopes: {(red_envelope1 + red_envelope2) % Hanny_2024}", file = f)
```

### 解题思路

#### ***Point 1***：椭圆曲线基本常识

关注到代码片段：
```python
Happy_2024, Haqqy_2024 = getPrime(New_Year // 4), getPrime(New_Year // 4)
while (Happy_2024 % 3 == Haqqy_2024 % 3):
    Happy_2024 = getPrime(New_Year // 4)

p, q = choice(Goldbach_christian)
Exuberant_p = EllipticCurve(GF(Happy_2024), [0, p])
Exuberant_q = EllipticCurve(GF(Haqqy_2024), [0, q])
```

此处素数模3是一个十分重要的提示，根据有限域上椭圆曲线点群的相关结论，如果素数 $p \equiv 2  \pmod{3}$ ，那么 $ \mathbb{F}_p $ 上形如 $ y^2 = x^3 + C $ 的椭圆曲线的阶一定满足 $ \lvert E(\mathbb{F}_p)\rvert = p + 1 $ 。因此上述代码一定满足下列条件：

```python
    assert (Exuberant_p.order() == Happy_2024 + 1 and Exuberant_q.order() != Haqqy_2024 + 1) or (Exuberant_p.order() != Happy_2024 + 1 and Exuberant_q.order() == Haqqy_2024 + 1)
```

证明 $p \equiv 2 \pmod{3}$ 的情况：    
根据熟知的有限域上椭圆曲线点的计算公式（公式依据就是遍历每个 $ x $ ，计算 $ y $ 可以开平方的数量）：    

$$ 
\begin{aligned}
\#E(\mathbb{F}_p) &= 1 + \sum\limits_{k\in\mathbb{F}_p} \left(\left(\dfrac{k^3+c}{p}\right) + 1\right) \\    
&= p + 1 + \sum\limits_{k\in\mathbb{F}_p} \left(\dfrac{k^3+c}{p}\right) \\ 
&= p + 1 + \sum\limits_{k\in\mathbb{F}_p} \left(\dfrac{k}{p}\right) \\ 
&= p + 1 \end{aligned}
$$ 

注：
1. 考虑 $ p \equiv 2\pmod{3} $，不考虑$k = 0$ ，由于 $ k_1 ^ 3 + c = k_2 ^ 3 + c $ $ \Leftrightarrow $ $ k_1 ^ 3 = k_2 ^ 3 $ ，而 $ p - 1 \equiv 1\pmod{3} $ ，根据费马小定理 $ k_1 ^ {p-1} = k_2^{p-1} = 1 $ ，那么 $ k_1 ^ 3 + c = k_2 ^ 3 + c $ $ \Leftrightarrow $ $ k_1^{3\cdot \frac{p-2}{3}} = k_2^{3\cdot \frac{p-2}{3}} $ $ \Leftrightarrow $ $ k_1 = k_2 $ 。因此 $ k^3+c $ 恰好构成 $ p $ 的一个完全剩余系。（听起来高级一点的说法就是 $ x \mapsto x^3 + c $ 为 $ \mathbb{F}_p^{*} $ 上的一个内自同构）    

2. 二次剩余和二次非剩余的数量相等；$0$ 的勒让德符号值为 $0$。

#### ***Point 2***：广度优先搜索+爆破

记 $p_H$ 和 $q_H$ 分别表示 `Happy_2024` 和 `Haqqy_2024` ，$n_H$ 表示 `Hanny_2024` ，另外记 $a$ 和 $b$ 分别表示 `p` 和 `q`     
$p_H, q_H$ 在十进制下可以被写成如下形式：    

$$
\begin{aligned}
p_H &= a_0 + a_1 10 + a_2 10^2 + \cdots \\
q_H &= b_0 + b_1 10 + b_2 10^2 + \cdots
\end{aligned}
$$

记 `f = lambda x: bytes_to_long(str(x).encode())` 为 $f$ 因此 $f(x)$ 就可以写成：

$$ f(x) = (48 + x_0) + (48 + x_1) 256 + (48 + x_2) 256^2 + \cdots $$

根据上一条对于有限域上椭圆曲线点群阶的讨论，不妨设 `Exuberant_p.order() == Happy_2024 + 1` ，记 $ n_g $ 表示这里的 `gift1 = f(Exuberant_p.order() + q) * f(Haqqy_2024 + p)` ，那么就有：

$$
\begin{aligned}
n_H &= p_H \cdot q_H \\
    &= (p_0 + p_1 10 + p_2 10^2 + \cdots)\cdot(q_0 + q_1 10 + q_2 10^2 + \cdots) \\
n_g &= f(p_H + 1 + b)\cdot f(q_H + a) \\
    &= f(p_H')\cdot f(q_H') \\
    &= ((48 + p_0') + (48 + p'_1) 256 + (48 + p_2') 256^2 + \cdots)\cdot((48 + q_0') + (48 + q_1') 256 + (48 + q_2') 256^2 + \cdots)
\end{aligned}
$$

考虑 $ n_H \equiv (\sum\limits_{j=0}^{i-1} p_j 10^j) \cdot (\sum\limits_{j=0}^{i-1} q_j 10^j) \pmod{10^i} $ 以及 $ n_g \equiv (\sum\limits_{j=0}^{i-1} (48+p_j') 256^j) \cdot (\sum\limits_{j=0}^{i-1} (48+q_j') 256^j) \pmod{256^i} $ 从低到高逐数位恢复即可。    

这里可以使用递推或者递归实现，但是递归实现小心爆栈。

#### ***Point 3***：小密钥空间

```python
New_Year = 2024
Goldbach_christian = [(p, New_Year - p) for p in sieve_base if p <= New_Year // 2 and isPrime(New_Year - p)]
```

根据上述代码可知，列表 `Goldbach_christian` 空间是很小的，因此枚举即可

> 至此已经可以求出两个素数了，下面先给出一部分代码。

```python
from Crypto.Util.number import *

New_Year = 2024
Goldbach_christian = [(p, New_Year - p) for p in sieve_base if p <= New_Year // 2 and isPrime(New_Year - p)]

def factor(nh, ng, a, b, f = lambda x: bytes_to_long(str(x).encode())):
    nh_p = None
    def test_digits(ps, qs):
        nonlocal nh_p
        if nh_p is not None:
            return False
        p = sum([pi * 10 ** i for i, pi in enumerate(ps)])
        p_with_a = p + a
        new_ps = [int(i) for i in str(p_with_a).zfill(len(ps))[::-1]][:len(ps)]
        pp = sum([(48 + pi) << (i * 8) for i, pi in enumerate(new_ps)])
        q = sum([pi * 10 ** i for i, pi in enumerate(qs)])
        q_with_b = q + b
        new_qs = [int(i) for i in str(q_with_b).zfill(len(qs))[::-1]][:len(qs)]
        qq = sum([(48 + qi) << (i * 8) for i, qi in enumerate(new_qs)])
        if p != 0 and p != 1 and nh % p == 0:
            nh_p = p
            return False
        m1 = 10 ** len(ps)
        m2 = 1 << (len(qs) * 8)
        return (p * q) % m1 == nh % m1 and (pp * qq) % m2 == ng % m2

    stack = [([], [])]
    while stack:
        ps, qs = stack.pop()
        stack += [(ps + [i], qs + [j]) for i in range(10) for j in range(10) if test_digits(ps + [i], qs + [j])]

    ng_p = f(nh_p + a)
    assert ng % ng_p == 0
    return nh_p, nh // nh_p

New_Year = 2024
Goldbach_christian = [(p, New_Year - p) for p in sieve_base if p <= New_Year and isPrime(New_Year - p)]
def get_factors(n, nn, ABlist):
    Hp, Hq, p, q = None, None, None, None
    for a, b in ABlist:
        try:
            Hp, Hq = factor(n, nn, a + 1, b)
            print(Hp, Hq, a, b)
            p, q = a, b
            break
        except: pass
    return Hp, Hq, p, q
pH, qH, p, q = get_factors(n, g1, Goldbach_christian)
if pH is None or qH is None:
    pH, qH, p, q = get_factors(n, g2, Goldbach_christian)
```

#### ***Point 4***：数论变换

这一部分是一个简单的数论变换。记 $p_H$ 和 $q_H$ 分别表示 `Happy_2024` 和 `Haqqy_2024` ，$n_H$ 表示 `Hanny_2024` ，另外记 $\omega_p$ 和 $\omega_q$ 分别表示 `Exuberant_p` 和 `Exuberant_q` ，这些变量现在都已经是已知的了    
那么根据代码片段：
```python
    red_envelope1 = pow(Happy_2024 * m + Exuberant_p.order(), Haqqy_2024, Hanny_2024)
    red_envelope2 = pow(Haqqy_2024 * m + Exuberant_q.order(), Happy_2024, Hanny_2024)
    print(f"OKOK ~ I'll give U both my red_envelopes: {(red_envelope1 + red_envelope2) % Hanny_2024}", file = f)
```
可以整理得到如下表达式：    

$$
c \equiv (p_H \cdot m + \omega_p)^{q_H} + (q_H \cdot m + \omega_q)^{p_H} \pmod{n_H}
$$

注意到 $n_H = p_H \cdot q_H$ ，那么根据费马小定理可以得到如下方程组：    

$$
\begin{cases}
c \equiv \omega_p^{q_H} + q_H \cdot m + \omega_q & \pmod{p_H} \\
c \equiv p_H \cdot m + \omega_p + \omega_q^{p_H} & \pmod{q_H} \\
\end{cases}
$$    

从而有：

$$
\begin{cases}
m_p \equiv q_H ^ {-1} \cdot(c - \omega_p^{q_H} - \omega_q) & \pmod{p_H} \\
m_q \equiv p_H ^ {-1} \cdot(c - \omega_p - \omega_q^{p_H}) & \pmod{q_H} \\
\end{cases}
$$    

最后利用中国剩余定理即可恢复 $m$ 。

这里主要注意一点是前面解出来的小 `p` 和小 `q` 可能顺序不对应，调整一下就好了：

```python
# sagemath
Ep = EllipticCurve(GF(pH), [0, p])
Eq = EllipticCurve(GF(qH), [0, q])
omega_p = Ep.order()
omega_q = Eq.order()
mp = inverse(int(qH), int(pH)) * (int(c) - int(pow(omega_p, qH, pH)) - int(omega_q)) % pH
mq = inverse(int(pH), int(qH)) * (int(c) - int(pow(omega_q, pH, qH)) - int(omega_p)) % qH
m = crt([mq, mp], [pH, qH])
print(long_to_bytes(int(m)))

Ep = EllipticCurve(GF(pH), [0, q])
Eq = EllipticCurve(GF(qH), [0, p])
omega_p = Ep.order()
omega_q = Eq.order()
mp = inverse(int(qH), int(pH)) * (int(c) - int(pow(omega_p, qH, pH)) - int(omega_q)) % pH
mq = inverse(int(pH), int(qH)) * (int(c) - int(pow(omega_q, pH, qH)) - int(omega_p)) % qH
m = crt([mp, mq], [pH, qH])
print(long_to_bytes(int(m)))
```

### 完整解题代码

```python
# sagemath

n, g1, g2 = 
c = 

from Crypto.Util.number import *

New_Year = 2024
Goldbach_christian = [(p, New_Year - p) for p in sieve_base if p <= New_Year // 2 and isPrime(New_Year - p)]

def factor(nh, ng, a, b, f = lambda x: bytes_to_long(str(x).encode())):
    nh_p = None
    def test_digits(ps, qs):
        nonlocal nh_p
        if nh_p is not None:
            return False
        p = sum([pi * 10 ** i for i, pi in enumerate(ps)])
        p_with_a = p + a
        new_ps = [int(i) for i in str(p_with_a).zfill(len(ps))[::-1]][:len(ps)]
        pp = sum([(48 + pi) << (i * 8) for i, pi in enumerate(new_ps)])
        q = sum([pi * 10 ** i for i, pi in enumerate(qs)])
        q_with_b = q + b
        new_qs = [int(i) for i in str(q_with_b).zfill(len(qs))[::-1]][:len(qs)]
        qq = sum([(48 + qi) << (i * 8) for i, qi in enumerate(new_qs)])
        if p != 0 and p != 1 and nh % p == 0:
            nh_p = p
            return False
        m1 = 10 ** len(ps)
        m2 = 1 << (len(qs) * 8)
        return (p * q) % m1 == nh % m1 and (pp * qq) % m2 == ng % m2

    stack = [([], [])]
    while stack:
        ps, qs = stack.pop()
        stack += [(ps + [i], qs + [j]) for i in range(10) for j in range(10) if test_digits(ps + [i], qs + [j])]

    ng_p = f(nh_p + a)
    assert ng % ng_p == 0
    return nh_p, nh // nh_p

New_Year = 2024
Goldbach_christian = [(p, New_Year - p) for p in sieve_base if p <= New_Year and isPrime(New_Year - p)]
def get_factors(n, nn, ABlist):
    Hp, Hq, p, q = None, None, None, None
    for a, b in ABlist:
        try:
            Hp, Hq = factor(n, nn, a + 1, b)
            print(Hp, Hq, a, b)
            p, q = a, b
            break
        except: pass
    return Hp, Hq, p, q
pH, qH, p, q = get_factors(n, g1, Goldbach_christian)
if pH is None or qH is None:
    pH, qH, p, q = get_factors(n, g2, Goldbach_christian)

Ep = EllipticCurve(GF(pH), [0, p])
Eq = EllipticCurve(GF(qH), [0, q])
omega_p = Ep.order()
omega_q = Eq.order()
mp = inverse(int(qH), int(pH)) * (int(c) - int(pow(omega_p, qH, pH)) - int(omega_q)) % pH
mq = inverse(int(pH), int(qH)) * (int(c) - int(pow(omega_q, pH, qH)) - int(omega_p)) % qH
m = crt([mq, mp], [pH, qH])
print(long_to_bytes(int(m)))

Ep = EllipticCurve(GF(pH), [0, q])
Eq = EllipticCurve(GF(qH), [0, p])
omega_p = Ep.order()
omega_q = Eq.order()
mp = inverse(int(qH), int(pH)) * (int(c) - int(pow(omega_p, qH, pH)) - int(omega_q)) % pH
mq = inverse(int(pH), int(qH)) * (int(c) - int(pow(omega_q, pH, qH)) - int(omega_p)) % qH
m = crt([mp, mq], [pH, qH])
print(long_to_bytes(int(m)))
```
