---
title: Obspy中的时间UTCDateTime
date: 2017-07-24 11:12:25
tags: Obspy
categories: Obspy
---
## UTCDateTime
UTCDateTime是Obspy中的一个类，在obspy中对地震数据时间操作，例如SAC类型的发震时刻，P波到时，S波到时等改写，都是绕不过UTCDatetime的。
UTC时间是什么?GMT时间是什么？PDT时间又是什么?
## GMT、UTC、
[saclink]:http://7j1zxm.com1.z0.glb.clouddn.com/downloads/sac-manual-v3.5.pdf
[GMT2BJ]:http://www.timebie.com/cn/greenwichmeanbeijing.php
**GMT(Greenwich Mean Time格林威治标准时间)时间**与标准北京时间（东8区）相差8小时，这在我们修改SAC道头是经常遇到，可以参见seisman翻译的[SAC手册][saclink]page39。时区换算可以以参考[这里][GMT2BJ]

**UTC**是Coordinated Universal Time的缩写,为世界标准时间

这里是[两者差别](http://www.diffen.com/difference/GMT_vs_UTC)，不想翻译了，在实际用的过程中UTC和GMT是一样的，UTC是原子测量的，GMT是通过地球旋转测量的。GMT通常是人类可读的钟表，而UTC是数字同步时钟。GMT是关联到某个时区可以作为某个地方的当地时间，UTC计时的一个系统，没有哪个国家用UTC作为当地时间
## UTCDateTime支持的操作
加法 UTCDateTime = UTCDateTime + delta delta单位是秒，int或者float型，返回的是UTCDateTime对象
减法 delta = UTCDateTime - UTCDateTime 返回的是秒，类型为int或者float

UTCDateTime(\*args,\*\*kwargs)
args(int,float,str,date.datetime,optional)
example:
使用多个输入，并且遵循下列顺序，year,month,day,[,hour[,minute,second[,microsecond]]]
UTCDateTime(year,month,day)