---
title: 初始用itchat
date: 2017-07-18 21:32:50
tags: itchat
categories: HaHa
---

## **itchat 无疑是逢年过节的利器**
----
每当过年过节的都会接收到一些群发祝福，不回显得不礼貌，还有自己想偷懒也想来个群发微信，但又想让人看起来显得有诚意的真心祝福，还是在前面加个昵称比较好，当然这是说笑的了！我是认真的发微信的，没看到字里行间都透露着我对你们的爱么?
- - -
开始实验
- - -
```python
#coding=utf8								 #必须加上，不然汉字识别不了
import itchat,time
#这是自动登录网页的界面
@itchat.msg_register(itchat.content.TEXT)
def print_content(msg):
        print(msg['Text'])
        itchat.auto_login()
        itchat.run()
itchat.auto_login(hotReload=True)
#测试给自己发送消息，在文件群助手中可以看到
itchat.send(u'测试消息发送', 'filehelper')
#搜寻好友名字itchat.search_friends
#其中remarkName是你在通讯录上备注的好友名字，请用unicode编码汉字什么的都在前面加上u

#itchat.get_friends得到通讯录好友信息
FD=itchat.get_friends(update=True)[0:]
#朋友备注名字
myfriends=[u'胡汉三',u'张小明',u'钱多多',u'郭德刚',u'薛之谦']
for i in FD:
        name=i['NickName']
        remarkName=i['RemarkName']
        str=remarkName+u'  每天好心情:) :)' 									# 好友名字+祝福语
        Username=i['UserName']
        if remarkName in myfriends: 									   	 #从好友列表中选取你的几个好友
                   for j in range(1,10): 								#循环不要太多100以下就好了
                          itchat.send(str,toUserName=Username)
                          time.sleep(.01)  #每隔0.1秒发一条信息
```
----
如果循环变多的话微信会让你停止发消息，导致你给其他人发消息受阻，请谨慎使用
其实这里面还有很多信息可以发觉，例如统计你微信好友中的男女比例，省份，签名等，FD变量中等着你发觉
不捣乱了
