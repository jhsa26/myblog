---
title: py_writesacheader
date: 2017-07-25 17:01:15
tags: obspy
categories: 
---
```python
import os
import commands
os.system("rm -rf ./sac_new")
os.system("mkdir sac_new")
with open("event.out","r") as f:
    picks = f.read().splitlines()
    nsta  = len(picks)
    for i in xrange(0,nsta):
        print i,nsta
        fid = open("wr_new.m","w")
        Temp = picks[i].split()
        Ecom = Temp[0]
        Ncom = Temp[1]
        Zcom = Temp[2]
        os.environ["Ecom"] = Ecom
        status,output = commands.getstatusoutput(" saclst b f $Ecom")
        begintime=output.split()
        Ppick = str(float(Temp[3]) + float(begintime[1]))
        Spick1 = str(float(Temp[4]) + float(begintime[1]))
        Spick2 = str(float(Temp[5]) + float(begintime[1]))
        snrP  = Temp[6]
        snrS1 = Temp[7]
        snrS2 = Temp[8]
        if snrS1 !="-1" or snrS2 !="-1" or snrP !="-1" :
            if snrS1 != "-1":
                Spick =  Spick1
            elif snrS2 != "-1":
                Spick =  Spick2
            if Ppick == "-1":
                Ppick = "-12345"
            if float(Spick)-float(Ppick)<3:
                Spick = "-12345"
                Ppick = "-12345"

            if Spick != "-12345" and Ppick != "-12345":
                fid.write("%s %s %s %s\n" % ("r ",Ecom,Ncom,Zcom))
                fid.write("%s %s %s %s\n" % ("ch t0 ",Ppick," t1 ", Spick))
                fid.write("wh\n")
                fid.write("%s %s %s %s %s \n " % ("sc cp -r ",Ecom,Ncom,Zcom,"  ./sac_new/" ))
                fid.write("q\n")
                fid.close()
                a=commands.getstatusoutput(" sac  wr_new.m")
```
```python
#!/bin/env python
import os,commands
from obspy import read,UTCDateTime
filestation="YASTA.LALO"
filecatlog="new_loc"
with open(filestation,'r') as f:
    temp=f.read().splitlines()
    stationinfo = {}
    for i in temp:
        sta = i.split()
        key = sta[0]
        stationinfo[key] = [sta[4],sta[8],0]
with open(filecatlog,'r') as f:
    temp1 = f.read().splitlines()
    eventinfo = {}
    for i in temp1:
        ev = i.split()
        key = ev[0]
        eventinfo[key] = ev[1:]

for count,path in enumerate(eventinfo):
    print count,path
    if os.path.isdir(path):
        os.chdir(path)
        Output=commands.getoutput('ls *.SAC').split()
        for sacfile in Output:
            [staname,component,days,sactype]=sacfile.split('.')

            if staname in stationinfo:
                if component=="000" or component=="001" or component=="002":
                    if component=="000":
                        component="E"
                    elif component=="001":
                        component = "N"
                    elif component=="002":
                        component = "Z"
                    pass
                    # print ".".join([staname,  days, component,sactype])
                    # new_name=".".join([staname,  days, component,sactype])
                    # trace=read(sacfile)[0]
                    # trace.stats.station = staname
                    # trace.stats.sac.kevnm = path
                    # trace.stats.channel = component
                    # trace.stats.sac.evdp= float(eventinfo[path][2])*1000
                    # trace.stats.sac.evlo =float(eventinfo[path][1])
                    # trace.stats.sac.evla =float(eventinfo[path][0])
                    # trace.stats.sac.stlo = float(stationinfo[staname][0])
                    # trace.stats.sac.stla = float(stationinfo[staname][1])
                    # trace.stats.sac.stdp = 0.0
                    # trace.stats.sac.mag  = float(eventinfo[path][11])
                    # nzyear = eventinfo[path][6]
                    # nzmon  = eventinfo[path][7]
                    # nzday = eventinfo[path][8]
                    # nzhour = eventinfo[path][9]
                    # nzmin = eventinfo[path][10]
                    # nzsec = eventinfo[path][11]
                    # # trace.write(new_name,format="SAC")
                    # # os.remove(sacfile)
                else:
                #注意station名字的修改以及通道号的修改
                    trace = read(sacfile)[0]
                    trace.stats.station = staname
                    trace.stats.sac.kevnm = path
                    trace.stats.channel = component
                    trace.stats.sac.evdp = float(eventinfo[path][2]) * 1000
                    trace.stats.sac.evlo = float(eventinfo[path][1])
                    trace.stats.sac.evla = float(eventinfo[path][0])
                    trace.stats.sac.stlo = float(stationinfo[staname][0])
                    trace.stats.sac.stla = float(stationinfo[staname][1])
                    trace.stats.sac.stdp = 0.0
                    trace.stats.sac.mag = float(eventinfo[path][11])
                    year = eventinfo[path][5]
                    mon = eventinfo[path][6]
                    day = eventinfo[path][7]
                    hour = eventinfo[path][8]
                    min = eventinfo[path][9]
                    sec = eventinfo[path][10]
                    time1 = "-".join([year,mon,day])+"T"+":".join([hour,min,sec])
                    nzyear = str(trace.stats.sac.nzyear)
                    nzjday = str(trace.stats.sac.nzjday)
                    nzhour = str(trace.stats.sac.nzhour)
                    nzmin = str(trace.stats.sac.nzmin)
                    nzsec = str(trace.stats.sac.nzsec)
                    time2 = "-".join([nzyear,nzjday])+"T"+":".join([nzhour,nzmin,nzsec])
                    origin = UTCDateTime(time1) - UTCDateTime(time2)
                    trace.stats.sac.o = float(origin)
                    trace.write(sacfile, format="SAC")

        os.chdir("../")



```
