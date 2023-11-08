## 时间设置

### 查看和修改系统时间

```sh
# 查看系统时间
date
# 修改系统时间
date -s "20180604 22:46:55"
```

### 查看和修改硬件时钟

```sh
# 查看硬件时钟
hwclock  --show
# 修改硬件时钟
hwclock --set --date="20180604 22:46:55"
```

### 用系统时间同步硬件时钟

```sh
hwclock --systohc
# 或者
clock --systohc

# 即将硬件时间改为和系统时间一样
```

### 用硬件时钟同步系统时间 

> 注意：必须使用root用户来修改时间才行。

```sh
hwclock --hctosys
# 或者
clock --hctosys

# hc代表硬件时间，sys代表系统时间，即将系统时间改为和硬件时钟一样
```

### 修改时区

```
# 修改时区
export TZ='Asia/Shanghai'
# 使时区生效
source ~/.bashrc
```

### 让Linux同步Internet网络上的时间

修改的是系统时间

```sh
ntpdate time.nist.gov
ntpdate time.windows.com

*/20 * * * * ntpdate ntp.jcloudcs.com > /dev/null 2>&1  
```

### 自动定时校正时间

> 安装时间同步工具：  yum install ntpdate -y

```sh
# 设定crontab计划任务自动校时：

# 使用命令crontab -e
crontab -e

#在里面写入下行命令
# 每天3:30自动进行网络校时，并同时更新BIOS的时间
30 3 * * * root /usr/sbin/ntpdate -u 210.72.145.44;hwclock -w
# 每隔一个小时同步一下internet时间，并同时更新BIOS的时间
* */1 * * * root ntpdatetime.nuri.net;hwclock -w
# 每隔2分钟执行,将输出日志到/var.log/ntpdate.log 
*/2 * * * * /usr/sbin/ntpdate 192.168.109.101 >> /var/log/ntpdate.log

# 重启服务 
service crond restart
```

### 时间修改补充

#### 查看详细时间信息

> timedatectl

结果如下

```sh
[root@localhost opt]# timedatectl
      Local time: 六 2023-02-18 17:16:06 CST
  Universal time: 六 2023-02-18 09:16:06 UTC
        RTC time: 六 2023-02-18 09:16:06
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

#### 修改时区

```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

