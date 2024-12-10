这里是部分 **UCATFLAGS 2024 官方参考题解（Write-Up）**，欢迎大家交流

赛后反馈问卷链接：[https://www.wjx.cn/vm/wecSL4S.aspx#](https://www.wjx.cn/vm/wecSL4S.aspx#)

# 表彰

## 选手优秀 WriteUp

感谢 **catalyst** 选手提供的[**超级详细的wp**](./wp/extra/catalyst-wp.md)！！！

感谢 **山路** 选手提供的[**超级逆天的backdoor Wp**](./wp/extra/shanlu-wp.md)！！！

[**点击直接跳转至官方参考 Write-Up**](#of_wp)

## 赛后总结

### 比赛结果

|组别|冠军|亚军|季军|
|:------:|:------:|:------:|:------:|
| **大一年级组** | 我做的都队 | 不知道对不队 | 你说的都队 |
| **大二年级组** | 三带一队 | 原神启动队 | 无敌暴龙战士队 |
| **大三年级组** | wtj4nq | 1 + 1 | Confused CTF Beginners/CCB |
| **大四级组** | NeSS（Never seek success） | UNION SELECT team_name FROM teams -- | Upgradables |

### 单项最佳

**CTF 总分前三 （满分 11600 分）**
- **NeSS（Never seek success）**：9895 分 
- **三带一队**：9050 分
- **我做的都队**：8295 分

**网络空间安全知识竞赛前三（满分 375 分）**
- **找不到队**：370 分，用时 774 秒
- **你说的都队**：370 分，用时 1083 秒
- **三带一队**：370 分，用时 3263 秒

|单项|最佳队伍（最快通关或单项总分最快最高分）|
|:------:|:------:|
| **逆向（RE）** | 原神启动队 |
| **漏洞挖掘（Pwn）** | NeSS |
| **杂项（Misc）**| UNION SELECT team_name FROM teams -- |
| **密码学（Crypto）** | NeSS |
| **Web** | 三带一队 |
| **编程基础** | 原神启动队 |


### 赛后反馈

关于**出题人**：

|称谓|出题人 id |
| :------: | :------: |
| **最狠出题人** | Sikesibian、dx3906 |
| **最善出题人** | doyo2024、Crazy-13754、Sikesibian（指Welcome题目） |
| **最坑出题人** | Crazy-13754、凉凉 |
| **最肝出题人（出题最多）** | doyo2024 |
| **（其他）最帅出题人** | jiuhao47（指烦人的登录条款1） |
| **（其他）最构思出题人** | jiuhao47 |

关于**题目**：

|称谓|题目名称|
| :------: | :------: |
| **构思最巧妙** | 图中自有flag，OH NO，既然来到了国科大，lvm-2 |
| **题干最优美** | readme，既然来到了国科大，既然来到国科大 |
| **做题过程最艰辛** | 艺术就是嵌套，艺术就是嵌套，既然来到了国科大，game5 |
| **最让人红温** | lvm-0，BigConcert，既然来到了国科大，game5 |
| **性价比最高** | game4，既然来到了国科大，PPC0-Welcome to Programming！，game4 |
| **性价比最低** | game5，GoldenYear（因为hint纯骗钱的），game5 |
| **最令人耳目一新** | BigConcert（《香水有毒》音频，确实耳目一新），OH NO，既然来到了国科大，flag是怎样炼成的 |
| **（其他）hint最多** | game5 | 
| **（其他）最难绷的题目** | lvm-0 |

<span id="of_wp"></span>

# 官方参考 Write-Up

## PPC

|    Challenge    | Author | Solutions | Score |                WriteUp                |
| :-------------: | :-------: | :---: | :---: | :------------------------------------: |
| **PPC1-HelpMe** | doyo2024 |    23    |  50  | 略 |
| **PPC2-真真假假** | doyo2024 |     21     |  50  | 略 |
| **PPC3-数星星** | doyo2024 |     22     |  50  | 略 |
| **PPC4-游走** | doyo2024 |     22     |  50  | 略 |
| **PPC5-评分系统** | doyo2024 |     20     |  50  | 略 |
| **PPC6-既然你诚心诚意地问了** | doyo2024 |     18     |  200  | 略 |
| **PPC7-Calling** | doyo2024 |     21     |  150  | 略 |
| **PPC8-括号匹配** | doyo2024 |     21     |  150  | 略 |
| **PPC9-队列** | doyo2024 |     19     |  150  | 略 |
| **PPC10-Tree** | doyo2024 |     22     |  100  | 略 |

## RE

|    Challenge    | Author | Solutions | Score |                WriteUp                |
| :-------------: | :-------: | :---: | :---: | :------------------------------------: |
| **game1** | jiuhao47 |    19    |  200  | [**game1-wp**](./wp/re/game1-wp.md) |
| **game2** | jiuhao47 |    12    |  300  | [**game2-wp**](./wp/re/game2-wp.md) |
| **game3** | jiuhao47 |    11    |  400  | [**game3-wp**](./wp/re/game3-wp.md) |
| **game4** | jiuhao47 |    10    |  500  | [**game4-wp**](./wp/re/game4-wp.md) |
| **lvm-0** | 凉凉 |    13    |  100  |  [**lvm0-wp**](./wp/re/lvm0-wp.md)  |
| **lvm-1** | 凉凉 |    10    |  300  |  [**lvm1-wp**](./wp/re/lvm1-wp.md)  |
| **lvm-2** | 凉凉 |     6     |  500  |  [**lvm2-wp**](./wp/re/lvm2-wp.md)  |

## Pwn

|          Challenge          | Author | Solutions | Score |                             WriteUp                             |
| :-------------------------: | :-------: | :---: | :---: | :-------------------------------------------------------------: |
| **just_overflow_1t** | dx3906 |    19    |  150  |  [**just_overflow_1t-wp**](./wp/pwn/just_overflow_1t-wp.md)  |
|       **abcd**       | dx3906 |    12    |  200  |              [**abcd-wp**](./wp/pwn/abcd-wp.md)              |
|     **IAmFree!**     | Sikesibian |     3     |  300  |           [**iamfree-wp**](./wp/pwn/iamfree-wp.md)           |
| **knight_and_dragon** | dx3906 |     1     |  350  | [**knight_and_dragon-wp**](./wp/pwn/knight_and_dragon-wp.md) |
|   **your_own_flag**   | dx3906 |     1     |  500  |     [**your_own_flag-wp**](./wp/pwn/your_own_flag-wp.md)     |
|  **virtual_machine**  | dx3906 |     0     |  500  |   [**virtual_machine-wp**](./wp/pwn/virtual_machine-wp.md)   |

## Misc

|         Challenge         | Author | Solutions | Score |                                WriteUp                                |
| :------------------------: | :-------: | :---: | :---: | :--------------------------------------------------------------------: |
|     **猩球崛起**     | doyo2024 |    20    |  50  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
| **flag是怎样炼成的** | doyo2024 |    19    |  100  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
|      **readme**      | doyo2024 |    16    |  200  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
|      **sudoku**      | doyo2024 |    17    |  200  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
|    **frequency**    | doyo2024 |    15    |  200  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
|   **图中自有flag**   | doyo2024 |    11    |  300  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
|   **艺术就是嵌套**   | doyo2024 |     9     |  400  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
| **既然来到了国科大** | doyo2024, Sikesibian |    14    |  500  |    [**doyo2024的misc题目简化版题解合集**](./wp/misc/misc_wp.md)    |
|    **BigConcert**    | Sikesibian |    11    |  200  | [**Sikesibian的misc题目简化版题解合集**](./wp/misc/misc-wp-sike.md) |
|      **game5**      | jiuhao47 |     0     |  200  |                [**game5-wp**](./wp/misc/game5-wp.md)                |
|     **backdoor**     | Sikesibian |     6     |  300  | [**Sikesibian的misc题目简化版题解合集**](./wp/misc/misc-wp-sike.md) |
|   **easypicture**   | Sikesibian |    16    |  300  | [**Sikesibian的misc题目简化版题解合集**](./wp/misc/misc-wp-sike.md) |

## Web

|      Challenge      | Author | Solutions | Score |                      WriteUp                      |
| :------------------: | :-------: | :---: | :---: | :-----------------------------------------------: |
| **烦人的登录** | Crazy-13754 |     7     |  100  | [**烦人的登录-wp**](./wp/web/烦人的登录-wp.md) |
| **Fake Flags** | Crazy-13754 |    11    |  300  | [**Fake Flags-wp**](./wp/web/fakeflags-wp.md) |
|   **OH NO!**   | Crazy-13754 |     4     |  400  |      [**OH NO!-wp**](./wp/web/ohno-wp.md)      |

## Crypto

|       Challenge       | Author | Solutions | Score |                         WriteUp                         |
| :--------------------: | :-------: | :---: | :---: | :------------------------------------------------------: |
| **MyLittlePoly** | Sikesibian |    12    |  250  | [**MyLittlePoly-wp**](./wp/crypto/mylittlepoly-wp.md) |
|   **EzPrime**   | Sikesibian |    11    |  250  |      [**EzPrime-wp**](./wp/crypto/EzPrime-wp.md)      |
|  **AfterStory**  | Sikesibian |     8     |  300  |   [**AfterStory-wp**](./wp/crypto/AfterStory-wp.md)   |
|   **BabyDLog**   | Sikesibian |     6     |  400  |     [**BabyDLog-wp**](./wp/crypto/BabyDLog-wp.md)     |
|  **GoldenYear**  | Sikesibian |     0     |  500  |   [**GoldenYear-wp**](./wp/crypto/GoldenYear-wp.md)   |
|    **Latte**    | Sikesibian |     0     |  500  |        [**Latte-wp**](./wp/crypto/Latte-wp.md)        |
