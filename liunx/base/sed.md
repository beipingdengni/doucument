## sed

选项允许你修改sed命令的行为，可以使用的选项如下表所示：

| 选项      | 描述                                                 |
| --------- | ---------------------------------------------------- |
| -e script | 在处理输入时，将script中指定的命令添加到已有的命令中 |
| -f file   | 在处理输入时，将file中指定的命令添加到已有的命令中   |
| -n        | 不产生命令输出，使用print命令来完成输出              |

### 替换标记

s/pattern/replacement/flags  

##### Flag有4种可用的替换标记：

- 数字，表明新文本将替换第几处模式匹配的地方；
- g，表明新文本将会替换所有匹配的文本；
- p，表明原先行的内容要打印出来
- w file，将替换的结果写到文件中

> sed '2s/root/redhat/' test.txt    # 2s 替换,第二行的root修改为redhat
>
> sed '2,3s/root/redhat/' test.txt  # 2,3s 替换,第二到三行的root修改为redhat

### 使用文本模式过滤器

/pattern/command

### 命令组合

sed '3,${  s/this/the/  s/root/redhat/  }' test.txt

### 删除行

sed '2,4d' test.txt      行区间指定删除

sed '4,$d' test.txt      通过特殊的文件结尾字符

sed '/root4/d' test.txt 模式匹配特性也适用于删除命令

