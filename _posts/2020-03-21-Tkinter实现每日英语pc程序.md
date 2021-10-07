---
layout: post
title: Tkinter实现每日英语PC程序
date: 2020-03-21
Author: 来自Adxiong
tags: [python, Tkinter, 爬虫]
comments: true
---
python爬取金山词典的每日一句，并用tkinter编写一个程序实现每日一句的展示
<!-- more -->

## 一、准备：
安装以下库：

 - Tkinter
 - requests
 - pillow
 - playsound
 - pyinstaller
 
 ## 二、步骤分析：
 
 1. 找到一个合适的接口，获取推送每日英语的数据。
	作者在网上看了一些API，没有比较合适的，就自己找了个接口，金山词典的每日一句：
	点我点我^v\^: [传送门](http://sentence.iciba.com/index.php?callback=jQuery19006703403891818815_1584669649852&c=dailysentence&m=getdetail&title=2020-03-21&_=1584669649854)
	
	看起来很舒服的，省去了不少麻烦，进入传送门就是我们需要的数据，每次请求数据的时候只用改下日期就可以了![图片1](https://img-blog.csdnimg.cn/20200321155359827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNDg0OTM1,size_16,color_FFFFFF,t_70)
 2. 接下来的任务就是，利用Tkinter撸个简单界面出来。通过请求返回的数据可以添加文字和美图和语音
 	![图片2](https://img-blog.csdnimg.cn/20200321155939935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIxNDg0OTM1,size_16,color_FFFFFF,t_70)
3. 项目打包，这里我们选择pyinstaller来进行打包成exe可执行程序。
		``pyinstaller -F -w -i [你的icon图片路径] [python源文件路径]``
## 三、难点解决	
1. 安装pyinstall时，可能会报错，缺少一些东西。可以先安装 pywin32和wheel在试一下
2. 图片和音频是给我们的url，如何应用？
	请求图片url，用io.BytesIO 在内容中读写bytes，用Image.open打开，通过ImageTk来实现
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321161113853.png)
音频URL，这里推荐使用 playsound（url）就可以了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321161600331.png)
3. 打包的时候，不通过，检查下你的ico是不是16X16

## 四、代码一瞥：

```python
# -*- coding: utf-8 -*-
# @Time    : 2020-3-20 下午 23:10
# @Author  : xiong
# @Site    :
# @File    : DailyEnglish.py
# @Software: PyCharm


import tkinter as tk
import io
import requests
from PIL import Image ,ImageTk
import datetime as Date
from playsound import playsound

class DailyEnglish:
    def __init__(self):
        # 创建主窗口
        App = tk.Tk()
        # 设置标题
        App.title("每日英语  by260245435")
        #图标
        App.iconbitmap('D:\QQ飞车2.0极速版\QQ飞车\otherconfigfile\JSStudio.ico')
        # 设置大小和位置
        App.geometry('800x400+500+300')
        # 进入消息循环，可以写控件
        self.dataGet()
        self.create(App)
        App.mainloop()

    def dataGet(self):
        self.date = Date.datetime.now().date()
        url = 'http://sentence.iciba.com/index.php?callback=jQuery19006703403891818815_1584669649852&c=dailysentence&m=getdetail&title={}&_=1584669649854'.format(
            self.date)
        js = requests.get(url).text
        data = eval(js[js.index('(') + 1:js.rindex(')')])
        self.sentence = data['content']  # 英文句子
        self.title = data['title']  # 日期
        self.note = data['note']  # 中文
        self.pic_url = data['picture'].replace('\\', '')  # 图片url
        self.tts_url = data['tts'].replace('\\', '')  # 音频url

    def playSound(self):
        playsound(self.tts_url)

    def tkImg(self ,pic_url):
        req = requests.get(pic_url).content
        Iopic = io.BytesIO(req)
        img = Image.open(Iopic)
        return ImageTk.PhotoImage(img)

    def create(self , App):
        # 日期
        d1 = tk.Label(App,text="Today is:" + str(self.date),font=("Comic Sans MS", 18),anchor='center')
        d1.pack()

        # 英文tkinter编写日历
        t1 = tk.Label(App,text=self.sentence,font=("Comic Sans MS", 18),wraplength=800,justify="left",anchor='center')
        t1.pack()

        # 中文
        t2 = tk.Label(App,text=self.note, font=("华为楷体", 18),wraplength=800,justify="left",anchor='center')
        t2.pack()

        #图片
        global img1
        img1 = self.tkImg(self.pic_url)
        lImg = tk.Label(App,image=img1 ,anchor='center',bg='black')
        lImg.pack()

        #按钮
        btn = tk.Button(App,text='朗读', command=self.playSound)
        btn.pack()

if __name__ == "__main__":
    DailyEnglish()
```
核心代码并不多，大家也可以尝试下，或者提出不同的看法。
