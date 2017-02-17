title: ELK学习之安装logstash
date: 2016-02-16
tags: ["ELK","logstash"]
categories:
  - 下载安装
  - 运行
---

## 下载安装 ##
* logstash基于jvm平台，所以安装前确认已安装jre
* 最新的logstash 5.*版本至少需要java8以上jre
* 对于Debian平台或者Redhat平台,官方推荐配置软件仓库并安装

### Debian/Ubuntu 平台 ###
```
wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | apt-key add -
cat >> /etc/apt/sources.list <<EOF
deb http://packages.elasticsearch.org/logstash/5.0/debian stable main
EOF
apt-get update
apt-get install logstash
```
一行一行复制输入命令行即可

有可能出现如下错误(也有可能是运行时出现)
```
Could not find any executable java binary. Please install java in your PATH or set JAVA_HOME.
```
此时需要修改logstash的启动配置文件，对Debian/Ubuntu系统，该文件路径为`/etc/logstash/startup.options`，其中的JAVACMD参数默认为`/usr/**`,顾导致找不到java命令，只需将改参数改为你的`$JAVA_HOME/bin/java`即可解决该错误

## 运行 ##
软件仓库安装后logstash bin所在路径为`/usr/share/logstash`

直接运行简单命令行输入input
```
bin/logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
```

配置文件运行

```
bin/logstash -f {配置文件路径}
```
