# find命令中的参数以及作用

| 参数                  | 作用                                                         |
| --------------------- | ------------------------------------------------------------ |
| -name                 | 匹配名称 inum 通过索引节点号搜索                             |
| -perm 644             | 匹配权限（mode为完全匹配，-mode为包含即可）                  |
| -user                 | 匹配所有者                                                   |
| -group                | 匹配所有组                                                   |
| -mtime -n +n          | 匹配修改内容的时间（-4指小于等于4天内的文件名；+4,大于等于5天前的文件名；4指前4~5那一天的文件） -cmin |
| -atime -n +n          | 匹配访问文件的时间 -amin                                     |
| -ctime -n +n          | 匹配修改文件权限的时间 -cmin                                 |
| -nouser               | 匹配无所有者的文件                                           |
| -nogroup              | 匹配无所有组的文件                                           |
| -newer f1 ! -newer f2 | 匹配比文件f1新但比f2旧的文件 -a -o ! -not                    |
| --type b/d/c/p/l/f    | 匹配文件类型（后面的字母参数依次表示块设备、目录、字符设备、管道、链接文件、文本文件） |
| -size                 | 匹配文件的大小（+50KB为查找超过50KB的文件，而-50KB为查找小于50KB的文件） |
| *-exec …… {}\;        | 后面可跟用于进一步处理搜索结果的命令 -ok -print              |
| -mindepth n           | 从第 n级目录开始搜索 eg: /etc 的第三级子目录开始搜索。 find /etc -mindepth 3 |
| -maxdepth n           | 表示至多搜索到第 n-1 级子目录。0当前目录                     |

使用案例

```shell
# 查找压缩
find / -name *.jpg -type f -print | xargs tar -cvzf images.tar.gz

# 保留三天日志
find /export/Logs/* -name "*.log" -mtime +3 | xargs rm -rf

# 复制文件，文件到一个外部的硬盘驱动
ls *.jpg | xargs -n1 -i cp {} /external-hard-drive/directory

```

