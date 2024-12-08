# Web 1 烦人的登录

## 题目描述

这个网站的登录页面很烦人，你能绕过它吗？

## 题目地址

[点击这里](http://123.249.92.229/Ques1/)

## 前置知识

本题目不需要任何前置知识。但如果你知道有关控制台的知识则能更快的解决这个问题。

## 解题方式

1. 打开题目地址，进入登录页面。
2. 不断尝试登录，直到通过验证。

## 注意事项

1. 唯一没有给出明确提示的验证是：

```php
$pattern = '/^[a-zA-Z0-9.#$@!^%&*()\/\\,<>~`]{16,30}$/';
    if (!preg_match($pattern, $password)) {
        echo "密码长度或字符种类不正确。";
        exit;
    }
```

2. 你可以通过控制台绕过下方的勾选框验证，其代码为：

```javascript
var checkedCount = 0;
const checkboxes = document.querySelectorAll(
  'input[type="checkbox"][name="agree"]'
);
const countDisplay = document.getElementById("response");

// 监听整个文档的点击事件
document.addEventListener("click", updateCount);

function updateCount() {
  checkedCount = 0;
  checkboxes.forEach((checkbox) => {
    if (checkbox.checked) {
      checkedCount++;
    }
  });
  // console.log(checkedCount);
  // countDisplay.textContent = `已勾选数量: ${checkedCount}`;
}

document.getElementById("Form1").addEventListener("submit", function (event) {
  event.preventDefault(); // 阻止表单的默认提交行为

  var formData = new FormData(this);

  if (checkedCount < 6) {
    document.getElementById("response").innerHTML = "请至少勾选 6 项";
    return;
  } else if (if_opened != 1) {
    document.getElementById("response").innerHTML =
      "似乎你同意条款 5 的时候内心是拒绝的";
    // 将所有的复选框都取消勾选
    checkboxes.forEach((checkbox) => {
      checkbox.checked = false;
    });
    return;
  } else {
    if_opened = 0;
    checkboxes.forEach((checkbox) => {
      checkbox.checked = false;
    });

    fetch("process.php", {
      method: "POST",
      body: formData,
    })
      .then((response) => response.text())
      .then((data) => {
        document.getElementById("response").innerHTML = data;
      })
      .then(() => {
        // 更新验证码
        document.getElementById("captcha_img").src = "captcha.php";
        // 清空验证码输入框
        document.getElementById("captcha").value = "";
      })
      .catch((error) => console.error("Error:", error));
  }
});
```

在控制台输入 `checkedCount = 6` 即可绕过验证…… 才怪，还有 `if_opened` 要等于 `1`。
3\. 密码输入与学号输入均设置了不可复制，因此出于 "人道主义"，没有设置每次清空密码，因此建议一次性做完，不要刷新页面。
4\. 本题目的 flag 为 `UCTAFLAGS{1s_th1s_4_r3al_1nput_v4l1d4t10n?}`
5\. 密码的验证分别为：

* 长度在 `16` 到 `30` 之间。
* 有且一定是字母、数字、以及 `.#$@!^%&\*()/,<>\~\` 和 ` 字符至少三种。
* 不能有空格。
* 无连续相同字符。
* 没有递增或递减的字符。
* 没有连续的数字。
* 密码内数字的每一位相加，其和应等于密码长度。
* 只能使用学号中出现的数字。
* 密码中含有 factorio 字符串。

6. 本题验证码存在 `_SESSION` 中，所以只能老老实实的输入验证码。
7. 灵感来源于国科大平台的更改密码。

下附本题完整逻辑：

