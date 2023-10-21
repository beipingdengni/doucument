## IF 函数

> 当 if 语句中使用 [ ] 条件修饰符时， $flag 变量必须加上引号。
>
> 当 if 语句中使用 [[]] 条件修饰符时，$flag 变量的引号可有可无
>
> 注意事项：`if [[ $file =~ ok$ ]]` 中注意有五个空格，而且正则表达式不能使用单引号或者双引号，否则会被当做普通字符串。

```
if [ 条件判断式 ]; then
    当条件判断成立时，可以进行的命令工作内容
elif [ 条件判断式二 ]; then
    当条件判断成立时，可以进行的命令工作内容
else   
    当上面的条件判断都不成立时，可以进行的命令工作内容
fi   
```

### 案例

#### 检查服务是否正常

```sh
#!/bin/bash

netstat -natp | grep :80 &> /dev/null

if [ $? -eq 0 ];then
   echo "httpd 服务正常运行！"
else
   echo "httpd 服务未开启，正在开启服务......"
   systemctl start httpd && echo "httpd 服务启动成功"
fi
```

## case语句结构

```
case 变量名称 in
“第一个变量内容”
    程序段
    ；；
“第二个变量内容”
    程序段
    ；；
*）
    不包含第一个变量内容与第二个变量内容的其他程序执行段
    默认程序段
    ；；
esac  
```

### 案例

### 一

```sh
#!/bin/bash
#Determine grades and scores
 
read -p "请输入你的分数（0-100）：" score
 
[ $score -ge 90 -a $score -le 100 ] && a="great"
[ $score -ge 70 -a $score -le 89 ] && a="medium"
[ $score -ge 60 -a $score -le 69 ] && a="pass"
[ $score -lt 60 ] && a="fail"
 
case $a in
great)
   echo "优秀"
;;
medium)
   echo "中等"
;;
pass)
   echo "及格"
;;
fail)
   echo "不及格"
;;
*)
  echo "输入有误"
esac
```

### 二

```sh
#!/bin/bash
#Determine Input
 
read -p "请输入一个字符：" string
 
case "$string" in
[a-z]|[A-Z])
    echo "你输入的是字母"
;;
[0-9])
   echo "你输入的是数字"
;;
*)
   echo "你输入的是其他字符"
esac
```

