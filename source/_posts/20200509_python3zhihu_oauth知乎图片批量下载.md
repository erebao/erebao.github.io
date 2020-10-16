---
title: python3 zhihu_oauth 知乎图片批量下载
date: 2020-05-09 14:11:57
categories:
  - python3
  - 知乎
  - 爬虫
tags:
  - python3
  - 知乎
  - 爬虫
---

使用 python3 zhihu_oauth 爬取知乎图片批量下载
<!--more-->

## 环境：python3，zhihu_oauth，Notepad++

#### down_token.py

```python
# encoding: utf-8
from zhihu_oauth import ZhihuClient
from zhihu_oauth.exception import NeedCaptchaException
client = ZhihuClient()
try:
    client.login('+86手机号', '密码')
    print(u"登陆成功!")
except NeedCaptchaException:
    # 保存验证码并提示输入，重新登录
    with open('a.gif', 'wb') as f:
        f.write(client.get_captcha())
    captcha = input('please input captcha:')
    client.login('+86手机号', '密码', captcha)
    print(u"登陆成功!")
client.save_token('token.pkl')
```

#### down_img.py
```python
from __future__ import print_function # 使用python3的print方法
from zhihu_oauth import ZhihuClient
import re
import os
import urllib.request
import socket
#设置超时时间为30s
socket.setdefaulttimeout(30)
client = ZhihuClient()
# 登录
client.load_token('token.pkl')  # 加载token文件
id = 281282523 # https://www.zhihu.com/question/24400664(长得好看是一种怎么样的体验)
question = client.question(id)
print(u"问题:",question.title)
print(u"回答数量:",question.answer_count)
os.mkdir(question.title + u"(图片)")
path = question.title + u"(图片)"
index = 0 # 图片序号
for answer in question.answers:
    try:
        content = answer.content  # 回答内容
        re_compile = re.compile(r'<img src="(https://pic\d\.zhimg\.com/.*?\.(jpg|png))".*?>')
        img_lists = re.findall(re_compile, content)
        if (img_lists):
            for img in img_lists:
                img_url = img[0]  # 图片url
                #try:
                index += 1
                urllib.request.urlretrieve(img_url, path + u"/%d.jpg" % index)
                print(u"成功保存第%d张图片" % index)
                #except:
                    #print("出现异常:"+str(e))
                    #print(u"-------------------保存第%d张图片异常" % index)
    except:
        print("--------------------------------------异常跳过--------------------------------------")
```
