# sql注入与xssSRC实战
sql注入与xss在这次之前真是一概不知，即使现在也可以说一窍不通，不过还是实打实的挖了两个站点，遂记录：

以我浅薄的认知看来，注入类漏洞本质就是错误闭合的艺术，sql注入也不免俗套，以下是我所挖掘的漏洞的经过，我会尝试复原 他们是如何错误闭合的。

## 上海XXXX幼儿园主页sql注入与反射形xss注入
这是一个非常“教科书”的sql注入漏洞
[![](https://pic.imgdb.cn/item/64f2ca29661c6c8e54af08c3.jpg)](https://pic.imgdb.cn/item/64f2ca29661c6c8e54af08c3.jpg)
id=1 那么第一步就是'
很显然出问题了 没有信息返回
[![](https://pic.imgdb.cn/item/64f2ca9d661c6c8e54af1bc8.jpg)](https://pic.imgdb.cn/item/64f2ca9d661c6c8e54af1bc8.jpg)
and 1=1
and 1=2
分别是正常回显与无回显
[=![](https://pic.imgdb.cn/item/64f2cc06661c6c8e54b02d4f.png)](https://pic.imgdb.cn/item/64f2cc06661c6c8e54b02d4f.png)
所有他大概就是个正常查询语句 没有闭合与后继大约是这样
`SELECT * FROM 表 id = $id`
然后就是 order by 数列数
union select 尝试找回显

然后就是反射xss 虽说这东西一般src都不收 但有输入框也蛮写一下
正如我前面所说 注入类攻击本质是错误闭合的艺术
例如 这是一个正常的script弹窗语句
`<script>alert(1)</script>`
他执行的效果是这样的
![](https://pic.imgdb.cn/item/64f86e7b661c6c8e5405cb06.jpg)

我们要做的就是想办法让他被网页解析
我们来看看这次挖掘到的源码
`<input type="text" placeholder="请输入您要搜索的关键词" name="product_name" value="11">`
url:
`http://www.xxx.com/index.php?m=product&c=product&a=product_list&product_name=11`
很显然 关键点在这个prodeuct_name 他传入的参数被直接拼接value 然后直接输出并没有做过滤
当我们直接输入`<script>alert(1)</script>`时就会出现
![](https://pic.imgdb.cn/item/64f87a5f661c6c8e54091015.jpg)
很显然他被框在了""中这样他只会正常显示而不会解析 那么目光回到">这个上 既然他没有对任何东西进行过滤 那么我们就可以利用这个来构造语句
`"><script>alert(1)</script>`
这样他与页面源码结合就会变成这样 
`<input type="text" placeholder="请输入您要搜索的关键词" name="product_name" value="11"><script>alert(1)</script>">`
此时便成功的闭合了前面的input语句 将srcipt独立了出来

