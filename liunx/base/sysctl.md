关于sysctl.conf 配置



问题描述：

> 当SecureCRT通过SSH2远程链接Linux系统时，出现  -bash fork 无法分配内测问题

2.原因分析：

> 提示这样的错误，导致shell 命令无法响应，应该是系统内存被占满的原因

3.定位差错：

> 输入：free    查看内存使用情况   （由于系统内存不足，需多敲击几次命令，才会显示内存使用情况）
>
> 输入：sysctl kernel.pid_max   查看系统最大pid使用数
>
> 输入：ps -eLf | wc -l   查看当前使用的pid数
>
> 可以看出确实接近系统设置的最大pid个数

4.解决方案

> 修改系统最大进程数 pid_max，配置文件sysctl.conf在/etc/sysctl.conf中
>
> ①当此生效：      输入   echo  1000000 > /proc/sys/kernel/pid_max
>
> ②永久生效：      输入   echo “kernel.pid_max = 1000000” >> /etc/sysctl.conf  ; sysctl -p