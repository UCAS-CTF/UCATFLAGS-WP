# Sikesibian 的 Misc 题目简化题解合集

## 1. BigConcert

> “一只耳”的音乐会 （大小写不敏感，单词之间加下划线）

使用 **Audacity** 查看音频文件，发现右声道隐藏了规律性消息，可以肉眼看，也可以使用脚本进行提取。猜测是摩斯电码，用摩斯电码转写即可。写脚本也行，但是肉眼快会更快一些。And注意断句，这是一个句子，而不是一大坨字符。那么怎么断句呢？这就是歌曲第一句的英文翻译啦！

歌曲：《香水有毒》

## 2. backdoor

> DNN is **Do Not be Nervous**.

这个题看懂代码就好了，其实就是基于原先的FLAG的所有字符构建了一个置换映射，而这个模型就是这个置换映射。对于置换映射来说，我们只要还原出来这个置换表即可。这件事情很简单，题目给到的变换后的字符种类和变换前的是一致的，所以我们类似地把这些字符直接送入模型中，然后观测输出即可获得置换表，之后将置换表逆过来即可。

```python
import torch
from torch import nn

pred_flag = b'0kRs34Rc2lta!a!!!aggg}t_BaUDyBs!yBVyCBfBMCBhRYVHtt{{{{gu'

dim = len(set(pred_flag))
input = sorted(list(set(pred_flag)))

class DNN(nn.Module):
    def __init__(self):
        super(DNN, self).__init__()
        self.fc1 = nn.Linear(dim, dim)
        self.fc2 = nn.Linear(dim, dim)
        self.fc3 = nn.Linear(dim, dim)
    
    def forward(self, x):
        x = self.fc1(x)
        x = torch.sigmoid(x)
        x = self.fc2(x)
        x = torch.sigmoid(x)
        x = self.fc3(x)
        x = torch.sigmoid(x)
        return x

model = DNN()
model.load_state_dict(torch.load(r"./model.pt"))
model.eval()

features = {}
for i in range(dim):
    fvec = [0.2 for _ in range(dim)]
    fvec[i] = 0.8
    pred_feature = model(torch.tensor(fvec, dtype=torch.float)).tolist()
    pred_chr = input[list(map(round, pred_feature)).index(1)]
    features[pred_chr] = input[i]
    print(f"{chr(pred_chr)}({pred_chr}) -> {chr(input[i])}({input[i]})")

flag = ""
for c in pred_flag:
    flag += chr(features[c])

print(flag)
```

## 3. easypicture

> 图片格式千千万。。。（本题flag中无空格，且不区分大小写）

用16进制阅读器（比如`winhex` 等工具）打开看到这是一个PDF，只是前面的PDF标识字符被改成了PDD，把这个字节改过来，然后把后缀名改成.pdf，打开即可。打开后发现是白色的，但是里面实际上是有文字的，不过也是白色的，复制出来即可。