```php
<?php

//必须至于顶部,多服务器端记录验证码信息，便于用户输入后做校验
     session_start();
     ini_set('display_errors', 1);
     ini_set('display_startup_errors', 1);
     error_reporting(E_ALL);
    //默认返回的是黑色的照片
    $image = imagecreatetruecolor(100, 30);
    //将背景设置为白色的
    $bgcolor = imagecolorallocate($image, 255, 255, 255);
    //将白色铺满地图
    imagefill($image, 0, 0, $bgcolor);

    //空字符串，每循环一次，追加到字符串后面
    $captch_code='';

   //验证码为随机四个字符，数字和字母
   for ($i=0; $i <4 ; $i++) {
    $fontsize=6;
    $fontcolor=imagecolorallocate($image,rand(0,120),rand(0,120),rand(0,120));
    //子典。因为o和0，l和1冲突，所以我们字典中不包括易混淆的
 $data='abcdefghijkmnpqrstuvwxy3456789';
 $fontcontent = substr($data,rand(0,strlen($data)) ,1);
 $captch_code.= $fontcontent;

 $x=($i*100/4)+rand(5,10);
 $y=rand(5,10);

 imagestring($image,$fontsize,$x,$y,$fontcontent,$fontcolor);
}



   $_SESSION['captcha'] = $captch_code;
//为验证码增加干扰元素，控制好颜色，
//点
    for ($i=0; $i < 200; $i++) {
    	$pointcolor = imagecolorallocate($image,rand(50,200),rand(50,200),rand(50,200));
    	imagesetpixel($image, rand(1,99), rand(1,29), $pointcolor);
    }

//为验证码增加干扰元素
//线
    for ($i=0; $i < 3; $i++) {
    	$linecolor = imagecolorallocate($image,rand(80,220),rand(80,220),rand(80,220));
    	imageline($image, rand(1,99), rand(1,29),rand(1,99), rand(1,29) ,$linecolor);
    }

    header('content-type:image/png');
	imagepng($image);

    //销毁
    imagedestroy($image);
    ?>
```

```php
<?php
// 高亮本文件
// highlight_file(__FILE__);
session_start();
include "flag1.php";
if ($_SERVER["REQUEST_METHOD"] == "POST") {

    $id = isset($_POST['ID']) ? $_POST['ID'] : '';
    $password = isset($_POST['password']) ? $_POST['password'] : '';
    $captcha = isset($_POST['captcha']) ? $_POST['captcha'] : '';

    if (empty($id) || empty($password) || empty($captcha)) {
        echo "请提供学号、密码和验证码。";
        exit;
    }

    if (!isset($_SESSION['captcha'])) {
        echo "验证码已过期或有错误发生。";
        exit;
    }

    if ($captcha !== $_SESSION['captcha']) {
        echo "验证码不正确。";
        exit;
    }

    $pattern = '/^[a-zA-Z0-9.#$@!^%&*()\/\\,<>~`]{16,30}$/';
    if (!preg_match($pattern, $password)) {
        echo "密码长度或字符种类不正确。";
        exit;
    }


    $strength = 0;
    if (preg_match('/[a-z]/', $password)) {
        $strength++;
    }

    if (preg_match('/[A-Z]/', $password)) {
        $strength++;
    }

    if (preg_match('/[0-9]/', $password)) {
        $strength++;
    }

    if (preg_match('/[.#$@!^%&*()\/\\,<>~`]/', $password)) {
        $strength++;
    }

    if ($strength < 3) {
        echo "密码强度不够。应有大写字母、小写字母、数字和特殊字符中的至少三种。";
        exit;
    }

    if (preg_match('/(.)\1{2,}/', $password)) {
        echo "密码不能有连续相同字符。";
        exit;
    }

    $len = strlen($password);
    for ($i = 0; $i < $len - 2; $i++) {
        $c1 = ord($password[$i]);
        $c2 = ord($password[$i + 1]);
        $c3 = ord($password[$i + 2]);
        if (($c1 + 1 == $c2 && $c2 + 1 == $c3) || ($c1 - 1 == $c2 && $c2 - 1 == $c3)) {
            echo "密码不能有连续递增或递减字符。";
            exit;
        }
    }

    if (preg_match('/\d{3,}/', $password)) {
        echo "密码不能有连续的数字。";
        exit;
    }

    $sum = 0;
    for ($i = 0; $i < $len; $i++) {
        if (is_numeric($password[$i])) {
            $sum += intval($password[$i]);
        }
    }

    if ($sum != $len) {
        echo "密码内的数字的每一位相加，其和应等于密码长度。";
        exit;
    }

    $id_digits = array();
    for ($i = 0; $i < strlen($id); $i++) {
        if (is_numeric($id[$i])) {
            $id_digits[] = $id[$i];
        }
    }

    // echo "学号中的数字：";
    // print_r($id_digits);

    $password_length = strlen($password);
    for ($i = 0; $i < $password_length; $i++) {
        if (is_numeric($password[$i])) {
            if (!in_array($password[$i], $id_digits)) {
                echo "密码只能使用学号中出现的数字。不允许使用数字：";
                echo $password[$i];
                exit;
            }
        }
    }

    // 密码中应含有 factorio 字符串
    if (strpos($password, 'factorio') === false) {
        echo "密码中应含有 factorio 字符串。";
        exit;
    }

    echo "密码验证通过。";

    echo $flag;

} else {
    echo "无效的请求方法。";
}
?>
```

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <!-- ico -->
  <link rel="icon" href="1.ico" type="image/x-icon" />
  <title>Form Submission</title>
  <link rel="stylesheet" href="1.css">
</head>

<body>
  <!-- 内容居中，并且灰色背景的方框 -->
  <div class="container">
    <h1><img src="output.gif" /> 身份验证</h1>
    <form id="Form1">
      <input type="text" id="ID" name="ID" placeholder="学号" required onpaste="return false;" />
      <br />
      <input type="password" id="password" name="password" placeholder="密码" required onpaste="return false;" />
      <br />
      <br />

      <p>验证码图片：
        <img id="captcha_img" src="./captcha.php" alt="" width="100" height="30">

        <a href="javascript:void(0)" onclick="document.getElementById('captcha_img').src='./captcha.php'">换一个?</a>
      </p>

      <p>
        <input type="text" id="captcha" name="captcha" placeholder="请输入验证码" required />
      </p>

      <button type="submit">确认</button>
    </form>
    <div class="cheboxes">
      <input type="checkbox" id="agree1" name="agree" required />
      <span style="color: #2f9e25">我无条件同意<a href="Term1.html"> 条款1</a></span>
      <br /><input type="checkbox" id="agree2" name="agree" required />
      <span style="color: #2f9e25">我无条件同意<a href="Term2.html">条款 2</a></span>
      <br /><input type="checkbox" id="agree3" name="agree" required />
      <span style="color: #2f9e25">我无条件同意<a href="Term3.html">条款 3</a></span>
      <br /><input type="checkbox" id="agree4" name="agree" required />
      <span style="color: #2f9e25">我无条件同意<a href="Term4.html">条款 4</a></span>
      <br /><input type="checkbox" id="agree5" name="agree" required />
      <span style="color: #2f9e25">我无条件同意<a href="Term5.html">条款 5</a></span>
      <br /><input type="checkbox" id="agree6" name="agree" required />
      <span style="color: #2f9e25">我无条件同意<a href="Term6.html">条款 6</a></span>
      <br />
    </div>
    <div id="response" class="warning" style="display: none"></div>
    <div></div>
  </div>
  <script src="1.js"></script>
</body>

</html>
```

