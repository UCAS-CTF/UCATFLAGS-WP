### 题干还原

```python
from Flag import FLAG
from Crypto.Util.number import bytes_to_long

flag = bytes_to_long(FLAG)

Names_of_MyLittlePoly = [
    "Pinkie Pie",
    "Rainbow Dash",
    "Princess Celestia",
    "Applejack",
    "Fluttershy",
    "Rarity",
    "Princess Twilight",
    "Princess Luna",
    "Princess Bubblegum",
    "Princess Buttercup",
    "Princess Rainbow",
    "Princess Sparkle",
    "Princess Sparkle",
]

def calc_poly(x: int, coeffs: tuple =()):
    res = 0
    for ai in coeffs: res += (res + ai) * x
    return res

MyLittlePoly = [
    calc_poly(flag, (ord(i) for i in name)) for name in Names_of_MyLittlePoly
]

with open(r"output.txt", "w") as f:
    f.write(f"{MyLittlePoly = }")
```

### 解题思路

`calc_poly` 计算的就是以 `Names_of_MyLittlePoly` 为系数的多项式，且没有常数项。注意到这些多项式的公因子恰好是 `x`，那么就对 `MyLittlePoly` 求最大公因子即可。

### 解题代码

```python
from Crypto.Util.number import long_to_bytes
import gmpy2

MyLittlePoly = []

with open(r"./crypto1-MyLittlePoly/SourceCode/output.txt", 'r') as f:
    exec(f.read().strip())

g = MyLittlePoly[0]

for x in MyLittlePoly[1:]:
    g = gmpy2.gcd(g, x)

print(long_to_bytes(g))
```
