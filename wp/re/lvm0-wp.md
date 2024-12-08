# LVM 0

## 解法 1

在 LVM_REPL 中输入 `.m(10)` 就会发现两位数无法被处理,
从而猜到可能存在 `11`, `12` 之类的隐藏 manual.

## 解法 2

呃, 克隆下仓库, 然后执行:

```shell
grep -R "ucatflags" . # 或者在 manual 下也行
```

就可以得到结果了.

## 解法 3

```shell
git log
```

flag 也写到了 git log 中了.

# FLAG

```text
ucatflags{grep-R-ucatflags-that-is-simple}
```
