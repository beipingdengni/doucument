

```sh
sudo hwclock --show

timedatectl

sudo timedatectl set-timezone Asia/Shanghai

#日历
cal

sudo apt-get install ntp
sudo service ntp stop
sudo ntpdate cn.pool.ntp.org
sudo service ntp start
```



## 系统时间及硬件时间同步

### 

```c
rm -f /etc/localtime

命令：date -s "2021-05-07 14:09:00"	#修改系统时间

命令：hwclock --show	#查看硬件时间
命令：clock --show	#查看硬件时间
命令：hwclock	#查看硬件时间


yum -y install ntp ntpdate #安装ntpdate
  命令：ntpdate asia.pool.ntp.org	#与网络时间同步 ！同步的只是系统时间
	命令：ntpdate us.pool.ntp.org	#上面的命令不管用就使用这条命令
  
命令：hwclock --hctosys	#使系统时间同步硬件时间（系统时间变成硬件时间）
命令：hwclock --systohc	#使硬件时间同步系统时间（硬件时间变成系统时间）
```