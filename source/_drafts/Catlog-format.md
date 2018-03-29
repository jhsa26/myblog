---
title: obspy中支持的地震目录格式
date: 2017-07-18 22:30:25
tags: [obspy,地震目录格式]
categories: obspy
---
# obspy中的地震目录格式
## ZMAP格式
[id]:http://www.geociencias.unam.mx/~ramon/ZMAP/intro.html
[ZMAP][id]是一款研究**地震活动性**的Matlab交互式软件,如果进不了[ZMAP][id]主页的话，软件可以在这[下载][id]。ZMAP的输出的[地震目录格式](https://northridge.usc.edu/trac/csep/wiki/ZMAP-format)：

1. Longitude [deg]
2. Latitude [deg]
3. Decimal year (e.g., 2005.5 for July 1st, 2005)
4. Month
5. Day
6. Magnitude
7. Depth [km]
8. Hour
9. Minute
10. Second
11. Horizontal error
12. Depth error
13. Magnitude error
14. Name of seismic network


## QUAKEML 格式
