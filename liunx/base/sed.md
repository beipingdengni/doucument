## sed

选项允许你修改sed命令的行为，可以使用的选项如下表所示：

| 选项      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| -e script | -e<script>或--expression=<script> 以选项中指定的script来处理输入的文本文件。 |
| -f file   | -f<script文件>或--file=<script文件> 以选项中指定的script文件来处理输入的文本文件。 |
| -n        | 不产生命令输出，使用print命令来完成输出                      |

### 替换标记

s/pattern/replacement/flags  

##### Flag有4种可用的替换标记：

- 数字，表明新文本将替换第几处模式匹配的地方；
- g，表明新文本将会替换所有匹配的文本；
- p，表明原先行的内容要打印出来
- w file，将替换的结果写到文件中

> `sed '2s/root/redhat/' test.txt`    # 2s 替换,第二行的root修改为redhat
>
> `sed '2,3s/root/redhat/' test.txt`  # 2,3s 替换,第二到三行的root修改为redhat

### 使用文本模式过滤器

/pattern/command

### command说明

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何东东；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正则表达式！例如 1,20s/old/new/g 就是啦！

打印匹配行：  `sed -n '/kefu-ikbs-test/p'  contentet.txt`

数据的搜寻并执行命令:   `nl testfile | sed -n '/oo/{s/oo/kk/;p;q}'` 

​		说明：找到 **oo** 对应的行，执行后面花括号中的一组命令，每个命令之间用分号分隔，这里把 **oo** 替换为 **kk**，再输出这行

修改：`sed -i 's/oo/kk/g' testfile`   注意，增加 -i 会处理原文件



### 命令组合

sed '3,${  s/this/the/  s/root/redhat/  }' test.txt

### 删除行

sed '2,4d' test.txt      行区间指定删除

sed '4,$d' test.txt      通过特殊的文件结尾字符

sed '/root4/d' test.txt 模式匹配特性也适用于删除命令

