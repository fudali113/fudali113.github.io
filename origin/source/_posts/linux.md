title: linux常用知识
date: 2017-03-20
tags: ["linux"]
categories:
  linux
---
## 内核日志 ##
```
cat /var/log/messages
```
## linux内存不足时kill进程策略 ##
OOM_killer是Linux自我保护的方式，在内存不足时将被唤醒，调出`/proc/{pid}/oom_score`最大者并将之kill掉

为了保护重要进程不被OOM_killer kill掉，可以配置`/proc/{pid}/oom_score_adj`,此值为内核在为每个进行打分时的附加值，最后得分等于实际值+该值
顾可以使用`echo -20 > /proc/{pid}/oom_score_adj`为实际值减去20(最后得分越小越不易被杀掉)

root进程默认获取-30的附加值
[详细来源，从源码的角度讲解](http://www.vpsee.com/2013/10/how-to-configure-the-linux-oom-killer/)
[计算oom_score的函数linux源码](https://github.com/torvalds/linux/blob/master/mm/oom_kill.c#L172-L222)

获取进程oom_score脚本
```
#!/bin/bash
for proc in $(find /proc -maxdepth 1 -regex '/proc/[0-9]+'); do
    printf "%2d %5d %s\n" \
        "$(cat $proc/oom_score)" \
        "$(basename $proc)" \
        "$(cat $proc/cmdline | tr '\0' ' ' | head -c 50)"
done 2>/dev/null | sort -nr | head -n 10
```