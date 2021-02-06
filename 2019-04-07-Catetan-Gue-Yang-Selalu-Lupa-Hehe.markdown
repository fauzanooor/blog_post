---
layout: post
title:  "Beberapa Catetan buat Gue yang Seringkali di-Lupain"
date:   2021-02-06
---
Check memory usage for specific process
====================================
```
ps aux | grep "process or service name" # and save the PID for that process
cat /proc/{PID}/smaps | grep -i rss |  awk '{Total+=$2} END {print Total/1024" MB"}'
```