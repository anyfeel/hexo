---
title: linux 命令助记
date: 2018-09-24 11:02:31
tags: Linux
---

### print all system information
```
uname -a
```

### kernel version:
```
uname -r
```

### see hardware info.
```
cat /proc/meminfo cat /proc/cpuinfo
```

### count line
```
find . -name '\*.c' | xargs wc -l {}\;
```

### yum remove without dependency
```
sudo rpm -e --nodeps vim-common-7.4.160-1.el7.x86_64
```
<!-- more -->

### fio test
```
sudo /usr/local/bin/fio --filename=/dev/sda --direct=1 --rw=randrw --refill_buffers --norandommap --randrepeat=0 --ioengine=libaio --bs=4k --rwmixread=100 --iodepth=16 --numjobs=16 --runtime=1000 --group_reporting --name=4ktest
```

### docker expose port to host machine
```
docker run -p 8080:3000 my-image
```

### docker get running container bash cmd
```
docker  exec -it contained  bash
```
### tcpdump
```
sudo tcpdump -i any port 9200  -w o.pcap
```
### 批量结束进程
```
kill -9 `ps -ef|grep nginx|grep -v grep |awk '{print $2}'`
```

### 查看进程占用端口
```
ps -ef | grep processname
netstat -nap | grep pid

lsof -i:9090
pwdx 9090
lsof -P 9090 | grep cwd
```

### vim replace
```
:  %s/\(randrecord\.get('ip')\)/ptest\.\1/g
```
> 将文件中的所有 randrecord.get(‘ip’)  替换为 test.randrecord.get(‘ip')，即使用 \1  \2 等可以匹配前面第几个括号内的内容，此括号需要使用 反斜杠 转意

### 查看 glibc 版本
```
strings /lib64/libc.so.6 |grep GLIBC
```

### linux 将标出输出重定向到 /dev/null
```
2>& 1
```

###  awk 打印奇偶行
```
awk '{print $0 > NR%2}'  file
```

### sort by frequence
```
cat tmp.txt| sort | uniq -c | sort -k1,1nr  | head -30 > stoplist.txt
```

### filter by stoplist
```
tr ' ' '\n' < stoplist.txt | grep -vwFf - tmp.txt
```

### 字符串拼接
```
ps -ef | grep zicogo | awk 'BEGIN{sum=""}{sum=($2","sum)}END{print sum}'
```

### 按组统计求和
```
awk '{s[$1] += $2; a[$1] += $3 }END{ for(i in s){  printf "%-50s %-20d %-20d\n", i,s[i],a[i] } }' upstats-2017-07-05-15-05.log+
```

### 查看某个库的版本号
```
ldconfig -p | grep libssl
```

### 删除 archlinux 软件包
```
pacman -Scc
```

### history clean
```
cat .zsh_history  |  awk 'BEGIN{FS=""}{if (NF > 40 ) print ;}' >> .zsh_history_new
```

### brew install
```
brew install vim --with-lua --with-override-system-vi  --build-from-source
```

#### global floder sed
```
sed -i "s/TINY/NOX/g" `grep -R TINY -rl ./`
```
#### 批量查找替换
```
find CHANGELOG.md -type f -exec vim +"retab | wq" {} \;
```

#### 查看用户所属组
```
id -g -n $whoami
```

#### 列出 osx 系统的 java 版本
```
/usr/libexec/java_home -V
```

#### 列出 java 的信任证书
```
keytool -list -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/jre/lib/security/cacerts
```

#### java 导入证书
```
keytool -import -trustcacerts -file [certificate] -alias [alias] -keystore $JAVA_HOME/lib/security/cacerts
```

### vim highlight
```
:so $VIMRUNTIME/syntax/hitest.vim
```

### check cpu info
```
sudo dmidecode -t 4 | grep -E 'Socket Designation|Count'
    Socket Designation: CPU1
    Core Count: 8
    Thread Count: 16
    Socket Designation: CPU2
    Core Count: 8
    Thread Count: 16

lscpu
```

### awk 和 grep 日志统计分析
```
cat nginx/logs/access.log  |   grep "17/Sep"  | grep -v '2\.43' | awk  '{if ($11 > 1) print $0}'
cat nginx/logs/access.log  |   grep "17/Sep" | grep -v '2\.43' | awk  'BEGIN{a=0}{if ($11>0+a) a=$11} END{print a}'
```

### centos 查看某个库是哪个 package 提供的
```
yum whatprovides libpci.so.3
```

### 查看 PPID
```
top -> G -> 2
```
