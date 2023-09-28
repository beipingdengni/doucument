# awk

过滤进程



### 自定义分隔符

```sh
awk -F: '{print $1}' /etc/passwd		#以冒号作为分隔符
awk -F"[:,_]" '{print $1}' /etc/passwd	#使用集合定义分隔符
```

### 调用系统变量、自定义变量

```sh
awk -v shell=$SHELL '{print shell}' /tmp/hosts	或者
awk '{print "'$SHELL'"}' /tmp/hosts		#双引号加单引号组合能正确获取系统变量

#自定义变量
awk -v x="bob" -v y=10 '{print x,y}' /tmp/hosts
```

