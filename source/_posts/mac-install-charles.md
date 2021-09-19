---
title: Mac安装破解版Charles
date: 2019-11-17 12:40:56
tags:
  - Mac
  - 软件
---

文章转自 https://www.jianshu.com/p/4f0573f3c5db

## 步骤

1. [下载Charles安装包](http://blog.store.dreamreal.online/Charles402.dmg)
2. 双击dmg文件，将charles拖入应用程序中
3. 在应用程序中右键 Charles，选择「显示包内容」，打开目录「Contents」=> 「Java」，用下载的 charles.jar (在 dmg 文件中) 替换目录中的 charles.jar
4. 打开 Charles，在菜单中找到 Help => Register Charles...，随便输入信息完成注册
5. 重启 Charles

## 可能遇到的问题

### 1. 打不开xxx软件,因为它不是从app store下载的

![image](https://user-images.githubusercontent.com/17465198/69003252-c4eeed80-0939-11ea-860b-42ea73b40543.png)

**解决方法：**

左上角苹果标志 => 系统偏好设置 => 安全性与隐私

![image](https://user-images.githubusercontent.com/17465198/69003259-e7810680-0939-11ea-942a-13d0f1d680d0.png)

选择"仍要打开"，就可以安装了

### 2. 破解之后显示软件已损坏

![image](https://user-images.githubusercontent.com/17465198/69003272-0e3f3d00-093a-11ea-9b6d-e31b7c1a4bf2.png)

其实并没有损坏,只是软件来自身份不明的开发者(见上图)，然后苹果就告诉你，它是坏的.....

**解决方法：**

打开终端，输入命令:
`sudo spctl --master-disable`
会让你输入密码，输入后按回车就好
再看一下安全性与隐私里面

![image](https://user-images.githubusercontent.com/17465198/69003336-aa694400-093a-11ea-9b34-9cd9072a4517.png)

多出一个任何来源，现在打开软件就没有任何问题了