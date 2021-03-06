---
layout: post
title: xshell使用技巧
categories: [tools]
tags: xshell
---

* content
{:toc}

xshell是我用过的最好用的ssh客户端工具，没有之一。这个软件完全免费，简单易用，可以满足通过ssh管理linux vps所有需要，唯一遗憾的是没有官方中文版。

***警告：不要下载所谓的汉化版，可能有木马。此前已有报道使用中文山寨版本密码被盗。*** 

官网下载地址：[http://www.netsarang.com/download/down_xsh.html](http://www.netsarang.com/download/down_xsh.html)
这里记录几则小技巧。




## 一、帐号密码保存。

**可以保存多个vps登陆信息，免去每次输入的烦恼。** 

![20120426130958_58584.png](https://dn-biezhi.qbox.me/2015/09/2503148554.png)

![20120426131016_74342.png](https://dn-biezhi.qbox.me/2015/09/778815461.png)

## 二、数字小键盘输入。

**如果不设置的话，输入数字小键盘，会显示乱码。如图设置即可：**

![20120426131324_55691.png](https://dn-biezhi.qbox.me/2015/09/393886034.png)

## 三、设置文字颜色。

**如图设置，就可以得到像黑客帝国那样绿色的文字，在你编译软件的时候，是不是恍然若见那华丽丽的数字瀑布？**

![20120426131745_96604.png](https://dn-biezhi.qbox.me/2015/09/2028490502.png)

## 四、设置命令快捷按钮

**当你管理多个vps或者经常操作vps的时候，不得不重复输入相同的命令，xshell可以设置快捷按钮，一键输入你设置的命令。**

![20120426132552_92006.png](https://dn-biezhi.qbox.me/2015/09/3598729230.png)

## 五、同一命令发送到多个ssh会话。

**也就是发送同一命令到已经登陆的多个vps。**

![20120426134229_88812.jpg](https://dn-biezhi.qbox.me/2015/09/3070724304.jpg)

## 六、通过代理登陆vps

**个人觉得这个很重要。比如你买burst.net的vps做英文站，西岸的vps到国内速度快，但是国人买的多，折腾的也多，不如东岸的稳定。但是用ssh登陆东岸vps，国内速度又很慢，这个时候加个代理，比如buyvm或者burst.net西岸的代理，访问东岸速度就有很大改善。**

![20120426135605_72432.png](https://dn-biezhi.qbox.me/2015/09/1559421879.png)

## 八、设置socks5代理服务器。

**这里设置好之后，就可以在浏览器设置代理为socks5类型，访问facebook等墙外网站。**

![20120426135942_59695.png](https://dn-biezhi.qbox.me/2015/09/3456879019.png)

## 九、上传下载。

**在vps里面安装rz、sz，就可以直接上传下载文件，不用sftp或者其他上传下载工具了。这对于下载上传小型文件非常方便，比如编辑配置文件啥的。**

**vps里面安装:**

`apt-get install lrzsz`
然后设置：

![20120426140418_86035.png](https://dn-biezhi.qbox.me/2015/09/3698758242.png)

用法：ssh输入

```bash
sz 文件名
```

即可下载vps里面的文件到本地。

ssh输入

```bash
rz
```

就会跳出窗口让你选择上传的文件，然后上传。

## 十、窗口透明和鼠标右键粘贴

**鼠标右键粘贴可将本地粘贴板内容复制到vps**

**Tools - Options**

![20120426141243_99479.png](https://dn-biezhi.qbox.me/2015/09/1946924709.png)
