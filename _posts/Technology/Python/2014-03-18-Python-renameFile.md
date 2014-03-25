---
layout: post
title: python文件批量处理
category: 技术
tags: Python
keywords: Python,File,rename
description: 
---

# python文件批量处理

----------

在日常生活中下载电视剧后导入手机、Pad中看，因文件名特别长不利于查找电视剧顺序，需要对文件进行批量重命名.
例如：将文件夹中所有的文件名的“【6v电影www.dy131.com】逃出克隆岛BD中英双字1024高清CD1.rmvb”替换为”逃出克隆岛BD中英双字1024高清CD1.rmvb“。
下面这个简易程序则能解决此问题。

```python
import os
path = 'C:\\Users\\XXXX\\Desktop\\temp'
for file in os.listdir(path):
    if os.path.isfile(os.path.join(path,file))==True:
	# 把tmp替换成jpg
        newname = file.replace("tmp", "jpg")
        os.rename(os.path.join(path, file), os.path.join(path, newname))
        print(file)
```

很简单是吧？但是这就会有6行代码，实现日常的小需求：


-----------------------------------------------
golang-python学习心得   
微信公众号：golang-python  
个人微信ID：fuhao1121    
网址：http://fuhao715.github.io  
QQ：243312452   
编程学习心得轻松学编程   
回复:『 p 』查看python课程回复  
回复:『 g 』查看golang课程回复  
回复:『 m 』查看项目管理  
回复:『 w 』查看其他文章   
点击"阅读全文"进入http://fuhao715.github.io  