```javascript
var if_opened = 0;

document.getElementById("agree5").addEventListener("change", function () {
  if (this.checked) {
    // 有 90% 的概率会触发
    if (Math.random() < 0.9) {
      // document.getElementById('response').innerHTML = '你的账号已被封禁';
      // 在新标签页中打开链接
      window.open(
        "https://ys-api.mihoyo.com/event/download_porter/link/ys_cn/official/pc_default",
        "_blank"
      );
      if_opened = 1;
    }
  }
});

var checkedCount = 0;
const checkboxes = document.querySelectorAll(
  'input[type="checkbox"][name="agree"]'
);
const countDisplay = document.getElementById("response");

// 监听整个文档的点击事件
document.addEventListener("click", updateCount);

function updateCount() {
  checkedCount = 0;
  checkboxes.forEach((checkbox) => {
    if (checkbox.checked) {
      checkedCount++;
    }
  });
  // console.log(checkedCount);
  // countDisplay.textContent = `已勾选数量: ${checkedCount}`;
}

document.getElementById("Form1").addEventListener("submit", function (event) {
  event.preventDefault(); // 阻止表单的默认提交行为

  var formData = new FormData(this);

  if (checkedCount < 6) {
    document.getElementById("response").innerHTML = "请至少勾选 6 项";
    return;
  } else if (if_opened != 1) {
    document.getElementById("response").innerHTML =
      "似乎你同意条款 5 的时候内心是拒绝的";
    // 将所有的复选框都取消勾选
    checkboxes.forEach((checkbox) => {
      checkbox.checked = false;
    });
    return;
  } else {
    if_opened = 0;
    checkboxes.forEach((checkbox) => {
      checkbox.checked = false;
    });

    fetch("process.php", {
      method: "POST",
      body: formData,
    })
      .then((response) => response.text())
      .then((data) => {
        document.getElementById("response").innerHTML = data;
      })
      .then(() => {
        // 更新验证码
        document.getElementById("captcha_img").src = "captcha.php";
        // 清空验证码输入框
        document.getElementById("captcha").value = "";
      })
      .catch((error) => console.error("Error:", error));
  }
});

setInterval(function () {
  if (document.getElementById("response").innerHTML === "") {
    document.getElementById("response").style.display = "none";
  } else {
    document.getElementById("response").style.display = "block";
  }
}, 1000); // 每秒检测一次
```
